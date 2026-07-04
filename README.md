# awesome-ai-agents

Google ADK を使って AI エージェントを作るためのサンプル集です。現在は、Gemini を使う基本版エージェント、Google Trends を MCP ツールとして接続する拡張版エージェント、OpenAI 互換モデルを LiteLLM 経由で使う版を収録しています。

## 収録プロジェクト

| ディレクトリ | 内容 | 主なファイル |
| --- | --- | --- |
| `build-ai-agent-google-adk/` | Google ADK の基本的なマルチエージェント構成。トピックからアウトラインを作り、記事本文を生成します。 | `blogger/agent.py` |
| `build-mcp-agent-google-adk/` | 基本版に MCP サーバーを追加し、Google Trends の関連クエリを参考に記事の切り口を決めます。 | `blogger/agent.py`, `blogger/server.py` |
| `build-openai-agent-google-adk/` | ADK の `LiteLlm` connector を使い、OpenAI 互換モデルで同じブログ生成ワークフローを動かします。 | `blogger/agent.py` |

## リポジトリ構成

```text
.
├── README.md
├── LICENSE
├── build-ai-agent-google-adk/
│   ├── README.md
│   ├── requirements.txt
│   └── blogger/
│       ├── __init__.py
│       └── agent.py
├── build-mcp-agent-google-adk/
    ├── README.md
    ├── requirements.txt
    └── blogger/
        ├── agent.py
        └── server.py
└── build-openai-agent-google-adk/
    ├── README.md
    ├── requirements.txt
    └── blogger/
        ├── __init__.py
        └── agent.py
```

## 前提条件

- `uv`
- Gemini 版を動かす場合は、Google AI Studio などで発行した Gemini API キー
- OpenAI 互換モデル版を動かす場合は、OpenAI API キー

各プロジェクトは独立しており、`blogger/agent.py` に PEP 723 形式の依存関係 metadata を持っています。`uv run --script` を使うと、仮想環境の作成や `pip install` を手動で行わずに起動できます。

## 基本版エージェントの使い方

```bash
cd build-ai-agent-google-adk
```

同じディレクトリに `.env` を作成します。

```text
MODEL=gemini-flash-latest
GOOGLE_API_KEY=your_actual_api_key
```

ADK Web UI を起動します。

```bash
uv run --script blogger/agent.py
```

表示されたローカル URL（通常は `http://127.0.0.1:8000`）をブラウザで開き、例として次のようなプロンプトを入力します。

```text
Write a technical blog post about building reliable AI agents.
```

このエージェントは、`BlogPlanner` でアウトラインを作成し、検証ループを通したあと、`BlogWriter` で記事本文を生成します。

## MCP 連携版エージェントの使い方

```bash
cd build-mcp-agent-google-adk
```

`.env` を作成します。

```text
MODEL=gemini-flash-latest
GOOGLE_API_KEY=your_actual_api_key
```

ADK Web UI を起動します。

```bash
uv run --script blogger/agent.py
```

この版では、`blogger/server.py` が stdio MCP サーバーとして動作し、`trends` ツールを提供します。エージェントは記事作成前に Google Trends の関連クエリを取得し、注目されている切り口を踏まえてアウトラインと本文を生成します。

例:

```text
Write a blog post about AI coding agents using current trend signals.
```

MCP サーバーを手動で起動する必要はありません。`blogger/agent.py` の `MCPToolset` が必要に応じて `blogger/server.py` を起動します。

## OpenAI 互換モデル版エージェントの使い方

```bash
cd build-openai-agent-google-adk
```

`.env` を作成します。

```text
MODEL=openai/gpt-5.5
OPENAI_API_KEY=your_actual_api_key
```

ADK Web UI を起動します。

```bash
uv run --script blogger/agent.py
```

この版では、ADK の `LiteLlm` connector を使って OpenAI 互換モデルを呼び出します。OpenAI 以外の互換エンドポイントを使う場合は、LiteLLM が期待する環境変数に合わせて `MODEL` や `OPENAI_API_BASE` を調整してください。

## 実装のポイント

- 各 `blogger/agent.py` は PEP 723 の inline script metadata で依存関係を宣言しています。
- `LoopAgent` を使い、アウトライン作成と本文作成に簡単な検証ループを入れています。
- Planner と Writer は `AgentTool` として root agent から呼び出されます。
- MCP 版では `pytrends` を使い、関連クエリが取得できない場合は daily trends、realtime trends、キーワードそのものへフォールバックします。
- MCP 版の `blogger/server.py` も PEP 723 形式で単独起動でき、通常は `agent.py` から同じ uv 環境の Python で起動されます。
- OpenAI 互換モデル版では `google.adk.models.lite_llm.LiteLlm` に `MODEL` を渡します。
- モデル名は `.env` の `MODEL` で切り替えられます。

## トラブルシューティング

- `uv: command not found` が出る場合は、先に uv をインストールしてください。
- `adk: command not found` が出る場合は、`adk web` を直接実行していないか確認し、`uv run --script blogger/agent.py` で起動してください。
- API キー関連のエラーが出る場合は、実行中のプロジェクトディレクトリに `.env` があるか、Gemini 版では `GOOGLE_API_KEY`、OpenAI 互換モデル版では `OPENAI_API_KEY` が正しいか確認してください。
- MCP 版で Google Trends から `429 Too Many Requests` が返ることがあります。実装にはフォールバックが入っているため、関連クエリが取れない場合でも記事生成は継続されます。

## 開発メモ

現時点では自動テストはありません。変更後は対象プロジェクトで `uv run --script blogger/agent.py` を起動し、簡単なブログ生成プロンプトで手動確認してください。MCP 周りを変更した場合は、`trends` ツールを使うプロンプトでフォールバックを含めて確認してください。

## ライセンス

Apache License 2.0 です。詳細は `LICENSE` を参照してください。
