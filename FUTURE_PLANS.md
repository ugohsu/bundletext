# Future Plans: LLM Context Optimization

## 1. 背景と課題 (Background & Issues)

現在の `bundletext` は人間にとっての可読性が高い「インデント形式」のプロジェクトツリーを採用している。しかし、Gemini 1.5 Pro などの Long Context モデルや、NotebookLM などの RAG (Retrieval-Augmented Generation) ツールに入力する際、以下の課題が明らかになった。

* **インデントの消失**: トークン化や前処理の段階で空白（インデント）がトリミングされ、ネストの深さ情報が失われるリスクがある。
* **文脈の分断 (Chunking)**: RAG において、ツリー定義とファイル内容が別のチャンクに分断された際、ファイルパスの文脈（親ディレクトリの情報）が欠落する。
* **トークンの浪費**: 視覚的なツリーとファイルヘッダーで情報を二重に提示することは、限られたコンテキストウィンドウの消費につながる。

## 2. 機能拡張の方針 (Roadmap)

AI モデルがコードベースを「正確に、かつ効率的に」理解できるようにするため、出力フォーマットのデフォルト仕様を以下の通り変更する。

### 2.1 デフォルト形式の変更：フラットリスト (Flat List as Default)

デフォルトの出力モードを「インデント付きツリー」から「ルートからの完全な相対パスのリスト」に変更する。

* **変更前 (Visual Tree):**
    ```text
    colab-nlp/
        colab_nlp/
            __init__.py
    ```

* **変更後 (Default - Flat List):**
    ```text
    colab-nlp/colab_nlp/__init__.py
    ```

**仕様のポイント:**
* **排他的表示**: フラットリスト（フルパス）モードが選択されている場合、視覚的なインデントツリーは**出力しない**。これによりトークン消費を抑え、AI の混乱を防ぐ。
* **オプション対応**: 人間が読むための「インデント付きツリー」は、明示的なオプション（例: `--visual-tree`）が指定された場合のみ出力する。

### 2.2 アンカーの完全一致 (Anchor Synchronization)

「Project Tree（フラットリスト）に記載されたパス」と「ファイル内容のヘッダー」を文字列として完全に一致させ、強力なアンカー（参照点）を作成する。

* **出力イメージ:**
    ```text
    === Project File List ===
    src/utils/logger.py
    src/main.py
    ...
    
    === File Contents ===
    <<<FILE: src/utils/logger.py>>>
    ...
    <<<END FILE>>>
    ```

この「完全一致」により、LLM は探索コストをかけずに、リスト上の定義とファイルの中身を一対一で確実に対応付けることができる。

## 3. 期待される効果 (Expected Outcomes)

* **RAG 耐性の最大化**: どの行（チャンク）が検索されても常にフルパスが含まれているため、NotebookLM 等でのソース特定精度が飛躍的に向上する。
* **Deep Nesting への対応**: ディレクトリ階層がどれだけ深くても、インデント解析に依存しないため、構造の誤認が発生しない。
* **コンテキスト効率の向上**: 重複する視覚情報を排除することで、より多くのコードやドキュメントを LLM に入力可能になる。

---
*Document updated based on findings regarding RAG behavior and token efficiency.*
