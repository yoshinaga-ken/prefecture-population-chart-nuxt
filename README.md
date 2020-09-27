# 都道府県別の総人口推移グラフを表示するSPA(Single Page Application)

![image](prefecture-population-chart-nuxt.gif)

## DemoPage
- [デモページA](https://sakanaclub.xsrv.jp/prefecture-population-chart-nuxt/)
- [デモページB](https://sakanaclub.xsrv.jp/prefecture-population-chart-nuxt2/)

## 内容
- RESAS(地域経済分析システム) APIの「都道府県一覧」からAPIを取得する
- APIレスポンスから都道府県一覧のチェックボックスを動的に生成する
- 都道府県にチェックを入れると、RESAS APIから選択された都道府県の「人口構成」を取得する
- 人口構成APIレスポンスから、X軸:年、Y軸:人口数の折れ線グラフを動的に生成して表示する

## Build Setup

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```

For detailed explanation on how things work, check out [Nuxt.js docs](https://nuxtjs.org).
