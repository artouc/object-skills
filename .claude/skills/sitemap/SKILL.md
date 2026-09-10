---
name: sitemap
description: XML サイトマップを設計・生成・運用する。掲載する正規 URL の選別、lastmod の正確な管理、50000 URL と 50 MB の上限による分割、sitemap index、robots.txt での宣言、更新ワークフローへの組み込み。sitemap.xml、サイトマップ、クロール・インデックスの改善、lastmod、Search Console への送信に関する実装・修正・レビューを求められたときに使う。
---

# sitemap

## 原則

サイトマップは URL の一覧表ではない。検索結果に出す意思がある正規 URL だけを、実際の重要更新日時とともに、更新単位ごとに管理したもの。

成果を左右するのは仕様への準拠より、掲載 URL の選別と lastmod の正確性の2点。

発見を助けるだけで、インデックス登録や上位表示は保証しない。孤立ページをサイトマップだけで救済しようとせず、カテゴリ、パンくず、関連記事、HTML リンクでクローラとユーザーの両方が到達できる構造にする。

- https://www.sitemaps.org/protocol.html
- https://developers.google.com/search/docs/crawling-indexing/sitemaps/build-sitemap

## 手順

1. 掲載対象を決める。インデックスさせたい正規 URL だけを選ぶ。
2. lastmod の供給源を決める。コンテンツの実質更新日時を持つカラムを特定する。なければ先にそれを作る。
3. 更新頻度の違いで分割する。件数で機械的に割らない。
4. sitemap index で束ねる。
5. robots.txt に宣言し、Search Console に送信する。
6. 公開・更新・削除・リダイレクト・noindex 化のワークフローにサイトマップ更新を組み込む。

## 掲載しない URL

- noindex を返すページ。
- robots.txt でクロールをブロックしたページ。
- canonical が別 URL を指す重複ページ。
- 3xx、4xx、5xx を返す URL。
- セッション ID、並び替え、絞り込み、計測用パラメータで量産される URL。
- ログイン必須、下書き、サイト内検索結果、薄いタグやフィルター結果ページ。

canonical、noindex、robots.txt、HTTP ステータスと矛盾させない。販売終了や募集終了でインデックス対象外にするなら、サイトマップからも削除し、返すステータスと整合させる。廃止 URL の扱いは errorpage スキルを参照。

## 制約

- 1ファイル最大 50,000 URL、かつ非圧縮で 50 MB 以下。どちらかを超えたら分割して sitemap index で束ねる。
- sitemap index も最大 50,000 件の子サイトマップを参照できる。
- URL は完全な絶対 URL。相対 URL や URL エンコード不備を避ける。
- UTF-8 で出力し、& < > " ' を適切にエスケープする。
- .xml.gz で配信できるが、50 MB の判定は解凍後のサイズ。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/articles/foo</loc>
    <lastmod>2026-09-05T02:23:00+09:00</lastmod>
  </url>
</urlset>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://example.com/sitemap-posts-2026-09.xml</loc>
    <lastmod>2026-09-05</lastmod>
  </sitemap>
</sitemapindex>
```

## lastmod

任意だが、適切に管理できるなら最も価値のある属性。実態と一貫して一致している場合に利用され、不正確な値は信頼されなくなる。

- サイトマップの生成日時ではなく、そのページが最後に重要な変更を受けた日時を入れる。
- W3C Datetime 形式。日付だけなら 2026-09-05、時刻を含めるなら 2026-09-05T02:23:00+09:00。
- 重要な変更にあたるもの。本文の更新、重要なリンクの変更、構造化データの追加や変更。
- 重要な変更にあたらないもの。CSS、共通ヘッダーやフッター、軽微な表記修正、解析タグの入れ替え。これらを理由に全ページの日時を更新しない。
- 正確な日時を判定できないページは、推測値を入れるより省略する。
- 毎日再生成すること自体は問題ない。全 URL の lastmod を毎日「今日」にすることが問題。

DB では少なくとも次の3つを区別する。

- published_at — 初回公開日時。
- content_updated_at — 本文と主要情報の実質更新日時。lastmod にはこれを使う。
- system_updated_at — キャッシュ更新、同期、テンプレート変更を含む技術的更新日時。これを流用すると実態のない一斉更新を送ることになる。

changefreq と priority は仕様にあるが順位を上げる機能ではない。正規 URL の選別と信頼できる lastmod のほうがはるかに重要。

## 分割

件数で機械的に割るのではなく、更新性と監視性に合わせる。更新の多いファイルだけが頻繁に変われば、エラーの切り分けもしやすい。

- sitemap-pages.xml — 固定ページ。
- sitemap-posts-2026-09.xml — 当月公開の記事。
- sitemap-products-a.xml — 商品群。
- sitemap-categories.xml — カテゴリとハブページ。
- sitemap-images.xml — 画像が検索流入に重要な場合。
- sitemap-index.xml — 以上を束ねる親。

サイト種別ごとの勘所。

- ニュース・メディア — 日別に追記し、訂正や大幅追記があった記事だけ lastmod を変える。
- EC・マーケットプレイス — 在庫や価格の変動だけを重要更新にしない。説明、構造化データ、画像の実質更新を重視する。
- 求人・不動産・旅行 — 掲載終了ページをいつ除外・リダイレクト・noindex にするかをサイト方針と一致させる。
- SaaS・ドキュメント — 廃止バージョン、重複翻訳、下書きを混ぜない。製品別、言語別、バージョン別に分ける。
- 大規模ブログ — リライトしたページだけ lastmod を更新する。テンプレート変更で全ページを一斉更新しない。
- UGC・コミュニティ — 薄いタグページ、低品質プロフィール、パラメータ URL を無差別に載せない。
- 多言語・国際サイト — canonical、hreflang、掲載 URL を整合させ、言語別・国別に分ける。

## 通知

robots.txt に書く。

```txt
Sitemap: https://example.com/sitemap-index.xml
```

Search Console に sitemap index を送信すると、取得エラー、URL 検出数、処理状況を監視できる。

Google の sitemap ping endpoint は廃止されている。呼ばない。更新を伝える手段は、robots.txt の宣言、Search Console での管理、正確な lastmod、通常の内部リンク構造。

## 確定前に確認する

- 正規化済みで 200 を返す URL だけを載せているか。
- canonical、noindex、robots.txt、HTTP ステータスと矛盾していないか。
- lastmod がコンテンツの重要更新時だけ変わるか。全ページに当日の日付を一律付与していないか。
- 50,000 URL と 50 MB（非圧縮）の上限を超えていないか。超えたら分割したか。
- 更新頻度が異なる URL 群を別ファイルに分けたか。
- sitemap-index.xml を robots.txt と Search Console の両方で管理しているか。
- 公開、削除、リダイレクト、noindex 化のワークフローにサイトマップ更新が入っているか。
- 取得できないファイル、送信 URL とインデックス登録 URL の差を定期的に見ているか。
