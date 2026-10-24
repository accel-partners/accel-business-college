# Cloudflare テスト版 accel-business-college（Worker：accel-college-test）

college を OpenNext で Cloudflare Workers に載せたテスト版。ブランチ `cloudflare-test` で作り、確認後に main へ。
本番（Vercel・college.accel-dash.com）とステージング（Vercel・stg.college.accel-dash.com）には影響しない。2026-10-24 作成。

- テスト版の URL：`accel-college-test.signup-3e3.workers.dev`
- **Slack ログインはテスト版の URL では使えない**（Slack アプリに登録した戻り先が college.accel-dash.com のため）。テスト版で見られるのはログイン不要のページだけ。
- データは本番と同じ Supabase。**実在のメンバーの個人情報があるので、テスト版では書き込まない**。

## 変えたこと

- Next.js を 16.4.1 に（@opennextjs/cloudflare 1.20.12 に合わせる）。`cf:build` などのコマンドを追加。
- `wrangler.jsonc`・`open-next.config.ts` を追加。定期実行（Cron）は書かない。
- `vercel.json` で `cloudflare-test` ブランチを Vercel で作らないようにした（プレビューでは Slack ログインが使えないため）。
- 画面やログインの仕組み（`middleware.ts`・`src/lib/session.ts`）は変えていない。

## ビルドの設定（Cloudflare の Settings → Builds）

- Production branch：`cloudflare-test`（main にマージした後は `main`）
- Build command：`npm run cf:build`
- Deploy command：`npx wrangler deploy`
- Google Fonts（Noto Sans JP）をビルド時に取りに行く。ネットにつながらない場所ではビルドが止まる（Cloudflare のビルドでは取れる）。

## 変数の入れ場所（2か所あるので注意）

**Settings → Builds の「Variables and secrets」**（ビルドのとき）
- `NEXT_PUBLIC_SUPABASE_URL`、`NEXT_PUBLIC_SUPABASE_ANON_KEY`、`NEXT_PUBLIC_APP_URL`（Text）

**Settings の「Runtime variables and secrets」**（動くとき。画面のいちばん上）
- Variable：`NEXT_PUBLIC_SUPABASE_URL`、`SLACK_CLIENT_ID`
- Secret：`SUPABASE_SERVICE_ROLE_KEY`（Supabase の cloudflare_prod）、`SESSION_SECRET`、`SLACK_CLIENT_SECRET`、`SLACK_SIGNING_SECRET`、`SLACK_BOT_TOKEN`、`VIMEO_ACCESS_TOKEN`、`ANTHROPIC_API_KEY`
- 後回しでよい：`UNSPLASH_ACCESS_KEY`（画像検索だけ使えない）
- 秘密の鍵は本人が Cloudflare の画面に直接入力する。チャット・文書・GitHub には書かない。
- Slack の鍵は api.slack.com のアクセルビジネスカレッジBot の画面から「Show」で写す。**Regenerate は押さない**（Vercel 版が止まる）。
- `SESSION_SECRET` を Vercel と違う値にすると、切替のときにメンバー全員が一度ログアウトされる（Slack でもう一度ログインすれば戻る）。

## 切替と戻し方

- 切替：Worker → Domains → Add Domain → accel-dash.com、サブドメイン `college`。直後に Slack ログインと、Slack からの通知（/api/slack/events）を確かめる。
- 戻す：Domains から college.accel-dash.com を外す（DNS の * で Vercel に戻る）。

## 経緯

- 2026-10-24：cloudflare-test を作成。Cloudflare に Worker accel-college-test を作り、Supabase の鍵と SESSION_SECRET を入れた（Slack・Vimeo・Anthropic の鍵は切替の前に入れる）。
