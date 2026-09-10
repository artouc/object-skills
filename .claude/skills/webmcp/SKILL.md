---
name: webmcp
description: WebMCP でページの機能を AI エージェントに公開する。document.modelContext.registerTool、入力スキーマの最小化、readOnlyHint と untrustedContentHint と consequentialHint、exposedTo によるクロスオリジン公開、プロンプトインジェクション対策、AbortSignal での登録解除と中断。WebMCP、modelContext、registerTool、ページをエージェントから操作可能にする実装・修正・レビューを求められたときに使う。
---

# webmcp

## 前提

WebMCP はページの機能を構造化してエージェントに伝えるブラウザ API。既存のバックエンド認可を置き換える仕組みではない。認可、課金、データ保護、副作用の確定は、従来どおりサーバー側の信頼境界で完結させる。

提案段階の仕様で、API 形状も互換性も変わり得る。特定ブラウザの挙動に依存する設計にしない。人間向けの通常 UI と既存 API を必ず残し、WebMCP が使えなくても機能する段階的強化として入れる。

- 仕様 https://webmachinelearning.github.io/webmcp/
- 実装ガイド https://developer.chrome.com/docs/ai/webmcp
- セキュリティ https://developer.chrome.com/docs/ai/webmcp/secure-tools

## 登録

```js
const controller = new AbortController();

document.modelContext.registerTool(
  {
    name: "update_display_name",
    title: "表示名を変更",
    description: "ログイン中のユーザーの表示名を変更する。他の項目は変更しない。",
    inputSchema: {
      type: "object",
      properties: {
        displayName: { type: "string", maxLength: 50 }
      },
      required: ["displayName"]
    },
    annotations: { readOnlyHint: false },
    async execute(input, options) {
      const res = await fetch("/api/profile/display-name", {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ displayName: input.displayName }),
        signal: options.signal
      });
      if (!res.ok) throw new Error("表示名を変更できませんでした。");
      return { ok: true };
    }
  },
  { signal: controller.signal }
);
```

- name は 1〜128 文字、英数字と _ - . のみ。一意にする。
- execute は Promise を返す。解決値は JSON として直列化できるデータにする。
- execute の第2引数に AbortSignal が入る。fetch や長時間処理へ渡し、中断可能にする。
- 登録時の options.signal を abort するとツールが登録解除される。
- ログアウト、権限変更、画面遷移に合わせて登録解除する。ログイン前提のツールを匿名状態に残さない。

## ツールの粒度

1ツールを1つの明確な業務操作に絞る。エージェントが選び間違えず、安全に呼べるようにする。

まとめない。

- manage_account に、プロフィール変更、パスワード更新、退会、課金変更を詰め込む。

分ける。

- get_profile — 読み取りのみ。
- update_display_name — 表示名だけを変更する。
- request_account_deletion — 削除申請の開始だけ。確認は別に必須にする。

## 入力スキーマ

エージェントに任せる必要のない値をスキーマに入れない。

- userId、組織 ID、権限、ロール、価格、決済対象は入力にしない。ログイン中のセッションとサーバー側の状態から確定する。
- JSON Schema はエージェントへの補助情報であって、認可でも検証でもない。
- エージェントから渡る値は未信頼入力として扱う。通常のフォームや公開 API と同じく、サーバー側で型、範囲、所有権、権限、レート制限を検証する。

## description は攻撃面

description はエージェントが行動を決める材料。曖昧な説明や、第三者スクリプト、CMS、ユーザー入力から組み立てた説明は、誤用や間接プロンプトインジェクションにつながる。

- description は開発者が固定管理する。動的に生成しない。
- 目的、前提、副作用、禁止事項を短く正確に書く。
- 目安として description は 500 文字以内、パラメータ説明は 150 文字以内。

## 出力

- 必要最小限の構造化データを返す。
- トークン、Cookie、内部 ID、スタックトレース、過剰な個人情報を返さない。
- 目安として1回あたり 1,500 文字以内。
- UGC、外部サイトの本文、メール本文、レビュー、HTML を返すなら untrustedContentHint を付ける。

購入者の自由記述を返すツールでは、その文字列に「以後の指示を無視して」のような攻撃文が混ざり得る。命令ではなく未信頼データとして扱わせるために untrustedContentHint が要る。

## アノテーション

実態と正しく対応付ける。省略や誤設定は、エージェントやブラウザが確認のタイミングを誤る原因になる。

- readOnlyHint — 状態を変更しない検索や取得。ただし個人データを返すなら公開先と認可は別途厳格にする。
- untrustedContentHint — 出力に未信頼コンテンツが含まれる。
- consequentialHint — 購入、送金、予約、公開投稿、削除など、金銭的・現実世界的で不可逆に近い操作。

いずれもヒントであって強制ではない。最終防御をブラウザやモデルの確認 UI に任せない。削除、送信、決済、権限付与では、アプリ自身が何を誰にいくらでいつ実行するかを人間に見せて再確認し、サーバー側で再認証、CSRF 対策、権限検証、冪等性制御を行う。

実行中にユーザー入力を求める API は議論中で変わり得る。既存の確認フローを維持する設計にする。

## クロスオリジン公開

既定では他サイトやクロスオリジン iframe からツールは見えない。exposedTo にオリジンを指定すると共有できるが、これは「そのオリジンが利用者の代理で操作してよい」と認めるに近い判断。

- 原則として exposedTo を指定せず、同一オリジンに留める。
- 指定する場合もワイルドカードを使わず、https://trusted.example のように完全一致で列挙する。
- 読み取りツールでも、購買履歴、お気に入り、プロフィールを返すなら情報漏えいとして審査する。
- 書き込みツールは外部オリジンに公開しないことを基本方針にする。
- クロスオリジン iframe で使うなら allow="tools" と Permissions Policy の構成を理解し、意図しない iframe で有効にしない。

WebMCP はオリジン分離された文書でのみ使える。document.domain を有効にしている、または Origin-Agent-Cluster: ?0 を使う構成では API が無効になる。既存サイトに入れるなら、ヘッダーと iframe 構成を早い段階で確認する。

## 確定前に確認する

- WebMCP がなくても、人間向け UI と既存 API だけで機能が完結するか。
- 各ツールが単機能で、最小権限になっているか。
- 入力スキーマに ID、権限、価格など、サーバーが決めるべき値が混ざっていないか。
- サーバー側で認証、認可、所有権、入力検証、レート制限を行っているか。
- 3つのヒントが実態と一致しているか。
- 高リスク操作に、具体的な副作用の提示、ユーザー確認、再認証、冪等キー、監査ログがあるか。
- exposedTo が最小限か。書き込みツールを外部に公開していないか。
- ログアウトや権限変更でツールを登録解除しているか。
- execute が AbortSignal を尊重し、中断できるか。
- 登録、実行、確認、失敗を監査ログに残し、想定外の連続実行や権限逸脱を検知できるか。
