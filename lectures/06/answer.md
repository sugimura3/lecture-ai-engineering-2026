### STEP 2 の [tool call]：
- glossary_search_glossary({'query': '3レベル'})

### STEP 3 の [tool call]：
- fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- fetch_fetch({'raw': True, 'url': 'https://modelcontextprotocol.io/'})
- fetch_fetch({'start_index': 5000, 'url': 'https://modelcontextprotocol.io/'})
- fetch_fetch({'url': 'https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro.md'})

### STEP 4 の [tool call]：
-  aws_aws___search_documentation({'search_phrase': 'Amazon S3 versioning overview', 'topics': ['general']})

### 気づき・疑問：
- 各STEPの動作結果をエクセルの様な一覧表にしないと差異が理解しにくい。他の方はどの様に整理されているのかが興味ある。
- 以下で、表形式で整理してみた。

| |STEP1：自分で作ったサーバーのツールを表示する|←|STEP2：エージェントに質問して、ツールの呼び出しを見る|STEP3：公開されているサーバーを取得して使う|STEP 4：外部で動くサーバーに接続する|
|:----|:----|:----|:----|:----|:----|
|command|uv run python -m app.agent --list-tools|uv run python -m app.mcp_agent --list-tools|uv run python -m app.mcp_agent --trace "AIE2026の講座の宿題にある「3レベル」とは何ですか？"|uv run python -m app.mcp_agent --trace "https://modelcontextprotocol.io/ の冒頭を日本語で3行に要約して"|uv run python -m app.mcp_agent --trace "AWSのドキュメントを検索して、Amazon S3 のバージョニングを2行で説明して"|
|output|モデルに見えているツール: 1 個<br>[search_materials]<br>  第1?3回の本講義資料を検索し、関連ページの本文・タイトル・ページ番号を返す。<br>  queryに調べたい内容を指定する。<br>  引数: {"query": {"type": "string"}}|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>モデルに見えているツール: 1 個<br>[glossary_search_glossary]<br>  講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。<br>  AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について<br>  答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。<br>  引数: {"query": {"title": "Query", "type": "string"}}|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>[model] gemini-3.5-flash-lite<br>[tools] MCP サーバー（app/mcp_servers.json）<br>以下省略。３レベルを回答<br>[tool call]   glossary_search_glossary({'query': '3レベル'})<br>[tool return] glossary_search_glossary -> [{"term": "宿題の3レベル", "lecture": 1, "definition": "AIE2026の宿題の構成。「基礎（講義内容の確認）」「実装（自分でコードを書く）」「発展（調査や試行）」の3段階があり、どれか1つに取り組…|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>[server] fetch: コマンドで起動（uvx mcp-server-fetch==2026.11.11 --no-readability-js）<br>[model] gemini-3.5-flash-lite<br>[tools] MCP サーバー（app/mcp_servers.json）<br>　以下省略<br>Model Context Protocol（MCP）の紹介ページの冒頭を3行で要約します。<br>　以下省略<br>[tool call]   fetch_fetch({'url': 'https://modelcontextprotocol.io/'})<br>[tool error]  →以下、省略<br>[tool call] →以下、省略<br>[tool return] →以下、省略<br>[tool call] →以下、省略<br>[tool return] →以下、省略<br>[tool call] →以下、省略<br>[tool return] →以下、省略<br>|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>[server] fetch: コマンドで起動（uvx mcp-server-fetch==2026.11.11 --no-readability-js）<br>[server] aws: URL に接続（https://knowledge-mcp.global.api.aws）　←起動するコマンドがない<br>[model] gemini-3.5-flash-lite<br>[tools] MCP サーバー（app/mcp_servers.json）<br>Amazon S3のバージョニングとは、同一バケット内で1つのオブジェクトに対して複数のバージョン（バリエーション）を保持し、上書きや削除が行われた際にも過去のバージョンを保存・復元できるようにする機能です。<br>誤操作やアプリケーションのエラーによるデータの消失を防ぐための強力な保護機能として機能します。|
|[usage]| | |[usage] 入力 715 tokens / 出力 149 tokens / モデル応答 2 回|[usage] 入力 9,915 tokens / 出力 238 tokens / モデル応答 5 回|[usage] 入力 6,359 tokens / 出力 103 tokens / モデル応答 2 回|

- もう少し、Quri で内容を確認してみた。

|STEP|接続するサーバー|接続方法|
|:----|:----|:----|
|STEP 2|自作の glossary サーバー|command でローカル起動|
|STEP 3|glossary ＋公開されている fetch サーバー|uvx で取得・起動|
|STEP 4|glossary ＋ fetch ＋ AWS サーバー|url で外部サーバーに接続|

- app/mcp_agent.pyの中身を変更するのではなく、app/mcp_servers.jsonにサーバーの設定を追加することで、接続するサーバーが増えることが分かった。



## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

### 取り組んだレベル：基礎 / 実装 / 発展


### やったこと：


### 結果：


### 気づき・疑問：