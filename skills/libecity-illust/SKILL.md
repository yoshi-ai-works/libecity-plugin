---
description: リベシティ記事の本文中挿絵をgpt-image-2で生成・保存する。サムネイルは /libecity-image を使う。
---

# リベシティ記事 挿絵生成スキル（gpt-image-2）

## スキルの目的

リベシティ完成記事の本文中に挿入する挿絵をgpt-image-2で生成し、記事フォルダに保存する。
サムネイル（記事の顔）は `/libecity-image`（Gemini）、本文中の挿絵は本スキル（gpt-image-2）で使い分ける。

> **前提：** `~/Desktop/リベシティ/ノウハウ図書館/画像生成/` に以下のファイルが必要です。
> - `dalle_image_gen.py`（OpenAI API呼び出しスクリプト）
> - `check_models.py`（新モデル確認スクリプト）
> - `models_config.json`（使用モデル設定）
> - `mascot.png`（マスコット参照画像）

---

## 【固定仕様】マスコットキャラクター

マスコットキャラを使う場合、以下の仕様を**毎回・必ずプロンプトに含める**。省略・変更禁止。

```
Mascot character spec (STRICT - do not change):
- Species: penguin
- Body color: purple-blue (#7B7FC4 approx) covering the ENTIRE body including face and head
- Face: fully purple-blue with NO white area on face
- Belly: white, oval-shaped on front torso only
- Beak: yellow, small
- Cheeks: two pink circle blush marks on the purple-blue face
- Feet: reddish-brown, small
- Body shape: round and compact. NOT overly fat. NOT 3D rendered.
- Eyes: simple black dots
- Style: flat 2D kawaii illustration, Japanese cute style, clean black outlines, soft colors
- Absolutely do NOT make it chubby/bloated/realistic/3D. Keep the slim round silhouette.
```

参照画像パス：`~/Desktop/リベシティ/ノウハウ図書館/画像生成/mascot.png`

---

## 【固定仕様】挿絵スタイル統一指定

記事内の挿絵は**全枚同じスタイル**にする。以下を**全プロンプトに必ず付加する**：

```
Style: flat 2D illustration, soft pastel colors, simple and clean,
suitable for blog article, no text in image,
consistent warm tone across all illustrations in the same article.
```

> **重要**：1記事内の挿絵はスタイルを統一すること。サムネイル（クレイ風）とは意図的に分けている。

---

## 【固定仕様】挿絵出力設定

```
Output spec:
- Aspect ratio: 16:9 または 4:3（本文中に自然に収まるサイズ）
- Resolution: 1200×675px 以上（16:9の場合）
- Format: PNG
- No text in image
```

---

## 手順

### STEP 0 ｜ 新モデルチェック

スキル起動時に毎回以下を実行する：

```bash
python "~/Desktop/リベシティ/ノウハウ図書館/画像生成/check_models.py"
```

- **新モデルが見つかった場合**：ユーザーに通知する。
- **新モデルなし**：そのまま STEP 1 へ進む（ユーザーへの報告不要）

### STEP 1 ｜ 引数を受け取る

`$ARGUMENTS` を確認する。

- **記事ファイル名が渡された場合**（例：`20260414_リベシティ投稿_本文.md`）→ STEP 2へ
- **プロンプトが渡された場合**（例：`/libecity-illust 青い背景に…`）→ STEP 3へ
- **引数なしの場合** → 完成記事フォルダのファイル一覧を表示してユーザーに選択を促す

完成記事フォルダ：`~/Desktop/リベシティ/ノウハウ図書館/完成記事/`

フォルダが存在しない場合は自動的に作成する。

### STEP 2 ｜ 記事のセクションを読んで挿絵プランを提案する

記事ファイルを読み込み、本文の各セクション（H2見出し単位）を確認する。

