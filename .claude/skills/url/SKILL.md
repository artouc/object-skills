---
name: url
description: URL の正規化と関係の明示。canonical、hreflang と lang、301 と 308 と 302 と 307 の使い分け、パラメータ URL の扱い、正規 URL をサイト全体で揃える。canonical タグ、重複コンテンツ、リダイレクト、サイト移行、URL 設計、多言語や地域別ページの実装・修正・レビューを求められたときに使う。
---

# url

## 原則

同じ内容に複数の URL が存在するとき、どれが正規かを1つに決め、サイト内のすべての表現をそこへ揃える。canonical タグを付けるだけでは足りない。

正規 URL を指すべきもの。ここがバラバラだと canonical は信頼されない。

- link rel="canonical"
- 内部リンクの href
- sitemap の loc
- og:url
- JSON-LD の @id と url と mainEntityOfPage
- リダイレクトの Location
- ページネーションや絞り込みからの復帰先

canonical はヒントであって命令ではない。矛盾する信号があれば無視される。決めた正規 URL を全経路で一貫させることが唯一の実効手段。

## 正規化の対象

同一内容が別 URL になる典型。

- http と https。www 有無。
- 末尾スラッシュの有無。
- 大文字小文字の揺れ。
- index.html や default.aspx などのデフォルトファイル名。
- utm_source などのトラッキングパラメータ。
- 並び替え、絞り込み、表示件数のパラメータ。
- セッション ID。
- CMS が生成する同一記事への複数経路。ID 経由とスラッグ経由など。
- 印刷用ページ、AMP、埋め込み用ページ。

方針を決める。https、www 有無、末尾スラッシュはサイト全体で1つに固定し、それ以外は 301 で寄せる。canonical だけで済ませずリダイレクトできるものはリダイレクトする。

## canonical

```html
<link rel="canonical" href="https://example.com/products/ai-minutes">
```

- 絶対 URL で書く。
- 自分自身を指す self-canonical を全ページに入れる。パラメータ付きでアクセスされたときに効く。
- canonical 先は 200 を返す実在のページにする。リダイレクト先や 404 を指さない。
- noindex と canonical を同じページに併用しない。矛盾した信号になる。
- ページネーションの2ページ目以降を1ページ目に canonical しない。別のページとして扱う。
- 内容が実質的に異なるページ同士を canonical で束ねない。統合したいなら 301。

## リダイレクト

- 301 Moved Permanently — 恒久移転。メソッドが GET に変わり得る。
- 308 Permanent Redirect — 恒久移転。メソッドとボディを保持する。API や POST を伴う経路ではこちら。
- 302 Found — 一時的。メソッドが変わり得る。
- 307 Temporary Redirect — 一時的。メソッドを保持する。
- 410 Gone / 404 Not Found — 代替がない場合。errorpage スキルを参照。

守ること。

- 代替がないのにトップページへ一律リダイレクトしない。移動したと誤認させる。
- リダイレクトチェーンを作らない。旧 URL から最終 URL へ1ホップで飛ばす。移行を重ねるほど連鎖しやすい。
- ループを作らない。https 化と www 統一と末尾スラッシュ処理を別々の層で書くと起きやすい。
- JavaScript リダイレクトだけに依存しない。サーバー側で返す。
- 移行時は旧 URL から新 URL への対応表を作る。パターンで一括変換できない例外を洗い出す。
- 移行後も旧 URL のリダイレクトを長期間維持する。外部リンクとブックマークが残る。

## hreflang と lang

多言語・地域別ページの関係を示す。

```html
<html lang="ja">
<link rel="alternate" hreflang="ja" href="https://example.com/ja/">
<link rel="alternate" hreflang="en" href="https://example.com/en/">
<link rel="alternate" hreflang="x-default" href="https://example.com/">
```

- html lang はそのページ本文の主言語。BCP 47 言語タグ。
- hreflang は同じ内容の言語・地域別バージョンの関係。役割が違う。両方書く。
- 相互参照を揃える。ja が en を指すなら、en も ja を指す。片方向は無効。
- 自分自身への hreflang も含める。
- x-default は言語が一致しないときの受け皿。言語選択ページかデフォルト言語版を指す。
- 各言語版の canonical は自分自身を指す。他言語版を canonical にしない。
- 内容が別物なら無理に hreflang でペアにしない。翻訳ではなく別コンテンツなら結ばない。
- 自動言語判定でリダイレクトすると、クローラが1言語版しか見られなくなる。切り替えは提示にとどめ、URL は保持する。

## パラメータ URL

- 内容が変わらないパラメータ（トラッキング、セッション）— self-canonical で正規 URL へ寄せる。リンクを生成しない。
- 内容が絞り込まれるパラメータ（フィルタ、ソート、ページ番号）— 索引させたい組み合わせだけを残し、残りは noindex。robots スキルを参照。
- 組み合わせ爆発を放置しない。薄いページを無数に生む。
- パラメータの順序や有無で別 URL にならないよう、生成側で正規化する。

## 確認

```sh
# 最初の応答とチェーンを見る
curl -sS -o /dev/null -w '%{http_code} -> %{redirect_url}\n' http://example.com/old-path
curl -sSL -o /dev/null -w '%{num_redirects} hops, final %{url_effective}\n' http://example.com/old-path

# canonical と hreflang
curl -sS https://example.com/page | grep -iE 'rel="(canonical|alternate)"'
```

確認する。

- http、www 有無、末尾スラッシュの4通りが1ホップで正規 URL に着くか。
- canonical、内部リンク、sitemap、og:url、JSON-LD の URL が一致しているか。
- canonical 先が 200 を返すか。
- 恒久移転に 301 か 308 を使っているか。POST を伴う経路で 301 にしていないか。
- hreflang が相互参照になっているか。自分自身を含んでいるか。
- リダイレクトチェーンとループがないか。
- 移行前の URL がまだリダイレクトされているか。
