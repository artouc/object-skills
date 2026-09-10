---
name: favicon
description: サイトの favicon 一式を最少構成（favicon.svg / favicon.ico / apple-touch-icon.png + head 3行）で設計・生成・設置する。favicon、ファビコン、サイトアイコン、apple-touch-icon、タブアイコン、PWA アイコン、site.webmanifest、theme-color、ダークモード対応アイコンの作成・追加・修正を求められたときに使う。
---

# favicon

## 構成

公開ルート直下にこの3つを置く。サイズ違いの PNG を並べる方式は取らない。

```text
favicon.svg           # モダンブラウザ。1ファイルでライト/ダーク両対応
favicon.ico           # 16/32/48 を内包。互換用
apple-touch-icon.png  # 180x180、非透過。iOS ホーム画面用
```

```html
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="icon" href="/favicon.svg" type="image/svg+xml" sizes="any">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
```

- ICO を先、SVG を後。同等候補ならブラウザは後方の宣言を採る。
- favicon.ico はルート直下必須。link を読まず /favicon.ico を直接取得する要求がある。
- SVG は iOS ホーム画面には使われない。apple-touch-icon.png は別途必要。

## favicon.svg

ライト用・ダーク用を2枚作らない。内部 CSS で切り替える。

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64" role="img" aria-label="サイト名">
  <style>
    .bg { fill: #0f2f5f; }
    .mark { fill: #ffffff; }
    @media (prefers-color-scheme: dark) {
      .bg { fill: #ffffff; }
      .mark { fill: #0f2f5f; }
    }
  </style>
  <circle class="bg" cx="32" cy="32" r="30" />
  <path class="mark" d="M20 14h10v14h4V14h10v36H34V36h-4v14H20z" />
</svg>
```

SVG のメディアクエリはブラウザ差が残る。ICO と PNG はライト側デザインで固定する。

## 生成

favicon.svg から全部を作る。背景色はライト側の背景色に合わせる。

ImageMagick がある場合。

```sh
magick -background none -density 384 favicon.svg -define icon:auto-resize=16,32,48 favicon.ico
magick -background '#0f2f5f' -density 384 favicon.svg -resize 180x180 -flatten -alpha off apple-touch-icon.png
```

ない場合は npx（初回はネットワーク取得が走る）。

```sh
npx --yes -p sharp node -e '
const sharp=require("sharp"), s="favicon.svg";
for (const [out,n,bg] of [["apple-touch-icon.png",180,"#0f2f5f"],
                          ["_16.png",16,null],["_32.png",32,null],["_48.png",48,null]]) {
  let p=sharp(s,{density:384}).resize(n,n,{fit:"contain",background:{r:0,g:0,b:0,alpha:0}});
  if (bg) p=p.flatten({background:bg});
  p.png().toFile(out);
}'
npx --yes png-to-ico _16.png _32.png _48.png > favicon.ico && rm -f _16.png _32.png _48.png
```

- density 384 が要る。低いと拡大時にぼやける。
- flatten / -alpha off が apple-touch-icon の非透過要件。
- ICO には3枚すべて渡す。1枚だと単一サイズになる。
- ImageMagick 内蔵 MSVG は style 内 CSS を無視することがある。崩れたら rsvg-convert で PNG 化してから渡す。

## デザイン制約

- 16px で識別できること。線・面は太く、要素数は最小限。
- 文字は不向き。頭文字1字か記号化したマーク。
- 白一色ロゴを透過背景で書き出さない。明るい UI 上で消える。背景面を必ず敷く。
- apple-touch-icon は透過なし、四角いっぱいに背景色。iOS が角丸マスクを掛けるので余白 10〜15%。
- 16px で潰れるなら、線を太くした 16px 専用 SVG を作り ICO 生成時のみ差し替える（配信はしない）。

## 配置先

配信時に /favicon.ico で引ける場所。判定できないときは推測せず、既存の og-image.png や robots.txt の場所に合わせる。

- Vite / CRA / 素の HTML — public/、index.html
- Next.js App Router — public/、app/layout.tsx。または app/ に icon.svg と apple-icon.png（link 不要、ただし app/favicon.ico も置く）
- Next.js Pages Router — public/、pages/_document.tsx
- Astro — public/、レイアウトの head
- Nuxt — public/、nuxt.config の app.head.link
- SvelteKit — static/、src/app.html
- Rails — public/、application.html.erb

## PWA のみ

インストール可能にする場合だけ追加する。SVG では代替できない。

```html
<link rel="manifest" href="/site.webmanifest">
<meta name="theme-color" content="#0f2f5f">
```

```json
{
  "name": "サイト名",
  "short_name": "サイト名",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ],
  "theme_color": "#0f2f5f",
  "background_color": "#ffffff",
  "display": "standalone"
}
```

icon-192.png と icon-512.png は上の生成コマンドにサイズを足して作る。

## 検証

```sh
sips -g pixelWidth -g pixelHeight -g hasAlpha apple-touch-icon.png   # 180/180/no
node -e 'const b=require("fs").readFileSync("favicon.ico"),n=b.readUInt16LE(4);
for(let i=0;i<n;i++){const o=6+i*16;console.log((b[o]||256)+"x"+(b[o+1]||256))}'  # 16/32/48
curl -sI http://localhost:3000/favicon.ico | head -1                  # 200
```

- 古い favicon 宣言の消し残しがないこと。
- OS をダークにして favicon.svg が消えないこと。
- favicon は強くキャッシュされる。確認はシークレットウィンドウかハードリロードで行う。
