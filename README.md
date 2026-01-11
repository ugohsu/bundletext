# bundletext

`bundletext` は、指定したディレクトリやファイル群から  
**LLM / AI に渡すための「文脈テキスト」**を自動生成する CLI ツールです。

プロジェクト構造・ソースコード・設定ファイルなどを  
**安全で再現可能な形で 1 つのテキストに集約**します。

特定の AI サービスやモデルには依存しません。

---

## できること（要点）

- 📦 非バイナリファイルを自動収集して 1 つのテキストにまとめる
- 🌳 実際に bundle に含まれるファイルだけで **Project Tree を生成（デフォルト）**
- 🙈 `.gitignore` を尊重した自然な除外
- 🧹 多様な除外指定（名前 / glob / パス / 明示指定）
- 📏 ファイルサイズ制御（スキップ / 先頭のみ取り込み）
- 🔗 シンボリックリンク対応（循環自動回避）
- 🧪 dry-run による **事前確認（入る / 入らない理由つき）**
- 🔁 再現性のある安定した出力順
- 🔌 標準出力対応（パイプ利用）

---

## インストール

```bash
git clone https://github.com/ugohsu/bundletext.git
cd bundletext
chmod +x bundletext
sudo mv bundletext /usr/local/bin/
```

※ `~/bin` など PATH が通った場所でも可

---

## 基本的な使い方

### カレントディレクトリ以下をまとめる

```bash
bundletext . --out bundle.txt
```

### 複数パスを指定

```bash
bundletext src docs README.md --out bundle.txt
```

---

## 出力先（`--out`）

```bash
# ファイルに書き出す
bundletext . --out bundle.txt

# ディレクトリ指定（自動命名）
bundletext . --out ./bundles/

# 標準出力（パイプ利用）
bundletext . --out - | less

# プレースホルダ
bundletext . --out bundle_{date}_{time}.txt
```

---

## dry-run（重要）

`--dry-run` を指定すると、**実際には bundle を生成せず**、
以下の情報を表示します。

* Candidate Tree
  （除外適用後・binary/size 判定前の候補）
* Included Project Tree
  （実際に bundle に入るファイルのみ）
* Included Files（truncate 情報つき）
* Skipped Files（理由つき）

  * binary
  * too_big
  * unreadable
* Summary（理由別件数）

```bash
bundletext . --dry-run --out -
```

👉
「なぜ入らないのか？」を **事前に確認できる**のが目的です。

---

## Project Tree の仕様（デフォルト）

### tree-only-included（デフォルト）

Project Tree には、

> **実際に bundle に含まれるファイルだけ**

が表示されます。

* 「ツリーにはあるが本文にない」という不整合は起きません
* サイズ超過で skip / binary 判定で除外されたものは載りません

ツリーを無効化する場合：

```bash
bundletext . --no-tree --out bundle.txt
```

---

## `.gitignore` の扱い

デフォルトで `.gitignore` を再帰的に読み込み、
Git に近い感覚でファイルを除外します。

```bash
bundletext . --no-gitignore --out bundle.txt
```

> ※ 否定パターン（`!`）は未対応

---

## 除外指定（安全なデフォルト）

### デフォルト除外

事故防止のため、以下のようなものは **最初から除外**されます。

* `.git`, `node_modules`, `.venv`
* `__pycache__`
* `*.db`, `*.csv`, `*.zip` など

### 追加指定（デフォルト挙動）

以下は **デフォルト除外に追加**されます。

```bash
bundletext . --exclude-dir data logs --out bundle.txt
```

### デフォルト除外を無効化したい場合

```bash
bundletext . \
  --no-default-exclude-dir \
  --no-default-exclude-file \
  --no-default-exclude-glob \
  --exclude-dir data \
  --out bundle.txt
```

---

## glob / パス指定による除外

```bash
bundletext . \
  --exclude-glob "*.db" "*.csv" "**/data/**" \
  --exclude-path /abs/path/to/secret.txt \
  --out bundle.txt
```

---

## ファイルサイズ制御

```bash
# サイズ超過はスキップ
bundletext . --max-bytes 200000 --big-file skip --out bundle.txt

# サイズ超過は先頭だけ取り込む
bundletext . \
  --max-bytes 200000 \
  --big-file truncate \
  --truncate-bytes 120000 \
  --out bundle.txt
```

---

## バイナリ判定について

### デフォルト挙動

* NUL バイトが含まれる → バイナリ
* UTF-8 として decode できる → テキスト
* それ以外は制御文字率で判定

👉 日本語テキストの **誤判定が起きにくい設計**です。

### 完全に無効化する場合

```bash
bundletext . --no-binary-check --out bundle.txt
```

---

## シンボリックリンク

```bash
bundletext . --follow-symlinks --out bundle.txt
```

* ファイル / ディレクトリのリンクを辿ります
* realpath ベースで循環参照を自動回避
* 同一実体は 1 度だけ bundle

---

## 設計思想

* **安全なデフォルト**

  * 事故りにくさを最優先
* **可視性**

  * dry-run による事前確認
* **再現性**

  * 出力順・判定は常に安定
* **LLM 非依存**

  * 単なるテキスト生成ツール

「AI に渡す前の、最後の整形工程」を担うツールです。

---

## 制限事項

* `.gitignore` の否定パターン（`!`）は未対応
* 巨大リポジトリでは除外指定が必要な場合あり

---

## ライセンス

MIT License
