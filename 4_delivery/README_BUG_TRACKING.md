# QAバグトラッキングシステム - 使用ガイド

**作成日**: 2025-10-14
**対象**: cursor_to_notion (nit) プロジェクト
**カスタムコマンド**: `/aipm/4_delivery/12-15_バグ管理`

---

## 📋 概要

QA実行中に発見したバグ・改善点・テスト失敗を体系的に管理するYAMLベースのトラッキングシステムです。

### 主要機能
- ✅ バグ・改善点の登録（BUG-XXX, IMP-XXX, TF-XXX）
- ✅ ステータス管理（New → Open → In Progress → Fixed → Verified → Closed）
- ✅ 優先度・深刻度による分類
- ✅ 詳細レポート自動生成（Markdown）
- ✅ リリース判定への自動反映

---

## 🚀 カスタムコマンド一覧

### 12. バグ登録
```
/aipm/4_delivery/12_バグ登録
```

**使用タイミング**: QA実行中にバグ・改善点を発見した直後

**生成物**:
- `bug_tracking/bugs.yaml` - バグエントリ追加
- `bug_tracking/bug_reports/{BUG_ID}.md` - 個別レポート

**対話的入力項目**:
1. バグタイプ（bug/improvement/test_failure）
2. タイトル（1行要約）
3. 深刻度（critical/high/medium/low）
4. 優先度（P0/P1/P2/P3）
5. 発見したテスト名
6. 詳細説明
7. 再現手順
8. 期待される動作
9. 実際の動作
10. エラーログ

**例**:
```bash
# 統合テスト中にInitコマンドのエラーメッセージが不親切と気づいた
→ 「バグ登録」と入力
→ type: improvement
→ title: nit initの引数エラーメッセージ改善
→ severity: medium
→ priority: P1
→ ...（対話的に入力）
```

---

### 13. バグステータス更新
```
/aipm/4_delivery/13_バグステータス更新
```

**使用タイミング**: バグ修正・検証後、ステータスを更新したい時

**更新可能項目**:
- ステータス（new/open/in_progress/fixed/verified/closed/wont_fix）
- コメント（作業履歴、修正内容）
- 担当者
- 修正バージョン
- 修正コミットハッシュ

**ステータス遷移**:
```
New → Open → In Progress → Fixed → Verified → Closed
                                              ↓
                                          Won't Fix
```

**例**:
```bash
# IMP-002の依存関係ドキュメントを追加完了
→ 「バグステータス更新」と入力
→ Bug ID: IMP-002
→ 新ステータス: fixed
→ コメント: README.mdにsetup手順追加、requirements.txtにnotion-client明記
→ 修正バージョン: v1.0.1
```

---

### 14. バグレポート生成
```
/aipm/4_delivery/14_バグレポート生成
```

**使用タイミング**: 
- 日次QAレビュー
- リリース判定前
- ステークホルダーへの報告前

**生成物**:
- `bug_tracking/reports/bug_summary_YYYYMMDD.md` - 日次サマリー
- `bug_tracking/reports/dashboard.html` - HTMLダッシュボード

**出力内容**:
- 📊 統計サマリー（総数、ステータス別、深刻度別、タイプ別）
- 🔴 Critical/Highバグ一覧（未解決のみ）
- 📋 全バグ一覧（表形式）
- 📈 トレンド分析（フェーズ別発見数）
- 🎯 リリース判定（自動評価）

**リリース判定ロジック**:
- ❌ **リリース不可**: Critical未解決 または P0未解決
- ⚠️ **条件付き可**: High未解決（Alpha/Betaリリース）
- ✅ **リリース可**: Critical/High全て解決

---

### 15. リリース判定
```
/aipm/4_delivery/15_リリース判定
```

**使用タイミング**: QA完了後、リリース可否を最終判断する時

**判定基準**:

#### MVP（最小限）
- ✅ Criticalバグ: 0件
- ✅ P0ブロッカー: 0件
- ✅ ユニットテスト成功率: 95%以上

#### GA（一般向け）
- ✅ Criticalバグ: 0件
- ✅ Highバグ: 0件
- ✅ 統合テスト1シナリオ実施
- ✅ ユニットテスト成功率: 95%以上
- ✅ 必須条件: 6/6達成

**生成物**:
- `RELEASE_DECISION_REPORT.md` - リリース判定レポート

**出力内容**:
- 🎯 リリース判定結果（承認/不可/条件付き）
- 📊 品質指標サマリー
- 🐛 バグ状況詳細
- 📋 リリース推奨形態（MVP/GA）
- 📝 リリースノート（自動生成）
- 🎯 アクションアイテム

---

