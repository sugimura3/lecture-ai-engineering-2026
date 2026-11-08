# 第3回 解答

## 演習

### 第1回（素のLLM）とエージェントの回答の違い：
* 「第1回（素のLLM」は、 AIE2026ではなく、一般的な **学習者のスキルや課題の難易度に応じた3段階の区分（初級・中級・上級、または基礎・応用・発展など）**を表示
* 一方、「app.agent」は、「data/glossary.json」からの定義を抽出している。

### `--trace` で見えたツール呼び出し（ツール名と引数）：
* 「--trace」オプションをいれない場合は、「LoRAとQLoRAの違いは、ベースモデルの重みの扱い方にあります。」だけだった。
* 一方、「--trace」オプションをいれた場合は、「LoRAとQLoRAの違いは、ベースモデルの重みをそのまま使うか、量子化してメモリを節約するかにあります。」と量子化にまで言及。[tool return]にて、「ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。」を抽出している。

### 1つの質問での LLM 呼び出し回数（`requests`）：
* 「[usage] ...requests=2...」から、LLMを２回呼び出している。

### 追加したツールと、docstring に書いた説明：
* 「app/tools.py」に「def make_quiz(lecture: int) -> str:」を追加。

* docstring は以下<br>
　"""指定した回の用語から1つ選び、理解度確認の問題と答えを作る。<br>
　その回で扱った用語の意味を確認するクイズを出すときに使う。<br>
　"""

* 「uv run python -m app.agent --list-tools」を実行すると、<br>
モデルに見えているツール: 3 個<br>
[search_glossary]<br>
[get_lecture_summary]<br>
[make_quiz]<br>
と表示された。

## 宿題

### 取り組んだレベル：基礎 / 実装 / 発展
* 基礎

### やったこと：
* app / agent.py のINSTRUCTIONSから「- 用語や講座に固有のことを答える前に、必ず search_glossary で用語集を確認する。」を削除。
* app / agent.py のINSTRUCTIONSに「- 用語や講座に固有のことを答える前に、必ず search_glossary で用語集の確認はしない。」に変更。
* app / tool.py の　def make_quiz(lecture: int) -> str:<br>
    """指定した回の用語から1つ選び、理解度確認の問題と答えを作る。<br>
を<br>
    """指定した回の用語をまとめて選び、理解度確認の問題と答えを作る。<br>
に変更する。

### 結果：
* 上記３件の確認を実施したが、変更前と差異はなかった。
* 「search_glossary」を削除するのは、<br>
agent.pyの「agent.tool_plain(tools.search_glossary)」を起動しないようにすると、<br>
「uv run python -m app.agent --list-tools」の動作結果は、「モデルに見えているツール: 2 個」となった。

### 気づき・疑問：
* 今回操作した「docstring（ドックストリング）」は、「関数の役割を説明する文章（docstring）」であるが、今回の変更した記入の仕方では効果が見られなかった。
* 別途、もう少し効果がみられるサンプルを用いて確認をしたい。