# bundletext

`bundletext` は、指定したディレクトリやファイル配下にある  
**（原則として）すべての非バイナリファイル**を 1 つのテキストにまとめる  
小さな CLI ツールです。

主な用途は **LLM / AI に与えるための文脈テキストの生成**ですが、  
特定のモデルやサービスには依存しない設計になっています。

---

## 何ができるのか

LLM を使った作業では、次のような情報を **まとめて渡したい**場面がよくあります。

- プロジェクト全体の構成
- README や設計ドキュメント
- 実際のソースコードや設定ファイル

これを手作業でコピーすると、

- 抜け漏れが起きやすい
- `.gitignore` を考慮し忘れる
- 再現性がない

といった問題が出がちです。

`bundletext` は、これらを **安全で再現可能な形で自動化**します。

---

## 特長

- 📦 指定したパス配下の **非バイナリファイルを自動収集**
- 🙈 `.gitignore` を **デフォルトで尊重**
- 🧠 バイナリっぽいファイルを自動判定して除外
- 📏 ファイルサイズ上限の指定（スキップ / 先頭のみ取り込み）
- 🧹 柔軟な除外指定
  - ディレクトリ名
  - ファイル名
  - パスの部分一致
  - glob パターン（`*.db`, `**/data/**` など）
  - 明示的なパス指定
- 🌳 プロジェクトツリーの自動生成（ON/OFF可）
- 🔁 ソート済み・再現可能な出力
- 🔌 標準出力対応（パイプで利用可能）

---

## インストール

リポジトリをクローンし、`bundletext` を PATH の通った場所に置いてください。

```bash
git clone https://github.com/yourname/bundletext.git
cd bundletext
chmod +x bundletext
sudo mv bundletext /usr/local/bin/
````

（`~/bin` などでも問題ありません）

---

## 基本的な使い方

### カレントディレクトリ以下をまとめる

```bash
bundletext . --out bundle.txt
```

### 複数のパスを指定する

```bash
bundletext src docs README.md --out bundle.txt
```

---

## 出力先の指定（`--out`）

`--out` は柔軟に指定できます。

### ファイルに書き出す

```bash
bundletext . --out bundle.txt
```

### ディレクトリを指定する（自動命名）

```bash
bundletext . --out ./bundles/
```

### 標準出力に出す（パイプ利用）

```bash
bundletext . --out - | less
```

### 日付・時刻プレースホルダを使う

```bash
bundletext . --out bundle_{date}_{time}.txt
```

---

## `.gitignore` の扱い

デフォルトでは、`.gitignore` を再帰的に読み込み、
Git と同様の感覚でファイルを除外します。

無効化したい場合は次のように指定してください。

```bash
bundletext . --no-gitignore --out bundle.txt
```

> ※ 現在は `!` による否定パターンはサポートしていません
> （実用重視の簡易実装です）

---

## ファイルの除外方法

### glob パターンで除外

```bash
bundletext . \
  --exclude-glob "*.db" "*.csv" "**/data/**" \
  --out bundle.txt
```

### 特定のファイル・パスを除外

```bash
bundletext . \
  --exclude-path data/secret.txt tmp/debug.log \
  --out bundle.txt
```

### パスの部分一致で除外

```bash
bundletext . \
  --exclude-path-substr "/outputs/" "/cache/" \
  --out bundle.txt
```

---

## ファイルサイズ制御

### サイズ超過ファイルをスキップ

```bash
bundletext . \
  --max-bytes 200000 \
  --big-file skip \
  --out bundle.txt
```

### サイズ超過ファイルを先頭だけ取り込む

```bash
bundletext . \
  --max-bytes 200000 \
  --big-file truncate \
  --truncate-bytes 120000 \
  --out bundle.txt
```

---

## プロジェクトツリー

デフォルトで、簡易的なプロジェクトツリーが出力されます。

無効化する場合：

```bash
bundletext . --no-tree --out bundle.txt
```

ツリーの深さを制限する場合：

```bash
bundletext . --tree-max-depth 3 --out bundle.txt
```

---

## 設計方針（Philosophy）

* **テキストファースト**
  出力はすべてプレーンテキスト。差分確認や再利用が容易です。
* **安全なデフォルト**
  バイナリ・巨大ファイルを誤って含めない設計。
* **LLM 非依存**
  特定の AI サービスやモデルに縛られません。
* **組み合わせやすさ**
  パイプやシェルスクリプトで自然に使えます。

「1 つのことを、うまくやる」ためのツールです。

---

## 制限事項

* `.gitignore` の否定パターン（`!`）は未対応
* バイナリ判定はヒューリスティック
* 巨大リポジトリでは追加の除外指定が必要な場合があります

---