## 📁 ファイル構造

```
documents/11_QA実行/bug_tracking/
├── bugs.yaml                    # マスターYAMLファイル
├── README.md                    # 使用方法
├── bug_templates/
│   └── bug_report_template.md  # テンプレート
├── scripts/
│   ├── create_bug.sh           # バグ作成スクリプト
│   ├── update_bug_status.py    # ステータス更新
│   └── generate_bug_report.py  # レポート生成
├── bug_reports/
│   ├── BUG-001.md              # 個別レポート（バグ）
│   ├── IMP-001.md              # 個別レポート（改善）
│   └── TF-001.md               # 個別レポート（テスト失敗）
├── attachments/
│   ├── screenshots/            # スクリーンショット
│   └── logs/                   # ログファイル
└── reports/
    ├── bug_summary_20251014.md # 日次サマリー
    └── dashboard.html          # HTMLダッシュボード
```

---

## 🎯 実際の使用例

### ケース1: 統合テスト中に改善点発見

**状況**: scenario1_new_project実行中、`nit init`の引数エラーが不親切と気づいた

**手順**:
1. `/aipm/4_delivery/12_バグ登録` を実行
2. 対話的に入力:
   - type: improvement
   - title: nit initの引数エラーメッセージ改善
   - severity: medium
   - priority: P1
   - found_in_test: scenario1_new_project
   - description: folder引数が必須なのにヘルプが表示されない...
   - reproduction_steps: 1. nit init を実行 / 2. エラー表示される...
3. 自動生成:
   - `bugs.yaml`にIMP-001追加
   - `bug_reports/IMP-001.md`作成

**結果**: 改善点が記録され、後で優先順位をつけて対応可能

---

### ケース2: 修正完了後のステータス更新

**状況**: IMP-002（依存関係不明確）を修正完了

**手順**:
1. `/aipm/4_delivery/13_バグステータス更新` を実行
2. 対話的に入力:
   - Bug ID: IMP-002
   - 新ステータス: fixed
   - コメント: README.mdにsetup手順追加
   - 修正バージョン: v1.0.1
   - 修正コミット: abc123f
3. 自動更新:
   - `bugs.yaml`のIMP-002ステータス更新
   - コメント履歴追加

**結果**: 修正履歴が記録され、リリースノートに自動反映

---

### ケース3: リリース前の最終確認

**状況**: 全QA完了、リリース判定したい

**手順**:
1. `/aipm/4_delivery/14_バグレポート生成` を実行
   - サマリーレポート確認
   - Critical/High: 1件（IMP-002）→ リリース不可判定
2. IMP-002を修正
3. `/aipm/4_delivery/13_バグステータス更新` でfixedに
4. `/aipm/4_delivery/14_バグレポート生成` を再実行
   - Critical/High: 0件 → リリース可判定
5. `/aipm/4_delivery/15_リリース判定` を実行
   - RELEASE_DECISION_REPORT.md生成
   - リリース承認推奨

**結果**: 自信を持ってリリース判断可能

---

## 💡 ベストプラクティス

### Do（推奨）
✅ **小さな問題も記録**: 「これは小さいから...」と思わず記録
✅ **再現手順を詳しく**: 他の人が再現できるレベルで
✅ **スクリーンショット添付**: エラー画面はattachments/screenshots/に
✅ **こまめにステータス更新**: 修正したら即座にfixed
✅ **日次レポート生成**: 毎日終業前にバグレポート生成

### Don't（非推奨）
❌ **口頭だけで伝える**: 記録しないと忘れる
❌ **曖昧な記述**: 「なんかエラー出た」→ 具体的に
❌ **優先度を適当に**: P0は本当に緊急の時だけ
❌ **ステータス放置**: fixedのまま放置しない→ verifiedへ

---

## 🔧 トラブルシューティング

### Q1: カスタムコマンドが見つからない
```bash
A: .cursor/commands/aipm/4_delivery/ 配下にファイルが存在するか確認
```

### Q2: bugs.yamlが壊れた
```bash
A: Git履歴から復元
git checkout HEAD -- bug_tracking/bugs.yaml
```

### Q3: レポート生成でエラー
```bash
A: Pythonスクリプトを直接実行して確認
cd bug_tracking/scripts
python3 generate_bug_report.py
```

---

## 📚 参考資料

- **bugs.yaml仕様**: `bug_tracking/README.md`
- **統合テスト結果**: `integration_tests/scenario1_new_project.md`
- **QA完了レポート**: `QA_COMPLETION_REPORT.md`

---

**作成日**: 2025-10-14  
**最終更新**: 2025-10-14  
**バージョン**: v1.0






















