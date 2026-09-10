---
name: html
description: 意味論に基づいた HTML を書く。要素の選択、文書構造とランドマーク、見出し階層、リンクとボタンの区別、フォームと label、alt、データ表、time、ARIA の使いどころ、markuplint での検証。HTML やマークアップの新規作成・修正・レビュー、div/span の置き換え、アクセシビリティ改善を求められたときに使う。
---

# html

## 判断順序

この順序を逆転させない。これがこのスキルの中心。

意味 → ネイティブ HTML → アクセシビリティ → CSS → 補助的な ARIA → 検証

タグは視覚表現や CSS クラス名ではなく、コンテンツ本来の意味・関係・ユーザー操作で選ぶ。見た目は CSS で制御し、見た目を変えるために要素を選ばない。

## 一次ソース

- 仕様 https://html.spec.whatwg.org/multipage/
- 要素・属性・親子関係の索引 https://html.spec.whatwg.org/multipage/indices.html
- 検証 https://markuplint.dev

W3C 自身が WHATWG HTML Living Standard を現行の唯一の HTML 標準として案内している。W3C Wiki の古いタグ一覧や過去の HTML 勧告を authoring rules の根拠にしない。要素の content model、許容属性、親子関係に確信がないときは推測せず索引を引く。

## 文書構造

- main は文書の支配的な主内容。単一ページでは1つ。モーダルやカード内に置かない。
- header / footer はページまたはセクション・記事の導入部と補足。
- nav は主要ナビゲーションのまとまり。リンクが複数あるだけでは使わない。
- article は単独で再配布・引用・切り出しできる自己完結コンテンツ。記事、投稿、レビュー、コメント、商品カード。
- section は見出しを持つテーマ上の区画。レイアウト用ラッパーには使わない。
- aside は本文に対して補足的・間接的な内容。関連リンク、補足欄、広告枠。
- div は他に適切な意味要素がない場合の汎用フローコンテナ。
- html lang を設定し、埋め込み言語の範囲には個別の lang を付ける。値は BCP 47 言語タグ。

避けるべき典型。section を「意味がありそうな div」として多用する。全コンテナを section にする。main を複数置く。

## 見出しと本文

- h1〜h6 は文字の大きさではなく情報構造で選ぶ。h1 がページの主題、以降は論理的な階層。
- 見出しを小さく見せたいときは要素を p や div に変えず CSS で調整する。
- 段落は p。順序に意味がある列挙は ol、順不同は ul、用語と定義や名前と値の対は dl。
- br をレイアウトや余白目的で連続使用しない。詩や住所など改行自体が内容である場合に限る。
- 引用は blockquote、文中の短い引用は q、作品タイトルは cite、コードは code、整形済みテキストは pre。

## リンクとボタン

- 遷移先 URL がある操作は a href。
- 現在の画面・状態を変える操作は button type="button"。送信は type="submit"、リセットは type="reset"。
- onclick を持つ div や span をボタンにしない。
- a の中に button、button の中に a など interactive content を入れ子にしない。
- target="_blank" は新規タブが必要な場合のみ。
- リンクテキストは「こちら」「詳細」ではなく、単独で遷移先が分かる語にする。

## フォーム

- すべての入力欄に label for を付ける。placeholder を label の代わりにしない。
- 入力値の性質に合う input type を使う。email、tel、url、date、number など。
- 必須には required、該当する欄には autocomplete。パスワード、クレジットカード、住所は用途に合うトークンを使う。
- 関連するコントロールは fieldset と legend でグループ化する。
- エラーは対象フィールド、エラー本文、修正方法を結び付ける。色や枠線など視覚表現だけに依存せず、テキストでも示す。

## 画像・表・日時

- 意味を持つ画像の alt は画像の目的を書く。「画像」「写真」とだけ書かない。
- 装飾画像は alt=""。
- 複雑な図表は本文または近傍に、内容・結論・数値をテキストで書く。
- データ表は table、見出しセルは th、必要に応じ scope="col" / scope="row"、表題は caption。
- レイアウト目的や非表形式の情報に table を使わない。
- 人間向け表記と機械可読値を併記する日時は time datetime。値そのものの機械可読表現が要るなら data value。

## ARIA

ネイティブ HTML で実現できるなら ARIA role / state / property を追加しない。ARIA は HTML の置換手段ではなく、ネイティブ要素で表せない意味を補うもの。

```html
<!-- 不可 -->
<div role="button" tabindex="0" onclick="save()">保存</div>
<!-- 可 -->
<button type="button" onclick="save()">保存</button>
```

button にはキーボード操作、フォーカス、無効状態、支援技術への通知が標準で備わる。前者を同等にするには Enter/Space の処理、フォーカス表示、状態管理を手で実装することになる。

ARIA が要るのはネイティブ要素だけでは表せない複合コンポーネント。タブ UI、補完付きコンボボックス、ライブ更新通知など。それでも先に button、input、dialog、details、select で置き換えられないかを検討する。

## 検証

markuplint を使う。HTML を書いたら必ず通す。

```sh
npx --yes markuplint page.html 'src/**/*.html'
```

設定は .markuplintrc に置き、プリセットを継承する。

```json
{ "extends": ["markuplint:recommended"] }
```

- recommended プリセットは仕様適合とアクセシビリティの両方を見る。lang や alt の欠落、アクセシブルな名前の不足、h1 の不在、要素の入れ子違反などを検出する。
- 指摘を規則の無効化で黙らせない。仕様・UX・アクセシビリティの観点で直す。無効化するなら理由をコメントで残す。
- 問題があれば終了コード 1 を返す。CI と pre-commit に組み込む。
- Vue、JSX、Svelte、Pug などは対応するパーサを設定して原文のまま検査する。生成後の HTML を検査対象にしない。
- 検出がゼロでも意味論が正しいことの保証にはならない。上の各節の判断は自分で行う。

確定前に確認する。

- markuplint を実行し、検出事項を解消したか。
- lang、title、main、見出し階層、label、alt が揃っているか。
- リンクとボタンが意味どおりに使い分けられているか。
- interactive content の不正な入れ子がないか。
- キーボードだけで主要操作ができるか。
