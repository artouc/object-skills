---
name: robots
description: クロールと索引を制御する。robots.txt の書き方と限界、noindex と Disallow の使い分け、meta robots と X-Robots-Tag の使い分け、AI クローラの扱い。robots.txt、noindex、nofollow、クロール制御、検索結果から消したい、AI に学習させたくない、といった要求への対応や、それらの設定の修正・レビューに使う。
---

# robots

## 3つの層

役割が違う。取り違えが事故の大半を占める。

- robots.txt — 取得してよいかの意思表示。クロールの制御。
- meta robots — HTML ページ単位の索引・表示の制御。
- X-Robots-Tag — HTTP ヘッダー。head を持たないリソースにも使えるページ単位の制御。

robots.txt は取得を止める。meta robots と X-Robots-Tag は取得された後に索引を止める。目的が「検索結果から消す」なら後者。

## 最大の落とし穴

Disallow したページの noindex は読まれない。取得しないのだから head の指定も見えない。結果として、外部リンクがあれば URL だけが検索結果に残り続ける。

検索結果から消したいなら次の順で行う。

1. robots.txt の Disallow を外す。クロールを許可する。
2. noindex を返す。meta robots か X-Robots-Tag。
3. 再クロールされ、索引から消えるのを待つ。
4. 消えたことを確認してから、必要なら Disallow を足す。

robots.txt はアクセス制御でも機密保護でもない。Disallow に書いたパス自体が公開情報として露出する。秘密のパスを Disallow に列挙しない。守るべきものは認証で守る。

## robots.txt

ルート直下に置く。ホスト単位で別。サブドメインごとに必要。

```txt
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap-index.xml
```

- User-agent — 対象クローラ。* は個別指定のないもの全部。
- Disallow — 取得を避けてほしいパス。前方一致。
- Allow — Disallow の例外。
- Sitemap — 絶対 URL。User-agent グループの外に書く。複数可。
- Crawl-delay — 一部のクローラだけが解釈する。Google は解釈しない。

注意点。

- JavaScript、CSS、画像をブロックしない。ページを描画できず、内容も判断できなくなる。
- 一時的な全面ブロックを本番に残さない。ステージング用の Disallow: / が本番に出る事故が多い。
- User-agent ごとにグループは1つに統合される。同じ User-agent を複数箇所に書き分けない。
- Allow と Disallow が競合したら、より長い（具体的な）パスが勝つ。
- 未知の記述は無視される。効くと思い込まない。

## meta robots と X-Robots-Tag

```html
<meta name="robots" content="noindex, follow">
```

```http
X-Robots-Tag: noindex
```

- index / noindex — 検索結果に載せるか。
- follow / nofollow — ページ内リンクを辿るか。
- nosnippet — テキスト抜粋を出さない。
- max-snippet:-1 — 抜粋の長さを制限しない。
- max-image-preview:large — 大きい画像プレビューを許可。
- noarchive — キャッシュ表示を抑止。

HTML には meta robots、PDF・画像・CSV・動画・JSON など head を持たないリソースには X-Robots-Tag を使う。どちらも同時に指定するなら矛盾させない。

noindex にするページの典型。サイト内検索結果、絞り込みやソートのパラメータ URL、薄いタグページ、印刷用ページ、サンクスページ、下書き、ステージング。

noindex を付けたページは sitemap から外す。sitemap スキルを参照。

## AI クローラ

学習用クローラと、検索・回答生成のためのクローラを区別する。robots.txt で User-agent 単位に指定できるが、遵守は各事業者の方針に依存し、名前も方針も変わる。

```txt
User-agent: GPTBot
Disallow: /

User-agent: Google-Extended
Disallow: /
```

- Google-Extended は Googlebot とは別。これを止めても通常の検索インデックスには影響しない。逆に Googlebot を止めれば検索から消える。混同しない。
- 学習拒否は robots.txt の宣言であって技術的な強制力はない。契約や利用規約とは別の層。
- 全 AI クローラを止めれば AI 経由の露出も失う。事業判断として決める。技術的な既定値にしない。
- 対象クローラの名前は変化する。書いたまま放置せず、定期的に見直す。

llms.txt は robots.txt の代替ではない。取得可否を伝えるものではなく、内容の案内。

## 確認

```sh
curl -sS https://example.com/robots.txt
curl -sSI https://example.com/report.pdf | grep -i x-robots-tag
curl -sS https://example.com/some-page | grep -i 'name="robots"'
```

確認する。

- robots.txt が 200 で text/plain で返るか。HTML を返していないか。
- ステージング用の Disallow: / が残っていないか。
- 検索結果から消したいページが Disallow ではなく noindex になっているか。
- noindex のページが sitemap に載っていないか。
- JS、CSS、画像がブロックされていないか。
- Sitemap 行が絶対 URL で、実在するファイルを指しているか。
- サブドメインごとに意図した robots.txt が配信されているか。
