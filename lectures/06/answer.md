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
- 基礎

### やったこと：

- 追加前は、「app/mcp_server.py」の「server.tool()(tools.search_glossary)」が実行される。<br>
　　→「server.tool()(tools.search_glossary)」とは、<br>
　　　「app/tools.py」の「def search_glossary(query: str) -> str:」を呼び出している<br>
<br>
    """講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。<br>

    AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について<br>
    答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。<br>
    """<br>

- app/mcp_server.py に、「server.tool()(tools.get_lecture_summary)」 を追加する。<br>
　　→「app/tools.py」の「def get_lecture_summary(lecture: int) -> str:」を呼び出している。<br>
<br>
    """指定した回（1始まりの整数）の講義の要点を返す。<br>
    「第3回では何を学ぶ？」のように、回の内容や関係を聞かれたときに使う。<br>
    """<br>

- この両者を比較した。


### 結果：

| |app/mcp_server.pyにおいて、1行を書き足す前|app/mcp_server.pyのTODOの行の下に、1行を書き足したあと|*|app/mcp_server.pyにおいて、1行を書き足す前|app/mcp_server.pyのTODOの行の下に、1行を書き足したあと|
|:----|:----|:----|:----|:----|:----|
|command|uv run python -m app.mcp_agent --list-tools|uv run python -m app.mcp_agent --list-tools| |uv run python -m app.mcp_agent --trace "第2回では何を学びま すか？"|uv run python -m app.mcp_agent --trace "第2回では何を学びま すか？"|
|output|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>モデルに見えているツール: 1 個<br>[glossary_search_glossary]<br>  講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。<br>  AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について<br>  答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。<br>  引数: {"query": {"title": "Query", "type": "string"}}|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>モデルに見えているツール: 2 個　←前回は1回だった<br>[glossary_search_glossary]<br>  講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。<br>  AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について<br>  答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。<br>  引数: {"query": {"title": "Query", "type": "string"}}<br>[glossary_get_lecture_summary]<br>  指定した回（1始まりの整数）の講義の要点を返す。<br>  「第3回では何を学ぶ？」のように、回の内容や関係を聞かれたときに使う。<br>  引数: {"lecture": {"title": "Lecture", "type": "integer"}}| |[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>[model] gemini-3.5-flash-lite<br>[tools] MCP サーバー（app/mcp_servers.json）<br>第2回では、主に **「LoRA」** や **「QLoRA」** といった効率的なモデルのファインチューニング手法（PEFT手法）について学びます。<br>具体的には以下の内容を扱います。<br>- 事前学習済みの重みを固定し、小さな低ランク行列（アダプタ）のみを学習する **LoRA** の仕組みと演習<br>- モデルを量子化してメモリを節約しながら学習・推論を行う **QLoRA** についての発展的な比較<br>関係する回：第2回<br>|[server] glossary: コマンドで起動（uv run python -m app.mcp_server）<br>[model] gemini-3.5-flash-lite<br>[tools] MCP サーバー（app/mcp_servers.json）<br>第2回では、**「ファインチューニングとLoRA」**について学びます。<br>具体的な学習内容は以下の通りです：<br>1. **事前学習とファインチューニングの違い**<br>   - 事前学習で汎用的な言語能力を身につけたモデルに対し、目的のデータで追加学習させる仕組みや、向く場面・向かない場面を学びます。<br>2. **SFT（教師あり微調整）**<br>   - 入力と望ましい出力のペアを用いて学習させる方法や、データの質と形式の重要性を学びます。<br>3. **PEFTとLoRA**<br>   - 全パラメータを更新せず、小さな低ランク行列だけを学習させることでメモリと容量を大幅に削減する「LoRA」の仕組みを理解します。また、QLoRA（4bit量子化とLoRAの組み合わせ）との関係についても扱います。<br>4. **過学習への注意**<br>   - 検証データを用いて性能を確かめることの重要性を学びます。<br>演習では、Google Colab上で小さなモデルを架空FAQのデータでLoRA学習させ、出力形式の精度がどう変わるかを実際に体験します。<br>第2回|
|trace| | | |[tool call]   glossary_search_glossary({'query': '第2回'})<br>[tool return] glossary_search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…|[tool call]   glossary_get_lecture_summary({'lecture': 2})<br>[tool return] glossary_get_lecture_summary -> # 第2回 ファインチューニングとLoRA<br>## この回のねらい<br>- 事前学習とファインチューニングの違い、ファインチューニングが向く場面と向かない場面を理解する<br>- LoRAの考え方を理解し、Google Colab上で小さなモデルを…|
|usage| | | |[usage] 入力 819 tokens / 出力 140 tokens / モデル応答 2 回|[usage] 入力 1,274 tokens / 出力 275 tokens / モデル応答 2 回|
|テストする| | | |uv run pytest -q<br>........................                                                                                            [100%]<br>24 passed in 5.31s|uv run pytest -q<br>........................                                                                                            [100%]<br>24 passed in 4.83s|


### 気づき・疑問：

- 作業的には簡単だったが、どの様に動作しているのかを理解知るのには、非常に苦労しました。
- 本講義の作業段階で、「app/tools.py」の「def make_quiz(lecture: int) -> str:」が保存されていなかった様で、「uv run pytest -q」で、エラーが発生。また、Quri により、エラーが回避出来、課題も達成できました。
