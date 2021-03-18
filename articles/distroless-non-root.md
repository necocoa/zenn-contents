---
title: 'Distrolessイメージをroot以外のユーザーで実行する'
emoji: '🤖'
type: 'tech'
topics: ['docker']
published: true
---

今回は Distroless を使う時に root 以外のユーザーで実行する方法を試してみました。

## 結論

`COPY --chown=nonroot:nonroot` `USER nonroot` を使う。

```Dockerfile:Dockerfile
FROM gcr.io/distroless/nodejs:14

WORKDIR /app

COPY --chown=nonroot:nonroot --from=build /app /app

USER nonroot
EXPOSE 3000
ENTRYPOINT ["dist/main"]
```

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details Node.js の Dockerfile

```Dockerfile:Dockerfile
#==================================================
# Build Layer
FROM node:14-slim as build

WORKDIR /app

COPY package.json yarn.lock ./
COPY . .

RUN yarn install --non-interactive --frozen-lockfile

RUN yarn build

#==================================================
# Package install Layer
FROM node:14-slim as node_modules

WORKDIR /app

COPY package.json yarn.lock ./
COPY . .

RUN yarn install --non-interactive --frozen-lockfile --prod

#==================================================
# Run Layer
FROM gcr.io/distroless/nodejs:14

WORKDIR /app

COPY --chown=nonroot:nonroot --from=build /app/dist /app/dist
COPY --chown=nonroot:nonroot --from=node_modules /app/node_modules /app/node_modules

USER nonroot
EXPOSE 3001
ENTRYPOINT ["dist/main"]

```

:::

<!-- textlint-enable -->

## Distroless で root 以外のユーザーを扱う

通常の Docker イメージだとシェルが入っているため `adduser` などでユーザーを作ります。
Distroless で同様に行おうとしたら実行するシェルが無いとエラーが発生します。

```
 > [stage-2 2/6] RUN groupadd -r app && useradd -r -g app -G app:
#6 0.332 container_linux.go:370: starting container process caused: exec: "/bin/sh": stat /bin/sh: no such file or directory
```

やり方を調べていたら `nonroot` という名前のユーザー・グループが作成されているようです。
https://github.com/GoogleContainerTools/distroless/issues/235

`gcr.io/distroless/nodejs:debug` を使って nonroot になっているか確認します。

```sh
$ whoami
nonroot

$ ls -l
drwxr-xr-x    2 nonroot  nonroot       4096 Mar 18 16:08 dist
drwxr-xr-x    3 nonroot  nonroot       4096 Mar 18 16:08 node_modules
```

問題なく nonroot になっていました。
続いて Distroless にあるユーザー・グループを確認しました。

```sh
$ cat /etc/passwd
root:x:0:0:root:/root:/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/sbin/nologin
nonroot:x:65532:65532:nonroot:/home/nonroot:/sbin/nologin

$ cat /etc/group
root:x:0:
nobody:x:65534:
tty:x:5:
staff:x:50:
nonroot:x:65532:
```

これらのユーザー・グループは初期から入っているようです。
