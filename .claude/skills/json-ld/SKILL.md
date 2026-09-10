---
name: json-ld
description: 構造化データを JSON-LD で実装する。schema.org の型選択、@id と sameAs によるエンティティの同定、Organization と Article と Product と BreadcrumbList、可視 HTML との整合、リッチリザルトの適格性。構造化データ、schema.org、リッチリザルト、SEO や AIEO 向けのマークアップの追加・修正・レビューを求められたときに使う。
---

# json-ld

## 位置づけ

JSON-LD は順位を上げる仕組みではない。検索エンジンや機械処理系に「このページは何か、誰が出しているか」を誤解なく渡す機械可読な補助層。

- 要件を満たせばリッチリザルトの適格性を得る。実装しなければ原則として対象になれない。適格性であって表示の保証ではない。
- 主題、著者、運営主体、商品、価格、公開・更新日時を一貫した形で示せる。
- 構造化データの手動対策はリッチリザルトの適格性を失わせるが、通常検索の順位そのものには影響しない。

AIEO については基礎整備であって主役ではない。JSON-LD を入れたから AI の回答に載る、引用される、推薦される、という因果はない。エンティティと関係を取り出す際の補助になるという間接的な寄与にとどまる。

投資対効果の順番を守る。良質で一次性のある本文、正しい HTML セマンティクス、クロール・インデックス可能な技術基盤、明確な著者・運営者情報が先。JSON-LD はその後。可視 HTML の意味づけは html スキルを参照。

Microdata や RDFa も受け付けられ、正しく実装されていれば検索上は同等。JSON-LD は可視 HTML と分離してネスト構造を表現でき、大規模運用でも保守しやすいため推奨形式。

- https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data
- https://developers.google.com/search/docs/appearance/structured-data/sd-policies
- https://schema.org/

## 手順

1. ページの主対象を1つ決める。記事なのか、商品なのか、店舗なのか。
2. それに対応する最も具体的な型を選ぶ。網羅ではなく特定。
3. 対象の型で必須とされるプロパティを満たす。
4. 推奨プロパティのうち、ページに実在する情報だけを足す。
5. @id で実体を同定し、サイト内で参照を揃える。
6. 可視 HTML、title、canonical と値が矛盾していないか照合する。
7. 検証する。

## 実装優先度

- 最優先 — 正しい HTML、固有の本文、明確な見出し、内部リンク、title、canonical、クロール可能性。JSON-LD ではなくこちら。
- 高 — Organization、WebSite、WebPage。運営者とサイトの同定。
- 高 — ページの主目的に合う型。Article、Product、Service、LocalBusiness など。
- 中 — Person、author、publisher、datePublished、dateModified。著者性と更新性。
- 中 — BreadcrumbList。階層構造。
- 条件付き — FAQPage、VideoObject、Event、JobPosting、Recipe。ページ上に実在し、対応機能の要件を満たす場合のみ。
- 条件付き — sameAs。公式 SNS、公式プロフィール、信頼できる外部識別子との接続。

商品、記事、組織、店舗、イベント、動画、求人など明確な実体を持つページでは優先度が高い。実体が曖昧なページに無理に付けない。

## 記述例

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "@id": "https://example.jp/articles/json-ld-seo",
  "mainEntityOfPage": "https://example.jp/articles/json-ld-seo",
  "headline": "記事タイトル",
  "inLanguage": "ja",
  "datePublished": "2026-09-05",
  "dateModified": "2026-09-05",
  "author": { "@type": "Person", "name": "著者名" },
  "publisher": { "@id": "https://example.jp/#organization" }
}
</script>
```

Organization はサイト全体で一貫した @id を持つ1つの定義に集約し、各ページの publisher からその @id を参照する。記事ごとに組織情報を書き下すと重複と同一性の混乱を招く。

同じ実体には安定した @id を使い、重複ページ間で識別子を揺らさない。@id には正規化した canonical URL を使う。

## 禁止と制約

- ページが実際に表示している内容を真実に表すこと。これが最上位の制約。
- ページに存在しない FAQ、レビュー、価格、在庫、著者、イベントを構造化データだけに作らない。
- 偽レビュー、ユーザーに見えないコンテンツ、無関係な型付け、古い価格、終了済みイベントを入れない。
- プロパティを埋め尽くさない。不完全・不正確な多数より、少数でも完全かつ正確な推奨プロパティ。
- 構造化データは記述対象のページ自身に置く。別ページの内容を代理で書かない。
- 画像 URL と構造化データをクローラが取得できるようにする。robots や認証で塞がない。
- CMS 更新時に本文と JSON-LD がズレないよう、同じデータソースから生成する。本文と JSON-LD を二重管理しない。
- 日付は ISO 8601。dateModified を更新しないまま本文だけ直さない。

## 検証

公開前に Rich Results Test で対象機能の要件を満たすか確認する。公開後は Search Console のリッチリザルト関連レポートと URL Inspection で、実際にクロールされた状態を確認する。

- 構文が通ることと、内容が正しいことは別。検証が通っても可視 HTML との矛盾は検出されない。目視で照合する。
- HTML と JSON-LD はそれぞれ検証する。
- エラーだけでなく警告も読む。警告は推奨プロパティの欠落を示すことが多い。

## 確定前に確認する

- ページの主対象に対応する最も具体的な型を選んだか。
- 必須プロパティを満たしたか。
- headline、価格、日付、著者、組織名が可視 HTML と一致しているか。
- @id と canonical URL が一致し、サイト内で揺れていないか。
- publisher が共通の Organization を参照しているか。
- ページに存在しない情報を書いていないか。
- 本文の更新時に JSON-LD も追随する仕組みになっているか。
