# 案件応募自動化システム

副業サイトで案件を自動検索・応募し、結果をGoogleスプレッドシートに記録するシステムです。
Coworkで毎日自動実行し、気付いたら案件が溜まっている設計。

## Claude Codeへの指示

**このファイルを読んだら、まず以下のメッセージをユーザーに送ってください。他のステップは何もせず、返答を待ってください。**

---

```
案件応募自動化システムが見つかりました。

このシステムをセットアップしますか？
（スプシ連携・プロフィール入力・Cowork設定まで一緒にやります）

y で進める / n でキャンセル
```

---

**「n」が返ってきたら「了解です。必要なときはいつでも声をかけてください。」と伝えて終了してください。**

---

**「y」が返ってきたら、以下のフェーズを順番に実行してください。**

---

## フェーズ1：技術セットアップ

### 1-0. Python・git の確認

```bash
python --version
```
失敗したら `python3 --version` を試す。

どちらも失敗 → Pythonのインストールを案内。
成功 → 以降はそのコマンドを使う。

```bash
git --version
```
失敗 → gitのインストールを案内。

---

### 1-1. システムパスの自動検出

```bash
python -c "
import json, pathlib, sys
base = str(pathlib.Path('.').resolve())
p = pathlib.Path('config.json')
cfg = json.loads(p.read_text(encoding='utf-8'))
cfg['base_dir'] = base
p.write_text(json.dumps(cfg, ensure_ascii=False, indent=2), encoding='utf-8')
print('OS:', sys.platform)
print('base_dir:', base)
"
```

表示されたパスをユーザーに確認。

---

### 1-2. Pythonパッケージのインストール

```bash
pip install requests
```

---

### 1-3. Googleスプレッドシート + GAS の設定

ユーザーにこう伝える：

```
Googleスプレッドシートを作成してスプシ連携を設定します。

1. https://sheets.new で新しいスプレッドシートを作成
2. シート名（下のタブ）を「案件管理」に変更
3. 「拡張機能」→「Apps Script」を開く
4. 既存のコードを全部消して、以下のコードを貼り付け：
```

以下のコードをユーザーに提示する：

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("案件管理");
  var data = JSON.parse(e.postData.contents);

  if (sheet.getLastRow() === 0) {
    sheet.appendRow([
      "No.", "サイト名", "案件名", "カテゴリ", "報酬（下限）", "報酬（上限）",
      "クライアント", "応募数", "契約状況", "締切日", "URL",
      "応募日", "ステータス", "結果", "メモ"
    ]);
  }

  var rows = Array.isArray(data) ? data : [data];

  rows.forEach(function(row) {
    var no = sheet.getLastRow();
    sheet.appendRow([
      no,
      row.site || "",
      row.job_title || "",
      row.category || "",
      row.reward_min || "",
      row.reward_max || "",
      row.client || "",
      row.applications || "",
      row.contracts || "",
      row.deadline || "",
      row.url || "",
      row.applied_at || "",
      row.status || "",
      row.result || "",
      row.memo || ""
    ]);
  });

  return ContentService.createTextOutput(
    JSON.stringify({ status: "ok", count: rows.length })
  ).setMimeType(ContentService.MimeType.JSON);
}
```

続けて案内：

```
5. 「デプロイ」→「新しいデプロイ」
6. 種類：ウェブアプリ
7. アクセスできるユーザー：「全員」
8. 「デプロイ」をクリック → アクセスを承認
9. 表示されるURLをここに貼り付けてください
```

URLを受け取ったら `config.json` の `gas_url` を更新する。

---

### 1-4. 動作確認

```bash
python sync_sheets.py
```

「inbox/ に処理対象ファイルなし」と表示されれば成功。

---

## フェーズ2：プロフィール設定

以下の質問を1つずつ聞く。まとめて聞かず、返答を受けてから次へ。

**Q1** あなたのお名前・ビジネスネームを教えてください。
**Q2** 一言で表すと、どんな仕事が得意ですか？
**Q3** 得意な領域・スキルを教えてください。箇条書きでOK。
**Q4** 実績を1〜3つ教えてください。数字があると◎。
**Q5** 応募したくない仕事の条件はありますか？
**Q6** 希望の単価・稼働時間はありますか？（任意）

全回答を `my_profile.md` に書き出す：

```markdown
# プロフィール

## 基本情報
- 名前: {Q1}
- キャッチコピー: {Q2}

## 得意領域
{Q3を箇条書き}

## 実績ハイライト
{Q4を箇条書き}

## 応募NG条件
{Q5を箇条書き}

## 希望条件
{Q6}
```

---

## フェーズ3：サイトログイン確認

ユーザーにこう伝える：

```
以下の3サイトにブラウザでログインしてください：

1. チイキズカン: https://chiiki-zukan.com/
2. スキルシフト: https://www.skill-shift.com/
3. ハイプロ Direct: https://talent.direct.hipro-job.jp/talent/

ログインしたら「完了」と教えてください。
```

---

## フェーズ4：Coworkスケジュール設定

ユーザーにこう伝える：

```
最後にCoworkでスケジュールタスクを設定します。
以下の10タスクを登録してください：

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
【チイキズカン】
08:00  スクレイピングを実行してください。チイキズカンで。
08:05  応募文を作成してください。チイキズカンで。
08:10  応募を実行してください。チイキズカンで。

【スキルシフト】
08:30  スクレイピングを実行してください。スキルシフトで。
08:35  応募文を作成してください。スキルシフトで。
08:40  応募を実行してください。スキルシフトで。

【ハイプロ】
09:00  スクレイピングを実行してください。ハイプロで。
09:05  応募文を作成してください。ハイプロで。
09:10  応募を実行してください。ハイプロで。

【スプシ同期】
09:30  スプシ同期してください。
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

設定完了したら「完了しました」と教えてください。
全部のセットアップが完了です！
```

---

## ファイル構成

```
案件自動応募/
├── CLAUDE.md              ← このファイル（セットアップ案内）
├── COWORK_SKILL.md        ← Coworkへの指示ルーター
├── SCRAPE_SKILL.md        ← Phase 1: 候補収集
├── DRAFT_SKILL.md         ← Phase 2: 応募文作成
├── APPLY_SKILL.md         ← Phase 3: フォーム送信
├── my_profile.md          ← プロフィール（セットアップで生成）
├── config.json            ← GAS URL・パス設定
├── cowork_config.json     ← 検索条件・応募基準
├── sync_sheets.py         ← Phase 4: GAS経由スプシ同期
├── applied_urls.txt       ← 応募済みURL（重複防止）
├── sites/
│   ├── skillshift/        ← サイト別設定（config.json + SKILL.md）
│   ├── hipro/
│   └── chiikizukan/
├── inbox/                 ← 各フェーズの出力（jsonl）
├── processed/             ← 処理済みファイル
├── error/                 ← エラーファイル
└── logs/                  ← 実行ログ
```

---

## 日常の運用

Coworkが毎朝自動で以下を実行：
1. 3サイトから候補を収集
2. プロフィールに合わせて応募文を自動生成
3. 各サイト最大5件ずつ応募（合計15件/日）
4. 結果をスプレッドシートに記録

**人間がやること：PCの電源を入れておく・各サイトのログインを維持するだけ。**
