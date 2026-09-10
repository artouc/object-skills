---
name: ogp
description: 検索結果と共有カードの見え方を整える。title、meta description、Open Graph、X Cards、og:image の設計、canonical との一致、本文との整合。OGP、OG image、Twitter Card、SNS 共有時のプレビュー、ページタイトルやディスクリプションの実装・修正・レビューを求められたときに使う。
---

# ogp

## 原則

検索結果と共有カードは、ページの内容を要約して外に出す表現。本文と食い違わせない。title、description、og:description、JSON-LD、本文の冒頭が別々のことを言っている状態を作らない。

すべてページ固有にする。テンプレートで同じ文言を量産しない。

## title

最も影響の大きいメタデータ。検索結果の見出し、ブラウザタブ、共有カードのフォールバックになる。

```html
<title>法人向け AI 議事録サービス | Example Inc.</title>
```

- 各ページで固有にする。
- ページ内容を先頭に寄せ、ブランド名は後ろに置く。表示が切れても主題が残る。
- キーワードを詰め込まない。主題を自然に書く。
- h1 と完全に同一である必要はないが、主題は一致させる。
- トップページ以外で会社名だけの title にしない。

## description

```html
<meta name="description" content="会議録の自動作成、要点抽出、タスク管理を一元化する法人向けサービス。">
```

- ページ固有の内容を書く。キーワードの列挙にしない。
- 検索エンジンが本文から別の抜粋を採用することはある。書いた通りに出る保証はない。
- 本文にない内容を書かない。
- 空にするくらいなら書かないほうがよい。中身のない定型文を全ページに入れない。

## Open Graph

Slack、Discord、Facebook、LinkedIn、iMessage など共有先の広い範囲で使われる。

```html
<meta property="og:type" content="website">
<meta property="og:site_name" content="Example Inc.">
<meta property="og:title" content="法人向け AI 議事録サービス">
<meta property="og:description" content="会議録の自動作成、要点抽出、タスク管理を一元化。">
<meta property="og:url" content="https://example.com/products/ai-minutes">
<meta property="og:image" content="https://example.com/og/ai-minutes.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="AI 議事録サービスの画面">
```

- og:url は canonical と完全に一致させる。ここがずれると共有のたびに別 URL が拡散する。url スキルを参照。
- og:type は website か article。記事なら article にし、article:published_time と article:modified_time を添える。
- og:title は title と同じでなくてよい。サイト名の重複を避けて主題だけにできる。
- 属性は property。name ではない。

## og:image

- 絶対 URL で指定する。相対 URL は解決されない。
- 認証、Cookie、Referer 判定、Bot 対策の背後に置かない。取得するのはクローラ。
- 1200x630 を基準にする。多くの共有先に収まる。
- og:image:alt を書く。
- 記事、商品、イベントごとに固有の画像を用意すると識別性が上がる。
- 画像内の文字は最小限にする。モバイルでは小さく表示され、切り取られることもある。
- 画像だけに重要な情報を載せない。同じ内容を本文にも置く。
- 動的生成する場合、生成の失敗時に壊れた画像や 404 を返さない。既定画像へフォールバックする。
- 画像を差し替えても URL が同じだと、共有先のキャッシュに古い画像が残る。更新時はファイル名を変える。

## X Cards

Open Graph だけで処理されることもあるが、X での表示を確実にしたいなら明示する。

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="法人向け AI 議事録サービス">
<meta name="twitter:description" content="会議録の自動作成、要点抽出、タスク管理を一元化。">
<meta name="twitter:image" content="https://example.com/og/ai-minutes.png">
<meta name="twitter:image:alt" content="AI 議事録サービスの画面">
```

- 属性は name。Open Graph の property と混同しない。
- card は summary_large_image か summary。画像がないのに summary_large_image にしない。
- Open Graph と内容を食い違わせない。同じ値でよいなら twitter:title と twitter:description は省略できる。

## 確定前に確認する

- title と description が全ページ固有か。テンプレートのまま残っていないか。
- og:url が canonical と一致しているか。
- og:image が絶対 URL で、無認証で 200 を返し、画像として表示できるか。
- og:image:alt と twitter:image:alt があるか。
- Open Graph が property、X Cards が name になっているか。
- title、description、og:description、本文の冒頭、JSON-LD の内容が矛盾していないか。
- 記事ページで og:type が article になり、公開日と更新日が本文の表示と一致しているか。
- 共有カードに出したい情報が、画像だけでなく本文テキストにもあるか。
