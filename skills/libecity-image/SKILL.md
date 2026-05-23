---
description: リベシティ記事のサムネイル画像をGemini APIで生成・保存する。記事ファイルまたは直接プロンプトを受け取る。
---

# リベシティ記事 サムネイル画像生成スキル

## スキルの目的

リベシティ完成記事のサムネイル画像をGemini APIで生成し、記事フォルダに保存する。

> **前提：** `~/Desktop/リベシティ/ノウハウ図書館/画像生成/` に以下のファイルが必要です。
> - `gemini_image_gen.py`（Gemini API呼び出しスクリプト）
> - `check_gemini_models.py`（新モデル確認スクリプト）
> - `gemini_models_config.json`（使用モデル設定）
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

## 【固定仕様】サムネイル出力設定

```
Output spec:
- Aspect ratio: 16:9（横長・SNSサムネイル標準）
- Resolution: 1280×720px 以上
- Format: PNG
- No text in image（タイトルは後で別途追加するため不要）
```

---

## 【固定仕様】構図パターン集

記事テーマに合わせて以下から構図を選び、プロンプトに明示する。

| パターン | 使いどころ | プロンプトキーワード |
|---------|-----------|-----------------|
| A：達成・喜び | 初投稿・完成・成功体験 | `mascot celebrating with both flippers raised, sparkles around` |
| B：作業・集中 | ツール紹介・手順解説 | `mascot sitting in front of a laptop, focused expression` |
| C：悩み→気づき | Before/After・壁を越えた話 | `mascot with lightbulb above head, surprised expression` |
| D：比較・提示 | 選択肢・比較記事 | `mascot pointing to two options on signboards` |
| E：旅・成長 | ロードマップ・変化の記録 | `mascot walking along a glowing path toward a bright horizon` |

---

## 【固定仕様】スタイル共通指定

以下を**全プロンプトに必ず付加する**（コピペ用）：

```
Style: clay animation style, sky blue background (#87CEEB),
soft studio lighting, warm fairy-tale atmosphere,
clean single-page layout, no multiple panels, no text in image.
Rendered in high quality 16:9 format.
```

---

## 手順

### STEP 0 ｜ 新モデルチェック

スキル起動時に毎回以下を実行する：

```bash
python "~/Desktop/リベシティ/ノウハウ図書館/画像生成/check_gemini_models.py"
```

- **新モデルが見つかった場合**：ユーザーに通知する。
- **新モデルなし**：そのまま STEP 1 へ進む（ユーザーへの報告不要）

### STEP 1 ｜ 引数を受け取る

`$ARGUMENTS` を確認する。

- **記事ファイル名が渡された場合**（例：`20260414_リベシティ投稿_本文.md`）→ STEP 2へ
- **プロンプトが渡された場合**（例：`/libecity-image 青い背景に…`）→ STEP 3へ
- **引数なしの場合** → 完成記事フォルダのファイル一覧を表示してユーザーに選択を促す

完成記事フォルダ：`~/Desktop/リベシティ/ノウハウ図書館/完成記事/`

フォルダが存在しない場合は自動的に作成する。

### STEP 2 ｜ 記事内容からサムネイルプロンプトを自動生成する

記事ファイルを読み込み、以下を抽出する：
- 記事のテーマ・タイトル
- 主人公の感情（Before/After）
- 記事の核心メッセージ

抽出した内容をもとに、以下の手順でプロンプトを組み立てる：

**① 構図パターンを選ぶ**
「構図パターン集」から記事テーマに最も合うパターンを1つ選び、理由を一言添えてユーザーに提示する。

**② プロンプトを組み立てる（テンプレート）**

```
[構図パターンのキーワード],
[記事テーマを象徴する背景・小道具 1〜2個],
[マスコットキャラクター仕様（全文）],
[スタイル共通指定（全文）]
```

**③ ユーザーに提示・承認を得る**
プロンプトを表示し、承認を得てから STEP 3 へ進む。
修正は**最大2回**まで受け付ける。3回目は「実際に生成して見た目で確認しましょう」と促す。

### STEP 3 ｜ 画像を生成する

以下のコマンドを Bash ツールで実行する：

```bash
python "~/Desktop/リベシティ/ノウハウ図書館/画像生成/gemini_image_gen.py" "[プロンプト]" "[出力パス]"
```

モデルを指定する場合（デフォルト：gemini_models_config.json の default_model）：

```bash
python "~/Desktop/リベシティ/ノウハウ図書館/画像生成/gemini_image_gen.py" "[プロンプト]" "[出力パス]" --model imagen-3.0-generate-002
```

**出力パスの決め方：**
- 記事ファイル名から `_本文.md` を除いた名前をフォルダ名にする
- フォルダ：`~/Desktop/リベシティ/ノウハウ図書館/完成記事/[フォルダ名]/`
- ファイル名：`[フォルダ名]_thumbnail_01.png`（連番・上書きしない）
- 既存ファイルを確認して次の連番を決める（例：`_01` があれば `_02`）
- フォルダが存在しない場合は自動的に作成する

プロンプト直接入力の場合は、今日の日付をフォルダ名・ファイル名のベースにする。

### STEP 4 ｜ 生成結果を評価してユーザーに確認する

生成完了後、以下を表示する：

1. 保存先ファイルパス
2. 使用したプロンプト（記録用）
3. OKかどうかをユーザーに確認する

ユーザーが「NG・やり直し」を選んだ場合：
- **何が気に入らないか**を1点だけ聞く（色？構図？キャラの雰囲気？）
- 回答をもとにプロンプトを修正して再生成する
- 再生成は**連番を上げて保存**（上書きしない）

ユーザーが「OK」を選んだ場合：
- 使用したプロンプトを `[フォルダ名]_thumbnail_prompt.txt` として同フォルダに保存する（次回の再利用・参考用）

### STEP 5 ｜ 完了報告

```
✅ サムネイル生成完了
📁 保存先：[ファイルパス]
📝 プロンプト記録：[プロンプトファイルパス]
🖼 エクスプローラーで確認：explorer "[フォルダパス]"
🔁 再生成する場合：/libecity-image [同じファイル名] で上書き再生成できます
```

---

## エラー対応

| エラー種別 | 対応 |
|-----------|------|
| APIキーエラー（401） | `GEMINI_API_KEY` 環境変数の設定を促す |
| レート制限（429） | 「30秒ほど待ってから `/libecity-image` を再実行してください」と伝える |
| ファイルが見つからない | 記事ファイルのパスをBashで確認してユーザーに報告する |
| その他エラー | エラーメッセージをそのまま表示し、原因の切り分けをユーザーと行う |

---

## 注意事項

- 再生成は連番を上げて保存（上書きしない）
- OKが出たプロンプトは必ず `_thumbnail_prompt.txt` に保存する（品質の再現性確保のため）
- 本文中の挿絵は `/libecity-illust`（gpt-image-2）を使う
