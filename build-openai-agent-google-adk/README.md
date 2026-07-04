# Google ADK で OpenAI 互換エージェントを構築する

このチュートリアルプロジェクトは、Gemini 版の例と同じブログ執筆エージェントの構成を使いながら、モデル呼び出しを ADK の `LiteLlm` コネクタ経由に切り替えます。`OPENAI_API_KEY` を使って OpenAI ホストのモデルを利用できるほか、LiteLLM が対応する OpenAI 互換エンドポイントにも接続できます。

ADK Python は初期リリースの `v0.1.0` から、LiteLLM 経由で Gemini 以外のモデルに対応しています。Gemini は ADK で最も直接的に統合された選択肢であり、LiteLLM は OpenAI、Anthropic、Cohere、ローカルの OpenAI 互換サーバーなどを使うための移植性レイヤーです。

## プロジェクト構成

- `blogger/agent.py`: ADK Web UI のエントリポイントとマルチエージェントワークフロー。
- `blogger/__init__.py`: ADK がパッケージを検出するための Python パッケージマーカー。
- `requirements.txt`: PEP 723 を使わない読者向けの従来型依存関係リスト。

## セットアップ

このプロジェクトディレクトリへ移動します。

```bash
cd build-openai-agent-google-adk
```

ローカルに `.env` ファイルを作成します。

```text
MODEL=openai/gpt-5.5
OPENAI_API_KEY=your_actual_api_key
```

`MODEL` の値は LiteLLM の provider/model 形式を使います。別の OpenAI 互換エンドポイントを使う場合は、`openai/` の provider 接頭辞を維持し、`OPENAI_API_BASE` など LiteLLM が期待する環境変数でエンドポイントを設定します。

## Gemini と他の LLM の使い分け

Google との統合を最も重視する場合は、Gemini を ADK から直接使います。Google AI Studio や Vertex AI の認証情報、Gemini 固有の機能、Google ホストのツール、または最もシンプルな ADK セットアップに依存するチュートリアルでは、通常 Gemini が第一候補です。Gemini モデルは `gemini-2.5-flash` のようなプレーンなモデル文字列として渡せるため、コネクタオブジェクトを作る必要がない場合が多いです。

モデルの移植性を重視する場合は `LiteLlm` を使います。このプロジェクトでは、各 `LlmAgent` に `LiteLlm(model=MODEL)` を渡しています。`MODEL` は LiteLLM の provider 接頭辞付き形式にする必要があります。

```text
MODEL=openai/<model-name>
```

OpenAI 互換ゲートウェイやセルフホストサーバーを使う場合は、LiteLLM が期待する provider 接頭辞を使い、`OPENAI_API_BASE` など provider 固有の環境変数で接続先を設定します。OpenAI 以外の provider を使う場合は、`ANTHROPIC_API_KEY` のような対応する API キーを設定し、その provider の LiteLLM モデル文字列を指定します。

## モデル互換性の注意点

ADK は各 `LlmAgent` に同じオーケストレーションのインターフェースを提供しますが、実際のモデルバックエンドの挙動は provider によって異なります。provider を変更したら、必ずワークフロー全体をテストしてください。

- **ツール呼び出し:** このプロジェクトでは planner と writer を agent tool として公開しています。関数呼び出しやツール呼び出しに強いモデルもあれば、より単純な指示や少ないツール候補が必要なモデルもあります。
- **構造化出力:** 検証ループは `ok` や `retry` のような厳密な文字列を期待します。指示追従が緩いモデルでは不要なリトライが起きるため、validator のプロンプトは短く厳密に保ちます。
- **ストリーミングとマルチモーダル:** Gemini、OpenAI 互換 API、ローカルモデルサーバーでは、LiteLLM 経由で利用できるストリーミング、音声、画像、ファイル処理の挙動が同一ではありません。選択したモデルで検証するまで、機能が使える前提にしないでください。
- **コンテキスト長:** 大きなコンテキストを持つモデルは outline や draft の状態を多く保持できますが、コストやレイテンシが増えることがあります。小さいモデルではプロンプトを短くし、リトライ回数も控えめにします。
- **レイテンシとレート制限:** LiteLLM は provider 抽象化を提供しますが、実際の制限は接続先 API やゲートウェイに依存します。ADK Web UI が停止したように見える場合やリトライが失敗する場合は、まず provider の rate limit エラーを確認します。
- **コスト管理:** 課金体系は provider ごとに異なります。マルチエージェントワークフローでは、planning、validation、writing、最終応答生成が別々の呼び出しになるため、1 回のユーザープロンプトで複数回モデルが呼ばれます。
- **モデル名:** LiteLLM のモデル識別子は provider の追加、名称変更、廃止に合わせて変わることがあります。モデルが解決できなくなった場合は、ADK の agent code ではなく `MODEL` を更新します。
- **シークレット:** API キーはローカルの `.env` に置き、コミットしないでください。選択した provider に必要な認証情報だけを設定します。

## 実践的な選択ガイド

ADK 自体を学ぶ、Google Cloud サービスを使う、または Google のドキュメント例に合わせることが目的なら Gemini から始めます。モデル品質を比較したい、既存の OpenAI 互換ゲートウェイを使いたい、社内プロキシ経由で接続したい、または agent graph を変えずに provider を差し替えたい場合は LiteLLM から始めます。

本番に近い例では、依存関係のバージョンを固定し、テスト済みのモデル文字列を README に記録し、provider を変更するたびに smoke test を実行します。

```text
AI コーディングエージェントを評価する方法について、技術ブログを書いてください。
```

planner が実行され、writer が実行され、validation が完了し、最終応答に代替タイトルと短い SNS 投稿文が含まれることを確認します。

## エージェントの実行

ADK Web UI を起動します。

```bash
uv run --script blogger/agent.py
```

ターミナルに表示されるローカル URL を開きます。通常は `http://127.0.0.1:8000` です。次のようなプロンプトを試してください。

```text
AI コーディングエージェントを評価する方法について、技術ブログを書いてください。
```

## 仕組み

- `LiteLlm(model=MODEL)` が、設定された OpenAI 互換モデルを ADK で使える形に変換します。
- `BlogPlanner` が outline を作成し、検証ループが弱い outline をリトライします。
- `BlogWriter` が最終記事を書き、検証ループが draft を確認します。
- root agent は planner と writer を tool として公開し、ワークフローを明示的に保ちます。

## トラブルシューティング

- 認証に失敗する場合は、このディレクトリに `.env` があり、`OPENAI_API_KEY` が設定されていることを確認してください。
- モデルが利用できない場合は、アカウントで利用可能な別の LiteLLM OpenAI モデル文字列を `MODEL` に設定してください。
- 依存関係のインストールに失敗する場合は、ネットワーク接続を確認してから再実行してください。`uv run --script` は PEP 723 の依存関係を自動解決します。
