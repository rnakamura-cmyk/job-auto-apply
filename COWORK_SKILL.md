# 案件応募スキル（ルーター）

## トリガー例
```
案件応募を実行してください。クラウドワークスで、GAS・業務自動化カテゴリのみで。
案件応募を実行してください。スキルシフトで、AI活用カテゴリで。
案件応募を実行してください。クラウドワークスとスキルシフト両方で。
```

---

## Step 0. システムフォルダのパスを取得する

**このStep 0は必ず最初に実行すること。**

`COWORK_SKILL.md` と同じフォルダにある `config.json` を読み込み、`base_dir` の値を取得する。
以降のすべてのパスは **`{base_dir}`** を基準に組み立てる。

| OS | base_dir の例 |
|----|--------------|
| Windows | `C:\Users\yamada\Desktop\案件自動応募` |
| Mac | `/Users/yamada/案件自動応募` |

`base_dir` が空欄の場合は、`COWORK_SKILL.md` が置かれているフォルダのパスを `base_dir` として扱う。

---

## Step 1. 対象サイトを特定する

指示文からサイトを判断して、該当フォルダのファイルを読み込む：

| 指示に含まれるキーワード | 参照フォルダ |
|------------------------|-------------|
| クラウドワークス / crowdworks / CW | `{base_dir}/sites/crowdworks/` |
| スキルシフト / skillshift / skill-shift | `{base_dir}/sites/skillshift/` |
| ハイプロ / hipro / HiPro Direct | `{base_dir}/sites/hipro/` |
| ふるさと兼業 / furusatokengyo | `{base_dir}/sites/furusatokengyo/` |
| オタノミ / otanomi | `{base_dir}/sites/otanomi/` |
| チイキズカン / chiikizukan | `{base_dir}/sites/chiikizukan/` |
| ライフル / lifull / LIFULL LOCAL MATCH | `{base_dir}/sites/lifull/` |
| ロッツフル / lotsful | `{base_dir}/sites/lotsful/` |

読み込むファイル：
- `{base_dir}/sites/{サイト名}/config.json`
- `{base_dir}/sites/{サイト名}/SKILL.md`
- `{base_dir}/my_profile.md`（共通）

カテゴリ指定がある場合は config.json の categories より優先する。

---

## Step 2. 応募済みURLリストを読む（重複応募防止）

`{base_dir}/applied_urls.txt` を読み込む。
このファイルに含まれるURLは **サイト問わず必ずスキップ** する。
ファイルが存在しない場合は空リストとして扱い続行する。

---

## Step 3. 案件を検索して候補を収集する（応募はまだしない）

読み込んだ `{base_dir}/sites/{サイト名}/SKILL.md` の手順に従ってサイトにアクセスし、案件を検索する。
**このステップでは応募せず、候補の収集のみ行う。**

収集した案件を以下の基準でふるいにかける：

**候補に残す条件（すべて満たす）：**
- 締切が3日以上ある
- ng_keywords に含まれるキーワードが案件内容にない
- my_profile.md の得意領域と案件内容が1つ以上合っている
- applied_urls.txt に含まれていない

**優先順位付け：**
- config.json の `search_filters.preferred_keywords` に含まれるキーワードが案件タイトル・本文に合致する場合、応募優先度を上げる
- 合致数が多い案件を先に処理する

**候補から除外する条件（どれか1つでも当てはまる）：**
- デザイン・翻訳・語学・常駐・対面必須・出社・イラスト・グラフィック
- 締切切れ・すでに終了
- プロフィールのスキルと全く合わない内容

---

## Step 4. 条件に合う案件に応募する

収集した候補案件すべてに順番に応募する。

各案件について：
1. 案件詳細ページを開く
2. 応募文を自動生成（下記ルールに従う）
3. フォームを埋めて応募送信
4. フォームに添付欄がある場合は `{base_dir}/documents/` 内のファイル（履歴書・ポートフォリオ等）を添付する

応募文の生成ルール：
- 冒頭は `my_profile.md` の「名前」を使い「はじめまして、{名前}と申します。」で始める
- 案件内容に直接触れる（「○○の自動化」「○○の立ち上げ支援」など）
- my_profile.md の実績を1つ具体的に引用する
- 300〜400文字程度
- 最後は「ぜひ詳細をお聞かせください」で締める
- 決して推測や嘘の実績を書かない

---

## Step 5. 結果をjsonlで出力する

出力先: `{base_dir}/inbox/{サイト名}_YYYYMMDD_HHMMSS.jsonl`

**応募した案件（1件1行）:**
```
{"action":"append_job","site":"サイト名","job_title":"案件名","category":"カテゴリ","reward_min":"下限","reward_max":"上限","client":"クライアント名","applications":応募数,"contracts":"契約状況","deadline":"締切日(YYYY-MM-DD)","url":"案件URL","status":"応募済","applied_at":"今日の日付(YYYY-MM-DD)","result":"","memo":"どのプロフィール項目を使ったか一言"}
```

**スキップした案件（記録のみ）:**
```
{"action":"append_job","site":"サイト名","job_title":"案件名","category":"カテゴリ","reward_min":"","reward_max":"","client":"クライアント名","applications":応募数,"contracts":"契約状況","deadline":"締切日","url":"案件URL","status":"スキップ","applied_at":"","result":"","memo":"スキップ理由"}
```

**ルール：**
- 1レコードは必ず1行で完結させる
- 推測値を入れず、画面から取得した事実のみ書く
- 応募した・しない両方を記録する
