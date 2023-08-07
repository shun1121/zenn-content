---
title: "Astro, GSAPでカウントアップアニメーションを実装する"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Astro", "GSAP", "React", "TailwindCSS"]
published: false
---

## はじめに

Astroのプロジェクトにアニメーションを当てたいと思ったので、GSAPを使用してカウントアップアニメーションを実装ました。その際行った手順を以下に記述していきます。
https://github.com/shun1121/astro-animation

## 主な使用技術

Reactをプロジェクトに入れていますが、今回の実装箇所では使用していません。

```js
  "astro": "^2.7.4",
  "gsap": "^3.12.2",
  "react": "^18.0.0",
  "tailwindcss": "^3.0.24"
```

## ディレクトリ構成

最終的なディレクトリ構成は以下のようになります。
`src/components/Countup/Countup.astro`を`src/pages/index.astro`にインポートしてカウントアップアニメーションを反映しています。

```
  .vscode/
  node_modules
  public/
  src/
    ├─ components/
    │    ├─ Countup/
    │    │   ├─ Countup.astro
    ├─ layouts/
    │    ├─ Layout.astro
    ├─ pages/
    │    ├─ index.astro
    ├─ env.d.ts
  .gitignore
  astro.config.mjs
  package.json
  pnpm-lock.yaml
  README.md
  tailwind.config.cjs
  tsconfig.json
```

## プロジェクトの作成

### Astroをインストール

```
pnpm create astro@latest
```
今回は`pnpm`を使用していますが、`npm`や`yarn`など他のパッケージマネージャーでも大丈夫です。

https://docs.astro.build/ja/install/auto/
https://larainfo.com/blogs/install-astro-react-tailwind-css

### Tailwind CSSをインストール

:::details Tailwind CSSのインストール
```jsx
pnpm astro add tailwind

Resolving packages...

Astro will run the following command:

If you skip this step, you can always run it yourself later

╭──────────────────────────────────────────────────────────────╮

│ pnpm add @astrojs/tailwind astro@^2.6.5 tailwindcss@^3.0.24  │

╰──────────────────────────────────────────────────────────────╯

✔ Continue? … yes

✔ Installing dependencies...

Astro will generate a minimal ./tailwind.config.cjs file.

✔ Continue? … yes

Astro will make the following changes to your config file:

╭ astro.config.mjs ─────────────────────────────╮

│ import { defineConfig } from 'astro/config';  │

│                                               │

│ import tailwind from "@astrojs/tailwind";     │

│                                               │

│ // https://astro.build/config                 │

│ export default defineConfig({                 │

│   integrations: [tailwind()]                  │

│ });                                           │

╰───────────────────────────────────────────────╯

✔ Continue? … yes

success  Added the following integration to your project:

- @astrojs/tailwind
```
:::

#### Reactをインストール
カウントアップアニメーションでは使用しないのでスキップしていただいて大丈夫です。

:::details Reactのインストール
```jsx
pnpm astro add react

astro-animation@0.0.1 astro /Users/fukunishi.s/personal-works/astro-animation

> astro "add" "react"

✔ Resolving packages...

Astro will run the following command:

If you skip this step, you can always run it yourself later

╭─────────────────────────────────────────────────────────────────────────────────────────────────────────╮

│ pnpm add @astrojs/react @types/react-dom@^18.0.6 @types/react@^18.0.21 react-dom@^18.0.0 react@^18.0.0  │

╰─────────────────────────────────────────────────────────────────────────────────────────────────────────╯

✔ Continue? … yes

✔ Installing dependencies...

Astro will make the following changes to your config file:

╭ astro.config.mjs ─────────────────────────────╮

│ import { defineConfig } from 'astro/config';  │

│ import tailwind from "@astrojs/tailwind";     │

│                                               │

│ import react from "@astrojs/react";           │

│                                               │

│ // https://astro.build/config                 │

│ export default defineConfig({                 │

│   integrations: [tailwind(), react()]         │

│ });                                           │

╰───────────────────────────────────────────────╯

✔ Continue? … yes

success  Added the following integration to your project:

- @astrojs/react

Astro will make the following changes to your tsconfig.json:

╭ tsconfig.json ──────────────────────────╮

│ {                                       │

│   "extends": "astro/tsconfigs/strict",  │

│   "compilerOptions": {                  │

│     "jsx": "react-jsx",                 │

│     "jsxImportSource": "react"          │

│   }                                     │

│ }                                       │

╰─────────────────────────────────────────╯

✔ Continue? … yes

success  Successfully updated TypeScript settings
```
:::

### パスのエイリアス変更

```js
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---
```

上記のように相対パスでコンポーネントをインポートしているものにエイリアスを当てます。

`tsconfig.json`を開いて下記のように記述します。


```
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "react",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },
  }
}
```
`paths`で`./src/`配下を`@`に変えます。この際に`baseUrl`に`.`を設定していないとエイリアスのパスが解決されないのであらかじめ確認しておきます。

https://docs.astro.build/ja/guides/aliases/


## 数字のカウントアップ


