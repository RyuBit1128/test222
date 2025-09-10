# AIチャットボット 開発メモ

プロジェクトの方向性と初期設計を素早く共有するためのメモ。随時更新していく。

## 概要
- 目的: ユーザーの質問に自然言語で応答するAIチャットボットを提供
- 提供形態: Webアプリ（将来的にAPI/Slack, Discord連携）
- スコープ（初期）: FAQ回答 + 簡単なタスク補助（要約、文章整形）

## 目標（SMART）
- 2週間でMVPを公開: シンプルUI + 会話履歴 + 既存モデル接続
- 応答時間: 3秒以内（サーバ側TTFB目安）
- 満足度: 内部ユーザ評価で4.0/5以上

## 想定ユーザー
- 一般ユーザー: 文章作成や調べ物の効率化
- 社内利用者: ナレッジ検索、定型文生成、議事録の要約

## コア機能（MVP）
- 会話UI（送信/停止/再生成）
- 会話履歴の保持（ローカル優先 → サーバ保存に拡張）
- モデルへのプロンプト送信（ストリーミング）
- システムプロンプト/トーンの切替（丁寧/カジュアル/厳密）
- 簡易セーフティ（NGワード/長文ガード）

## 将来機能（候補）
- ツール呼び出し（Web検索、計算、コード実行のサンドボックス）
- RAG（社内/指定コーパスからの検索・引用）
- マルチモーダル（画像→説明/要約）
- エクスポート（Markdown/HTML/テキスト）
- 外部連携（Slack, Discord, Notion, Google Drive）

## アーキテクチャ（ラフ）
```
Client (React/Vue/Svelte)
  └─ WebSocket/Fetch -> API Gateway
                        └─ LLM Provider SDK (OpenAI/Local)
                        └─ (Option) RAG: Vector DB + Retriever
                        └─ (Option) Tools: Search/Calc/Funcs
DB: (MVPは不要 or KV/SQLiteで履歴保存)
```

## 技術スタック候補
- フロント: Next.js or Vite + React, Tailwind CSS
- バックエンド: Node.js（Express/Fastify）or Python（FastAPI）
- モデル接続: OpenAI API 互換のクライアント（後で差し替え可能に）
- データ: SQLite/Prisma（MVPではローカル保存でも可）

## APIデザイン（MVP）
- `POST /api/chat` 入力: `{ messages: ChatMessage[], config? }`
  - 応答: ストリーミング（SSE/WebSocket）でトークン逐次配信
- `GET /api/health` 監視用

### ChatMessage 例
```json
{
  "role": "user",
  "content": "会議の議事録を要約して",
  "meta": { "lang": "ja" }
}
```

## プロンプト設計（初期）
- システム: 「あなたは有能で簡潔な日本語アシスタント。根拠を明示し、曖昧な点は質問で確認する。」
- ガードレール: 個人情報/機密の扱い、禁止事項の明記、参照不可時の回答方針

## セキュリティ/法務
- ログに機微情報を保存しない（マスキング/オプトアウト）
- モデルプロバイダのデータ保持ポリシーを確認
- 利用規約/プライバシーポリシーの雛形作成

## 開発ロードマップ
1. 画面モック作成（送受信/履歴/設定）
2. APIスタブ + ストリーミング実装
3. モデル接続（環境変数/キー管理）
4. エラーハンドリング/リトライ/タイムアウト
5. 最小のRAG PoC（任意）
6. デプロイ（Render/Vercel/Fly.io 等）

## 成功指標（MVP）
- 1会話あたり離脱率 < 30%
- 24h稼働率 > 99%
- 重大例外（5xx）< 0.5% リクエスト

## TODO（チェックリスト）
- [ ] UIモック（Figma/紙）
- [ ] `/api/chat` スタブ + SSE
- [ ] OpenAI互換SDK接続（APIキーを`.env`で管理）
- [ ] ストリーミング表示（トークン逐次）
- [ ] 会話履歴（ローカルのみで開始）
- [ ] システムプロンプト切替UI
- [ ] 簡易セーフティ
- [ ] 最小デプロイ

## サンプル環境変数（案）
```
OPENAI_API_KEY=sk-...
MODEL=gpt-4o-mini
RUNTIME_ENV=development
```

## ディレクトリ構成（例）
```
app/
  pages/
  components/
  lib/
  api/
server/
  index.ts
  routes/chat.ts
  services/llm.ts
  services/rag.ts (optional)
```

## 開発メモ
- 日本語最適化のため、回答は簡潔+要点の箇条書きを基本
- ストリーム切断時の再接続（指数バックオフ）
- 「わからない」と言える方針を維持

---
このファイルは初期叩き台。進捗に合わせて更新していく。
