---
title: 'NestJSをDistrolessを使った今ドキのDocker本番環境で作った'
emoji: '🦔'
type: 'tech'
topics: ['nestjs', 'docker']
published: false
---

## Distroless とは？

Google 製の Docker イメージです。

https://github.com/GoogleContainerTools/distroless

> アプリケーションとそのランタイムの依存関係だけが含まれています。

本番環境に特化したイメージであり、ランタイムに不要なプログラム（シェルなど）を含まないのでパフォーマンスの向上・セキュリティ向上が見込めます。

また alpine ではなく Distroless を使う理由はこちらの記事が参考になるので、気になる方はこちらを参照してください。
Distroless で本番環境を作ろうとなったきっかけの記事でもあります。
https://blog.inductor.me/entry/alpine-not-recommended

## 結果

| 削減対策無し slim | マルチビルド slim | マルチビルド distroless |
| :---------------: | :---------------: | :---------------------: |
|     667.66 MB     |     177.56 MB     |        129.1 MB         |

イメージサイズを削減しつつ、ランタイムに不要な最小限の構成になったことでセキュリティも上がった。

## 今回の環境バージョン

```
$ node -v
v14.16.0

$ yarn -v
1.22.10
```

## NestJS で Hello World! を表示する

公式の Introduction に沿って Hello World! が表示するまで作成します。
https://docs.nestjs.com/

```
npm i -g @nestjs/cli
nest new nestjs-docker
```

途中で出てくるパッケージマネージャーはお好きな方をお使いください。
今回は yarn を使っていきたいと思います。

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details nest new nestjs-docker 実行結果

```sh
$ nest new nestjs-docker
⚡  We will scaffold your app in a few seconds..

CREATE nestjs-docker/.eslintrc.js (631 bytes)
CREATE nestjs-docker/.prettierrc (51 bytes)
CREATE nestjs-docker/README.md (3339 bytes)
CREATE nestjs-docker/nest-cli.json (64 bytes)
CREATE nestjs-docker/package.json (1975 bytes)
CREATE nestjs-docker/tsconfig.build.json (97 bytes)
CREATE nestjs-docker/tsconfig.json (339 bytes)
CREATE nestjs-docker/src/app.controller.spec.ts (617 bytes)
CREATE nestjs-docker/src/app.controller.ts (274 bytes)
CREATE nestjs-docker/src/app.module.ts (249 bytes)
CREATE nestjs-docker/src/app.service.ts (142 bytes)
CREATE nestjs-docker/src/main.ts (208 bytes)
CREATE nestjs-docker/test/app.e2e-spec.ts (630 bytes)
CREATE nestjs-docker/test/jest-e2e.json (183 bytes)

? Which package manager would you ❤️  to use?
  npm
❯ yarn
```

```sh
✔ Installation in progress... ☕

🚀  Successfully created project nestjs-docker
👉  Get started with the following commands:

$ cd nestjs-docker
$ yarn run start


                          Thanks for installing Nest 🙏
                 Please consider donating to our open collective
                        to help us maintain this package.


               🍷  Donate: https://opencollective.com/nest
```

:::

<!-- textlint-enable -->

```
cd nestjs-docker
yarn run start
```

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details yarn run start 実行結果

```sh
$ yarn run start
yarn run v1.22.10
$ nest start
[Nest] 75685   - 2021/03/14 15:45:58   [NestFactory] Starting Nest application...
[Nest] 75685   - 2021/03/14 15:45:58   [InstanceLoader] AppModule dependencies initialized +65ms
[Nest] 75685   - 2021/03/14 15:45:58   [RoutesResolver] AppController {}: +4ms
[Nest] 75685   - 2021/03/14 15:45:58   [RouterExplorer] Mapped {, GET} route +2ms
[Nest] 75685   - 2021/03/14 15:45:58   [NestApplication] Nest application successfully started +1ms
```

:::

<!-- textlint-enable -->

