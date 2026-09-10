---
name: well-known
description: /.well-known/ に何を置くべきかを判断し、正しく配信する。IANA 登録名の確認、security.txt、change-password、OpenID Connect や OAuth のメタデータ、assetlinks.json、mta-sts.txt、acme-challenge、webfinger。well-known、security.txt、ドメイン検証ファイル、ディスカバリエンドポイント、サイト直下の設定ファイルの追加・修正・レビューを求められたときに使う。
---

# well-known

## 原則

/.well-known/ はオリジン単位の固定パスで、URI 空間を汚さずに機械可読なメタデータを置くための場所。RFC 8615 が定める。

- 名前は IANA レジストリに登録されたものだけを使う。自分のアプリ設定を勝手な名前で置く場所ではない。
- 必ずオリジンのルート直下。https://example.com/.well-known/<suffix>。サブディレクトリ配下には置けない。
- ホスト名ごとに独立している。www 有無、サブドメイン、国別ドメインで別々に必要かを判断する。
- 名前は大文字小文字を区別する。
- 一覧 https://www.iana.org/assignments/well-known-uris
- 仕様 https://www.rfc-editor.org/rfc/rfc8615.html

## 置くかの判断

順に確認する。

1. 第三者の機械（クローラ、ブラウザ、認証クライアント、CA、メールサーバー、OS）が、事前の取り決めなしに固定パスで取りに来る必要があるか。ないなら置かない。
2. その用途の名前が IANA に登録されているか。登録名を使う。
3. 自サービスの利用者向け情報なら、通常の URL に置く。/.well-known/ は人間が読むページの置き場ではない。
4. 認証や秘密を要する情報を置かない。ここは常に無認証で公開される前提。

置く動機が「ルート直下に config を置きたい」なら、それは /.well-known/ の用途ではない。

## 通常のサイトで検討する価値があるもの

- security.txt（RFC 9116、登録済み）— 脆弱性の連絡先。公開サービスを運用しているなら最優先で置く。
- change-password（登録済み、暫定）— パスワード変更画面へのリダイレクト。パスワード認証があるなら置く。パスワードマネージャが使う。
- mta-sts.txt（RFC 8461、登録済み）— 受信メールの TLS ポリシー。独自ドメインでメールを受けるなら検討する。
- acme-challenge（RFC 8555、登録済み）— Let's Encrypt 等の HTTP-01 検証。証明書自動更新の経路として、リダイレクトや認証で塞がないことが重要。
- assetlinks.json（登録済み）— Android アプリとドメインの関連付け。App Links を使うなら必須。
- gpc.json（登録済み、暫定）— Global Privacy Control への応答方針。
- did.json（登録済み、暫定）— did:web を使う場合のみ。

## 認証・ID を提供する場合

自サービスが認可サーバーや OpenID Provider として振る舞うときだけ置く。ただの利用側なら不要。

- openid-configuration（登録済み）— OpenID Connect Discovery。
- oauth-authorization-server（RFC 8414、登録済み）— 認可サーバーメタデータ。
- oauth-protected-resource（RFC 9728、登録済み）— 保護リソースメタデータ。
- webauthn（登録済み）— WebAuthn 関連。
- webfinger（RFC 7033、登録済み）— アカウント単位のディスカバリ。
- host-meta（RFC 6415、登録済み）— ホスト単位のメタデータ。

これらは実装が仕様に追随している場合のみ置く。手書きの JSON を静的ファイルとして置き、鍵ローテーションやエンドポイント変更に追随できない状態にしない。

## 連合・分散サービス

該当するプロトコルを実装している場合のみ。

- matrix（登録済み）— Matrix のサーバー・クライアント発見。
- nodeinfo（登録済み、暫定）— 連合ソーシャルのインスタンス情報。

## 登録されていない名前

広く使われていても IANA に登録されていない名前がある。apple-app-site-association がその代表。実装上は必要なので置くが、レジストリ由来の名前と同じ扱いにしない。ベンダーの現行ドキュメントを確認してから配置する。

llms.txt や ai.txt は登録されていない。これらをルート直下に置く場合、/.well-known/ ではなくオリジン直下（/llms.txt）が現在の慣行。/.well-known/ に独自の名前を作らない。

## 配信要件

- HTTPS で配信する。
- 他オリジンへリダイレクトしない。取得側がリダイレクトを追わないことがある。
- 認証、Cookie、User-Agent 判定、Basic 認証、WAF、Bot 対策で塞がない。取りに来るのは人間のブラウザではない。
- robots.txt でブロックしない。
- Content-Type を正しく返す。text 系は security.txt と mta-sts.txt が text/plain; charset=utf-8、JSON 系は application/json、webfinger は application/jrd+json。
- webfinger など、ブラウザからのクロスオリジン取得を想定する仕様では Access-Control-Allow-Origin: * を付ける。
- 存在しない suffix には 404 を返す。SPA のフォールバックで HTML を 200 で返さない。errorpage スキルを参照。
- SPA やフレームワークのルーティングより前に静的配信する。/.well-known/ 配下がアプリのルーターに吸われていないか確認する。
- change-password は内容を持たず、パスワード変更画面へのリダイレクトとして応答する。
- キャッシュは中程度。連絡先や鍵の変更が反映されない事態を避ける。

## security.txt

```txt
Contact: mailto:security@example.com
Expires: 2027-01-01T00:00:00.000Z
Preferred-Languages: ja, en
Canonical: https://example.com/.well-known/security.txt
Policy: https://example.com/security-policy
```

- Contact と Expires は必須。Expires を過ぎたファイルは無効とみなされる。期限切れを放置しない。更新を運用カレンダーに入れる。
- Expires は1年程度を上限にする。
- Contact に個人の私用アドレスを書かない。退職や異動で連絡が届かなくなる。
- Canonical は実際の配信 URL と一致させる。
- 受信体制がないのに連絡先だけ置かない。届いた報告に応答できる運用を先に用意する。

## 確認

```sh
curl -sS -i https://example.com/.well-known/security.txt
curl -sS -o /dev/null -w '%{http_code} %{content_type} %{redirect_url}\n' \
  https://example.com/.well-known/change-password
curl -sS -o /dev/null -w '%{http_code}\n' https://example.com/.well-known/no-such-thing
```

確認する。

- ステータスが 200 で、Content-Type が実体と一致しているか。
- 他オリジンへリダイレクトしていないか。
- 未登録・未提供の名前に 404 が返るか。HTML の 200 が返っていないか。
- www 有無、対象サブドメイン、国別ドメインのそれぞれで必要な応答が返るか。
- security.txt の Expires が将来の日時か。
- 認証やアクセス制御の対象になっていないか。
