# Vite + React + JS + TailWind

Vite + React + JS + TailWind による開発環境の構築。

---

<span id="index"></span>

## 0. 目次
- [開始](#initiate)
- [TailWindCss 導入](#install_tailwindcss)
- [vite.config.js の設定](#write_vite_config)

---

<span id="initiate"></span>

## 1. 開始

※ 正確な情報は [https://ja.vitejs.dev/guide/](https://ja.vitejs.dev/guide/) を参照。

### 1.1. プロジェクト作成

次のコマンドをうち、vanilla js の react を選択。

```bash
npm create vite@latest my-app
```

### 1.2. プロジェクトの初期化

```bash
cd my-app
npm i
```


---

<span id="install_tailwindcss"></span>

[目次](#index)

## 2. TailWindCss の導入

### 2.1. コマンド

```bash
npm i -D autoprefixer tailwindcss
npx tailwindcss init -p
```
[tailwind のガイド](https://tailwindcss.com/docs/guides/vite#react)では、ここで postcss もインストールすることになっているが、vite がデフォルトで持っている postcss を使うので不要。
また、init したときに、postcss.config.js も作成されるが、vite.config に書くので、必要ない。消してよい。


### 2.2. src/index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```


### 2.3. tailwind.config.js

purge:[] は、content:[] に名前が変更されているので、注意。
言い換えると、purge に書くはずだったものは content に書く。

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}

```

---

<span id="write_vite_config"></span>

[目次](#index)

## 3. vite.config.js

### 3.1. 目指すツリー構造

```bash
tree -I node_modules -I dist
.
├── README.md
├── package-lock.json
├── package.json
├── public/
├── *.code-workspace
├── src/
│   ├── App.jsx
│   ├── index.css
│   ├── index.html
│   └── main.jsx
├── tailwind.config.js
└── vite.config.js
```

### 3.2. vite.config.js

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react-swc';
import path from 'path';
// tailwindcss と autoprefixer を使うため。
import tailwindcss from 'tailwindcss';
import autoprefixer from 'autoprefixer';

export default defineConfig({
  /**
   * root:
   * 規定値はプロジェクトルート(=vite.configのあるdir)。vite プロジェクト作成時、index.html がプロジェクトルートにあるので、規定値の設定でよい。
   * しかし、index.htmlを含めた全ソースを src ディレクトリに置きたいので、書き換える。エントリーポイントである index.html と同じ場所にしなければならない。
   */
  root : path.resolve(__dirname, 'src'),

  /**
   * base:
   * 規定値は '/'。
   * ソース中に記述する相対パスの先頭の文字となる。build 後の依存関係がおかしくなるので、'./'とする。
   */
  base: './',

  /**
   * publicDir:
   * public ディレクトリのパスを指定。
   * 規定値は単に 'public'。root(=上記で設定) 起点の相対パスとしての'public'なのだが、
   * root を変更したなら、絶対パスとして public dir を指定しなおさねばならない。
   * なお、public ディレクトリの内容物は、ビルド後の dist ディレクトリ直下にコピーされる。
   */
  publicDir: path.resolve(__dirname, 'public'),

  css: {
    postcss: {
      /**
       * css.postcss.plugins[]:
       * tailwindcss, autoprefixer の導入。
       * 冒頭の import で、node_module 内のjsファイルを読んでおく必要あり。 plugins 配列内で require('tailwindcss') と書く設定方法は効かない。
       */
      plugins: [
        tailwindcss,
        autoprefixer
      ],
    },
  },


  build: {
    /**
     * build.outDir:
     * 規定値 = 'dist'。root がプロジェクトルートである場合の相対パスとしての 'dist' である。
     * root を変更するので、root起点の相対パスではなく、絶対パスとして dist ディレクトリのパスを指定しなおしている。
     */
    outDir: path.resolve(__dirname, 'dist'),

    /**
     * emptyOutDir:
     * build 時に outDir の中身を削除するか否か。true なら削除。
     * なお、outDir(=dist) が root 傘下にある場合(=初期状態)、emptyOutDir は自動的に true 扱いとなる。
     * 今回は、root を変更することにより、outDir が root の外に置かれることになったので、明示的に true にする必要がある。
     */
    emptyOutDir: true,

    /**
     * minify:
     * 値は false | 'esbuild'。 false だとミニファイしない。
     */ 
    minify: 'esbuild',

    rollupOptions: {
      // 複数の html ファイルを使う場合、ここに全てのhtmlのパスを書く。key 名は、そのhtmlに対応したjsファイルの名前でもある。
      input: {
        main: path.resolve(__dirname, 'src/index.html'),
      }
    }
  },

  // react
  plugins: [react()],

  resolve: {
    // ソース上でパスを書く時に使えるエイリアスをここに設定しておく。
    alias: {
      '@src' : path.resolve(__dirname, 'src'),
    }
  },
})
```