[http://localhost:3000](http://localhost:3000) を開いて Hello World! が表示されたら完了です。

## Docker 環境を作る

### Dockerfile の作成

`Dockerfile` と `.dockerignore` のファイルを作成します。

```
touch Dockerfile
touch .dockerignore
```

`Dockerfile` の内容を書き換えます。
コメントには npm を使った場合のコマンドを表示しています。

```Dockerfile:Dockerfile
#==================================================
# Build Layer
FROM node:14-slim as build

WORKDIR /app

COPY package.json yarn.lock ./
# COPY package.json package-lock.json ./

RUN yarn install --non-interactive --frozen-lockfile
# RUN cpm ci

COPY . .

RUN yarn build
# RUN npm run build

#==================================================
# Package install Layer
FROM node:14-slim as node_modules

WORKDIR /app

COPY package.json yarn.lock ./
# COPY package.json package-lock.json ./

RUN yarn install --non-interactive --frozen-lockfile --prod
# RUN cpm ci --only=production

#==================================================
# Run Layer
FROM gcr.io/distroless/nodejs:14

WORKDIR /app

ENV NODE_ENV=production

COPY --from=build /app/dist /app/dist
COPY --from=node_modules /app/node_modules /app/node_modules

CMD ["dist/main"]
```

```ignore:.dockerignore
.git/
node_modules/
dist/
```

### イメージをビルド

作成した `Dockerfile` をビルドします。

```sh
docker build --tag nestjs-docker .
```

<!-- textlint-disable ja-technical-writing/ja-no-mixed-period -->

:::details 実行結果

```sh
% docker build --tag nestjs-docker .
[+] Building 80.8s (17/17) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 668B
 => [internal] load .dockerignore
 => => transferring context: 78B
 => [internal] load metadata for gcr.io/distroless/nodejs:14
 => [internal] load metadata for docker.io/library/node:14-slim
 => [build 1/6] FROM docker.io/library/node:14-slim@sha256:e8a3dbe7f6d334acfe0365260626d3953073334de4c0fde00f93e8e9e19ed5d5
 => [stage-2 1/4] FROM gcr.io/distroless/nodejs:14@sha256:24601f9c48037634eb06dc24d0d288af3a1ad7f4175845fbd0085b4ac51effa5
 => [internal] load build context
 => => transferring context: 535.75kB
 => CACHED [stage-2 2/4] WORKDIR /app
 => CACHED [build 2/6] WORKDIR /app
 => [build 3/6] COPY package.json yarn.lock ./
 => [node_modules 4/4] RUN yarn install --non-interactive --frozen-lockfile --prod
 => [build 4/6] RUN yarn install --non-interactive --frozen-lockfile
 => [build 5/6] COPY . .
 => [build 6/6] RUN yarn build
 => [stage-2 3/4] COPY --from=build /app/dist /app/dist
 => [stage-2 4/4] COPY --from=node_modules /app/node_modules /app/node_modules
 => exporting to image
 => => exporting layers
 => => writing image sha256:edbed248b26d6b341b3b430016547cc190245bd8517021f74841fede11d775df
 => => naming to docker.io/library/nestjs-docker
```

:::

<!-- textlint-enable -->

イメージサイズは `129.1 MB` になりました。

![](https://storage.googleapis.com/zenn-user-upload/53kot48bm59jg6ifif5qocwzqa7x)

### イメージを実行

<!-- textlint-disable ja-technical-writing/ja-no-redundant-expression -->

ビルドしたイメージを実行します。

<!-- textlint-enable -->

```
docker run -d -p 3000:3000 --name nestjs-docker nestjs-docker
```

[http://localhost:3000](http://localhost:3000) を開いて Hello World! が表示されたら成功です。

### 削減対策をしない slim 環境

シングルステージでのビルドです。
ランタイムに不要なパッケージなど含まれるのでイメージサイズが大きいです。

:::details Dockerfile

```Dockerfile:Dockerfile
FROM node:14-slim

WORKDIR /app

COPY . .

RUN yarn install --non-interactive --frozen-lockfile
RUN yarn build

ENV NODE_ENV=production

CMD ["yarn", "start:prod"]
```

:::

`SIZE 667.66 MB`

![](https://storage.googleapis.com/zenn-user-upload/stdqe59sey9aj8v4nijtiek5hec8)

### マルチステージビルドでの slim 環境

マルチステージビルドを使いランタイムに必要なプログラムのみをイメージに含めています。
`yarn` や `sh` など本番では使わないプログラムが含まれるため Distroless と比べると少しサイズが大きいですね。

:::details Dockerfile

```Dockerfile:Dockerfile
#==================================================
# Build Layer
FROM node:14-slim as build

WORKDIR /app

COPY package.json yarn.lock ./

RUN yarn install --non-interactive --frozen-lockfile

COPY . .

RUN yarn build

#==================================================
# Package install Layer
FROM node:14-slim as node_modules

WORKDIR /app

COPY package.json yarn.lock ./

RUN yarn install --non-interactive --frozen-lockfile --prod

#==================================================
# Run Layer
FROM node:14-slim

WORKDIR /app

ENV NODE_ENV=production

COPY --from=build /app/dist /app/dist
COPY --from=node_modules /app/node_modules /app/node_modules

CMD ["node", "dist/main"]
```

:::

`SIZE 177.56 MB`

![](https://storage.googleapis.com/zenn-user-upload/k0n7oezlz126e05r3c7eaoogn3c5)

## まとめ

| 削減対策無し slim | マルチビルド slim | マルチビルド distroless |
| :---------------: | :---------------: | :---------------------: |
|     667.66 MB     |     177.56 MB     |        129.1 MB         |

- 減対策無し slim と比較し `538.56 MB` のイメージサイズ削減
- マルチビルド slim と比較し `48.46 MB` のイメージサイズ削減

イメージサイズが小さくなるのはとても楽しいですね！
マルチステージビルドは使っていたのですが Distroless は知らなかったので積極的に使っていこうと思います。

Distroless にはシェルが含まれていないため、アタッチしてデバッグする際はデバッグ用のイメージを使いましょう！

`gcr.io/distroless/nodejs:debug`