以下の基準で「挿絵を入れると効果的なセクション」を2〜4箇所ピックアップする：
- 概念・仕組みの説明が続く箇所（視覚化で理解が深まる）
- Before/After や変化の瞬間を描ける箇所（感情的な共感を強化）
- 手順・ステップを示している箇所（流れを一目で伝えられる）

ピックアップしたセクションごとに以下を提示する：
- 挿絵のコンセプト（何を描くか・なぜここか）
- 英語プロンプト案（スタイル統一指定・キャラ仕様を含む完成形）

**提示フォーマット：**
```
【挿絵プラン】
① [セクション名]
   コンセプト：〇〇が△△している場面。理由：手順の流れを一目で伝えるため。
   プロンプト：[完成形プロンプト]

② ...

合計 X 枚 → 約 $X.XX（1枚 約$0.04 / medium品質・1536×1024）
生成する番号を選んでください（例：1 3 / すべて / なし）
```

### STEP 3 ｜ 画像を生成する

**① 既存連番を確認する**

完成記事フォルダ内の該当フォルダでファイル一覧を確認し、最大連番を特定する（ファイルがなければ `_01` から開始）。

**② 生成コマンドを実行する**

```bash
python "~/Desktop/リベシティ/ノウハウ図書館/画像生成/dalle_image_gen.py" "[プロンプト]" "[出力パス]"
```

モデルを指定する場合（デフォルト：models_config.json の default_model）：

```bash
python "~/Desktop/リベシティ/ノウハウ図書館/画像生成/dalle_image_gen.py" "[プロンプト]" "[出力パス]" --model gpt-image-2
```

**出力パスの決め方：**
- 記事ファイル名から `_本文.md` を除いた名前をフォルダ名にする
- フォルダ：`~/Desktop/リベシティ/ノウハウ図書館/完成記事/[フォルダ名]/`
- ファイル名：`[フォルダ名]_illust_01.png`（連番・上書きしない）
- フォルダが存在しない場合は自動的に作成する

複数枚生成する場合は1枚ずつ実行し、生成のたびにユーザーに確認する。

### STEP 4 ｜ 生成結果を評価してユーザーに確認する

生成完了後、以下を表示する：

1. 保存先ファイルパス
2. 使用したプロンプト（記録用）
3. OKかどうかをユーザーに確認する

ユーザーが「NG・やり直し」の場合：
- **何が気に入らないか**を1点だけ聞く
- プロンプトを修正して再生成（連番を上げて保存・上書きしない）

ユーザーが「OK」の場合：
- 次の挿絵があれば続けて生成する
- 全枚OK後、使用プロンプト一覧を `[フォルダ名]_illust_prompts.txt` として同フォルダに保存する

### STEP 5 ｜ 完了報告

```
✅ 挿絵生成完了（X枚）
📁 保存先：[フォルダパス]
📝 プロンプト記録：[プロンプトファイルパス]
🖼 エクスプローラーで確認：explorer "[フォルダパス]"
💰 今回の費用：約 $X.XX
```

---

## エラー対応

| エラー種別 | 対応 |
|-----------|------|
| APIキーエラー（401） | `OPENAI_API_KEY` 環境変数の設定を促す |
| レート制限（429） | 「30秒ほど待ってから `/libecity-illust` を再実行してください」と伝える |
| ファイルが見つからない | 記事ファイルのパスをBashで確認してユーザーに報告する |
| その他エラー | エラーメッセージをそのまま表示し、原因の切り分けをユーザーと行う |

---

## 注意事項

- gpt-image-2 は1枚あたり約$0.04（medium品質・1536×1024）。high品質は約$0.17
- 再生成は連番を上げて保存（上書きしない）
- 1記事内の挿絵は必ずスタイルを統一する（同じスタイル指定を全枚に付加）
- OKが出たプロンプトは `_illust_prompts.txt` に保存する（品質の再現性確保のため）
- サムネイル生成は `/libecity-image`（Gemini）を使う