```js
---
---

<div class="countup-trigger">
  <div class="h-full w-full bg-gray-400 py-[60px]">
    <div class="w-[90%] h-[500px] bg-gray-900 mx-auto"></div>
    <div class="flex mt-[52px] mx-auto w-[90%] gap-[24px]">
      <div class="flex justify-center items-center w-full h-[400px] bg-gray-300 text-[80px]" >
        <span class="countup-target" data-from="0" data-to="100">0</span>
      </div>
      <div class="flex justify-center items-center w-full h-[400px] bg-gray-300 text-[80px]" >
        <span class="countup-target" data-from="0" data-to="50">0</span>
      </div>
    </div>
  </div>
</div>

<script>
  import { gsap } from 'gsap'
  import { ScrollTrigger } from 'gsap/ScrollTrigger'

  gsap.registerPlugin(ScrollTrigger)
  const countupTargets = document.querySelectorAll<HTMLElement>('.countup-target')
　　　　console.log(countupTargets)

  countupTargets.forEach((countupTarget) => {
    const countFrom = countupTarget.dataset.from
    const countTo = countupTarget.dataset.to
    console.log(countFrom) 
    console.log(countTo)

    if (countFrom === undefined || countTo === undefined) {
      // エラーハンドリングなど、データ属性が存在しない場合の処理を行います
      return
    }

    const countupSpan = {
      from: parseInt(countFrom),
      to: parseInt(countTo)
    }

    const countupFrom = {
      count: countupSpan.from
    }

    gsap.to(countupFrom, {
      count: countupSpan.to,
      duration: 1,
      ease: 'none',
      scrollTrigger: {
        trigger: countupTarget,
        start: 'top 80%', // スクロールトリガーの開始位置を調整します
        toggleActions: 'play none none reverse',
        markers: true,
      },
      onUpdate: () => {
        countupTarget.textContent = Math.floor(countupFrom?.count).toLocaleString()
      }
    })
  })
</script>
```
### GSAPのインポート

GSAPとカウントアップをスクロールをトリガーに行うので、`ScrollTrigger`をインポートします。GSAPは`ScrollTrigger`をデフォルトで組み込んでいないため、`registerPlugin()`に登録します。

```js
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);
```

### datasetに関する説明

`data-`から始まる接頭辞、例えば`data-from`, `data-to`が、`dataset.from`, `dataset.to`と対応しています。ここでカウントアップの最小値と最大値を設定します。`countupTargets` が複数要素がある配列なのでループさせてそれぞれの場合の`countFrom`, `countTo`を用意します。

```jsx
<span class="countup-target" data-from="0" data-to="100">0</span>
<span class="countup-target" data-from="0" data-to="50">0</span>

countupTargets.forEach((countupTarget) => {
  const countFrom = countupTarget.dataset.from
  const countTo = countupTarget.dataset.to
  console.log(countFrom) // 1回目   0,  2回目 0
  console.log(countTo)   // 1回目 100,  2回目 50

  ...
}
```


`countFrom`, `countTo`が`undefined`の可能性があるので警告が出ます。`countupSpan`でカウントアップの最小値と最大値を設定する前に`countFrom`とc`ountTo`が`undefined`であった場合の処理を書いておきます。

```jsx
if (countFrom === undefined || countTo === undefined) {
  return
}

const countupSpan = {
  from: parseInt(countFrom),
  to: parseInt(countTo)
}
```

### メソッドの引数とアニメーションプロパティ関する説明

#### メソッドの説明

`gsap.to()`には二つの引数を指定します。第一引数には、アニメーションの対象となるプロパティを持たせたオブジェクトを指定します。第二引数にはアニメーションのプロパティやオプションを指定します。


```jsx
const countupFrom = {
  count: countupSpan.from
}

gsap.to(countupFrom, {
  count: countupSpan.to,
  duration: 1,
  ease: 'none',
  scrollTrigger: {
    trigger: countupTarget,
    start: 'top 80%', // スクロールトリガーの開始位置を調整します
    markers: true,
    toggleActions: 'play none none reverse',
  },
  onUpdate: () => {
    countupTarget.textContent = Math.floor(countupFrom?.count).toLocaleString()
  }
})
```
#### アニメーションプロパティの説明

#### count :

カウントアップの最大値を設定。

#### duration :

アニメーションの持続時間。

#### ease :

アニメーションの緩急。

https://note.com/ritar/n/n5e8ed0e07917

#### scrollTrigger :

スクロールによって引き起こされるアニメーションの設定をするプロパティ。

#### trigger :

スクロールでアニメーションを引き起こす位置を指定。

#### start :

アニメーションがどのタイミングでスタートするかを設定。

例えば、`start: 'top center’` と設定すると、今回のカウントアップアニメーションでは数字を記述している`countupTarget`の`top`の位置にscroller-startのバーが来た時にカウントアップが始まる。現状scroll-startのバーは`center`に指定していて`countupTarget`の`top`と交わらないのでアニメーションが起こりません。これを`center`ではなく、`80%`や`bottom`と設定すると交わるようになるのでアニメーションが走ります。

#### marker :

デバッグ用のマーカーを表示するかどうかを設定。`true`であればマーカーが表示される。

#### toggleActions :

アニメーションの開始と終了の動作を指定。`onEnter`, `onLeave`, `onEnterBack`, `onLeaveBack`の4つの状態の時の動作を指定できる。
`'play none none reverse'` は、スクロールトリガーの開始時にアニメーションを再生し、終了時に逆方向にアニメーションを逆再生する。
`'play none none reset'` の場合は最初の位置に戻った際に、アニメーションをリセットする。
https://gyazo.com/c8aa2fd24e17bee7c96a7768da4f9080

#### onUpdate :
アニメーションの更新時に実行されるコールバック関数を指定。ここでは、`countupFrom.count`の整数部分を取得し、countupTarget.textContent に表示します。これにより、アニメーションの進捗に応じて要素のテキストが更新されます。

https://greensock.com/docs/v3/Plugins/ScrollTrigger
