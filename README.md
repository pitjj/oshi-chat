# 推しとのトキメキ ✦

オタクユーザが推しキャラと会話できるブラウザアプリ。  
キャラの名前・一人称・口調・性格・関係性を設定して、Claude AIが推しとして会話してくれます。

## 機能

- キャラ設定（一人称・口調・性格・関係性のタグ選択 + 自由記述）
- 音声読み上げ TTS（ブラウザ内蔵 SpeechSynthesis API）
- 会話の保存・ロード（localStorage）

---

## デプロイ手順（Vercel）

### 1. リポジトリを用意する

```bash
git init
git add .
git commit -m "initial commit"
```

GitHubにリポジトリを作成してpushする：
```bash
git remote add origin https://github.com/YOUR_USERNAME/oshi-chat.git
git push -u origin main
```

### 2. Vercelにインポート

1. https://vercel.com にアクセスしてログイン
2. 「Add New Project」→ GitHubのリポジトリを選択
3. 「Import」をクリック

### 3. 環境変数を設定（重要）

Vercelのプロジェクト設定画面で：

```
Settings → Environment Variables
```

以下を追加：

| Name | Value |
|------|-------|
| `ANTHROPIC_API_KEY` | `sk-ant-...` |

### 4. デプロイ

「Deploy」ボタンを押すだけ。  
数分で `https://your-project.vercel.app` が発行されます。

---

## デプロイ手順（Netlify）

### Netlify Functionsへの変換

`api/chat.js` をNetlify Function形式に変換する場合：

1. `netlify/functions/chat.js` を作成：

```js
const Anthropic = require('@anthropic-ai/sdk');

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

exports.handler = async (event) => {
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  const { system, messages } = JSON.parse(event.body);

  const response = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1000,
    system,
    messages,
  });

  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(response),
  };
};
```

2. `index.html` の `API_ENDPOINT` を変更：
```js
const API_ENDPOINT = '/.netlify/functions/chat';
```

3. `netlify.toml` を作成：
```toml
[build]
  functions = "netlify/functions"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Netlify環境変数

```
Site Settings → Environment Variables
ANTHROPIC_API_KEY = sk-ant-...
```

---

## ローカル開発

```bash
npm i -g vercel
vercel dev
```

`.env.local` を作成：
```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

ブラウザで `http://localhost:3000` を開く。

---

## ファイル構成

```
oshi-chat/
├── index.html        # フロントエンド（アプリ本体）
├── api/
│   └── chat.js       # Vercel Edge Function（APIキープロキシ）
├── vercel.json       # Vercel設定
└── README.md
```
