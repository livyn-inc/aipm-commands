# 07 : 開発タスク分解（AIPMハッカソン）- ストーリーベース版

## 前提
- 直前に `03/04/05/06` を完了済み（problem/solution/story/ui/spec が存在）
- **ストーリーマップ（integrated_story_map.yaml）を基にした親子関係でのタスク分解**
- LLMが実装担当（人手ではなく、実装ステップが明確なタスク粒度にする）

## 目的
- **ストーリー単位での親子関係を持つタスク構造**を生成
- **MVP/Release1のスライスを適切に切り直し**、各ストーリーの受け入れ基準を明確化
- **Story配下にTasksを紐づける階層構造**で管理
- 各ストーリーの成功指標・測定方法を明記
- 依存関係・優先度・所要時間・成果物・テスト観点を明記
- 初期状態から各タスクに `status: TODO` を付与（以降のコマンドで DONE/IN_PROGRESS/TODO を更新）

## 実行手順
```yaml
- trigger: "(実装_開発タスク分解|DevTaskBreakdown|ストーリータスク分解)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/integrated_story_map.yaml','**/story_map*.yaml','**/solution_map.yaml','**/ui_wireframe_todo.md','**/spec_mvp.yaml']))}}"]
      instructions: |
        スレッド直近の成果物から、ストーリーマップ構造、MVP/Release1スライス、優先ストーリー、所要時間の初期値を推定し、display用の箇条書きにまとめてください。
        特に integrated_story_map.yaml の構造を重視してください。
      store_as: "auto_story_scope"
    - name: "prefill_from_artifacts"
      action: "display"
      content: |
        🔎 ストーリーベース読み込み対象
        - **integrated_story_map.yaml** / story_map_mermaid.md
        - spec_mvp.yaml
        - solution_map.yaml
        - ui_wireframe_todo.md
        
        参考サンプル（構造）: .cursor/commands/aipm/dev_tasks_sample.yaml
        
        推定ストーリースコープ:
        {{auto_story_scope}}
    
    - name: "ensure_output_dir"
      action: "execute_shell"
      command: "mkdir -p Flow/{{today}}/{{flow_dir}}/07_開発タスク分解"
    
    # 開発用ディレクトリ構造の作成
    - name: "create_dev_structure"
      action: "execute_shell"
      command: |
        mkdir -p "Flow/{{today}}/{{flow_dir}}/dev/src" && \
        mkdir -p "Flow/{{today}}/{{flow_dir}}/dev/assets" && \
        mkdir -p "Flow/{{today}}/{{flow_dir}}/dev/docs"
      message: "開発用フォルダ構造（dev/src, dev/assets, dev/docs）を作成しました"
    
    - name: "create_dev_readme"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/dev/README.md"
      content: |
        # 開発ディレクトリ
        
        ## 構造
        ```
        dev/
        ├── src/           # ソースコード（HTML/CSS/JS）
        ├── assets/        # 画像・アイコン等
        ├── docs/          # 開発ドキュメント
        └── README.md      # このファイル
        ```
        
        ## 実装ファイル
        - `src/index.html` - メインHTML
        - `src/styles.css` - スタイルシート
        - `src/app.js` - アプリケーションロジック
        
        ## サーバー（LLM/ワークフロー統合）
        - `server/index.js` - Express LLMプロキシサーバー
        - `server/.env.sample` - 環境変数サンプル
        - `server/workflow.js` - ワークフローエンジン
        
        ## 動作確認
        1. **フロントエンド**: `src/index.html` をブラウザで開く
        2. **サーバー**: `cd server && node index.js` で起動
        3. 開発者ツール（F12）でConsoleを確認
        4. 期待される動作・ログを確認
        
        ## 参照
        - トータル開発仕様書: `../07_開発タスク分解/total_development_spec.md`

    - name: "ingest_story_map_structure"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/integrated_story_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/integrated_story_map_mermaid.md']))}}",
        "{{read_files(find_files(patterns=['**/03_ソリューションマップ/solution_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/02_課題定義/problem_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/02_課題定義/customer_problem_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/05_UIワイヤーフレーム/ui_wireframe_todo.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/07_オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/spec_mvp.yaml']))}}"
      ]
      instructions: |
        読み込んだ Discovery/Focus/Spec の成果物を解析し、以下方針で「ストーリーベース開発タスク分解」を生成してください。
        
        **ストーリーベース構造の重視**:
        - integrated_story_map.yaml の構造を最優先で参照
        - 各Story（ST001, ST002, ST006, ST014, ST017等）を親として扱う
        - 各Story配下に実装Tasks（T001, T002等）を子として配置
        - MVP Priority 1/2/3 および Release1 のスライスを明確に区別
        
        **技術制約の考慮**:
        - 選択された技術スタック: {{tech_constraints}}
        - 技術制約に応じてタスク内容・成果物・テスト方法を調整
        - デフォルト（HTML/CSS/JS + Node.js Express）: localStorage、Vanilla JS、Console ログ、LLMプロキシサーバー
        - React系の場合: useState、useEffect、React DevTools
        - Node.js系の場合: Express、API エンドポイント、Postman テスト
        
        **Story単位の受け入れ基準**:
        - 各Storyに story_acceptance_criteria を明記
        - 各Storyに story_success_metrics を明記（測定方法含む）
        - Story配下のTasksがStoryの受け入れ基準を満たすことを確認
        
        **MVP/Release1スライス**:
        - MVP: ST001, ST002, ST003, ST006, ST014, ST017 等（integrated_story_map.yamlのMVP指定）
        - Release1: ST009（BigQuery統合）等（integrated_story_map.yamlのRelease1指定）
        - 各Storyの release フィールドで明示
        
        **タスク詳細度**:
        - 受け入れ基準: story_map.yaml の全ACを満たす具体的な基準
        - 成果物: 選択技術に応じた具体的な変更箇所・ファイル
        - テスト観点: 技術スタック対応のテスト方法
        - 見積: 1-6時間（適度な粒度）
        
        **トータル仕様情報も併せて抽出**:
        - ペルソナ要約（名前・課題・期待価値）
        - ストーリー要約（ST1/ST2/ST3の概要・受け入れ基準・画面対応）
        - UI設計要約（画面構成・フロー・主要要素）
        - 技術構成詳細（選択技術に応じた詳細）
        
        出力形式: {"stories_tasks_yaml": "stories: [...]", "mermaid": "ST001-->T001\n...", "summary": "要約", "persona_summary": "...", "stories_summary": [...], "ui_summary": "...", "tech_summary": "..."}
      store_as: "story_dev"
    
    - name: "collect_generation_scope"
      action: "ask_questions_with_template"
      template: |
        === ストーリーベース生成スコープ・技術制約 ===
        1) 対象リリース（MVP/Release1/両方）
        →
        2) 技術制約条件（実装方法の選択）
        選択肢:
        - HTMLとCSSと標準のJS + Node.js Express（デフォルト・推奨）
        - React + TypeScript
        - Next.js + TypeScript
        - Vue.js + TypeScript
        - Python + Flask/FastAPI
        - その他（カスタム）
        → 【デフォルト: HTMLとCSSと標準のJS + Node.js Express】
        
        3) 想定実装時間の上限（時間/日数）
        →
        4) Story優先度付与の基準（例: ペルソナ価値貢献/依存クリティカル）
        →
        =====================================
    
    - name: "wait_scope"
      action: "wait_for_all_answers"
    
    # 自動提案（ストーリーベース開発タスク）
    - name: "auto_propose_story_tasks_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks_story_proposed.yaml"
      content: |
        # ストーリーベース開発タスク（親子関係・MVP/Release1スライス）
        {{story_dev.stories_tasks_yaml}}
    
    # トータル開発仕様書の生成（チケット開始・実行で参照用）
    - name: "generate_total_development_spec"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/total_development_spec.md"
      content: |
        # トータル開発仕様書（ストーリーベース・MVP）
        
        ## 📋 プロダクト概要
        **何を作るか**: {{story_dev.summary}}
        
        ## 👤 ターゲットペルソナ（from Discovery）
        - **ペルソナ**: {{story_dev.persona_summary}}
        - **核心的課題**: {{core_problem}}
        - **期待する価値**: {{expected_value}}
        
        ## 🎯 実装するストーリー（from integrated_story_map.yaml）
        {{#each story_dev.stories_summary}}
        ### {{story_id}}: {{story_title}}
        - **ペルソナ**: {{persona}}
        - **Story受け入れ基準**: {{story_acceptance_criteria}}
        - **Story成功指標**: {{story_success_metrics}}
        - **対応画面**: {{related_screens}}
        - **リリース**: {{release}}
        - **実装優先度**: {{story_priority}}
        {{/each}}
        
        ## 🎨 UI設計概要（from Discovery）
        - **画面構成**: {{story_dev.ui_summary}}
        - **画面フロー**: {{screen_flow_summary}}
        - **主要UI要素**: {{ui_elements}}
        
        ## 🔧 技術構成
        - **技術スタック**: {{tech_constraints | default:"HTML/CSS/Vanilla JavaScript + Node.js Express"}}
        - **データ保存**: {{data_storage | default:"localStorage"}}
        - **状態管理**: {{state_management | default:"createStore + EventBus"}}
        - **ログ出力**: {{logging | default:"Console観測ログ"}}
        - **サーバー**: {{server_stack | default:"Node.js Express + LLMプロキシ"}}
        
        ## 📊 ストーリーベース開発タスク概要
        {{#each consolidated_stories}}
        ### Story {{story_id}}: {{story_title}}
        - **ペルソナ**: {{persona}}
        - **Story概要**: {{story_description}}
        - **Story成功指標**: {{story_success_metrics}}
        - **リリース**: {{release}}
        
        #### 配下実装Tasks:
        {{#each tasks}}
        - **{{id}}**: {{title}}
          - 概要: {{description}}
          - 成果物: {{outputs}}
          - 受け入れ基準: {{test_points}}
          - 見積: {{estimate_h}}時間
          - 依存: {{depends}}
        {{/each}}
        {{/each}}
        
        ## ✅ 完成時の期待状態（Story別）
        {{#each target_stories}}
        - **{{story_id}}**: {{persona}}の課題（{{core_problem}}）が{{success_metric}}で解決される
        {{/each}}
        - 全ストーリーの受け入れ基準を満たす
        - レスポンシブ対応・アクセシビリティ配慮済み
        - 本番環境での動作準備完了
        
        ---
        **重要**: 本仕様書は全チケットで参照し、実装の方向性を常に確認してください。
    
    - name: "auto_propose_story_dependency_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks_story_mermaid.md"
      content: |
        # ストーリーベース依存関係図
        
        ```mermaid
        flowchart TD
        {{story_dev.mermaid}}
        ```
    
    - name: "export_story_tasks_order"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks_order.md"
      content: |
        # 推奨実行順（ストーリーベース・依存考慮）
        
        ## フェーズ1: 基盤ストーリー（MVP Priority 1）
        1) ST001: 統合プラットフォーム（田中さん）
           - T001_FOUNDATION → T002_CONSUMER_UI → T003_AI_PLATFORM
        2) ST002: セルフサービス型システム（田中さん）
           - T004_ROI_DASHBOARD
        3) ST006: 統合タスク管理（山田さん）
           - T003_AI_PLATFORM（再利用）
        
        ## フェーズ2: 重要機能ストーリー（MVP Priority 2）
        4) ST003: ROI可視化（田中さん）
           - T020_OKR_ALIGNMENT_TRACKER
        5) ST014: 承認管理（田中さん）
           - T005_MANAGEMENT → T010_APPROVAL_STATE_MACHINE
        6) ST017: 外部ワーカー管理（高橋さん）
           - T011_WORKER_DIRECTORY
        
        ## フェーズ3: 横断機能・基盤強化
        7) 横断機能タスク群
           - T007_STORAGE_SCHEMA → T008_DATA_EXPORT_IMPORT
           - T009_FILEFLOW_UPLOAD_PREVIEW → T021_CONTEXT_FOLDER_MANAGER_UI
           - T012_ROI_CALCULATOR
        
        ## フェーズ4: LLM/WF統合
        8) サーバーサイド統合
           - T013_LLM_PROXY_SERVER → T014_WORKFLOW_ENGINE
           - T016_CONTEXT_INDEXER → T017_PROMPT_LIBRARY → T015_SAMPLE_WORKFLOWS_DOCS
           - T018_UI_WORKFLOW_BUILDER
        
        ## フェーズ5: 仕上げ・セキュリティ
        9) 統合・セキュリティ
           - T019_SECURITY_KEYS → T006_INTEGRATION
           - T022_CUSTOM_DASHBOARD_UI
        
        **重要**: まずStory単位で受け入れ基準を満たしてから次のStoryへ進んでください。
    
    - name: "display_story_proposal"
      action: "display"
      content: |
        ✍️ ストーリーベース開発タスクを自動提案しました。修正・追加・削除を入力してください（空欄は採用）。
        
        **構造**:
        - Story（ST001, ST002等）が親
        - Task（T001, T002等）が子
        - 各StoryにAcceptance Criteria・Success Metricsを明記
        - MVP/Release1スライスを区別
        
        すべてのタスクには初期 `status: TODO` を付与しています。
    
    - name: "collect_story_corrections"
      action: "ask_questions_with_template"
      template: |
        === ストーリーベース修正テンプレ ===
        1) 追加したいStory（story_id/story_title/persona/story_acceptance_criteria/release）
        →
        2) 追加したいTask（id/title/parent_story/depends/estimate_h/status）
        →
        3) 修正したいStory/Task（idと変更点）
        →
        4) 削除したいStory/TaskID
        →
        5) 依存関係の変更
        →
        =====================================
    
    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正内容で確定成果物（dev_tasks.yaml / dev_tasks_mermaid.md / dev_tasks_order.md / dev_runbook.md / tickets/*.md）をストーリーベース構造で生成します。よろしいですか？（statusは未指定時TODO）"
    
    - name: "export_story_dev_tasks_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks.yaml"
      content: |
        # ストーリーベース開発タスク（親子関係・MVP/Release1スライス対応）
        # integrated_story_map.yamlに基づく構造化タスク分解
        {{story_dev_tasks_final_yaml | default: story_dev.stories_tasks_yaml}}
    
    - name: "export_story_dev_tasks_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks_mermaid.md"
      content: |
        # ストーリーベース依存関係図
        
        ```mermaid
        flowchart TD
        {{story_dev_tasks_mermaid_final | default: story_dev.mermaid}}
        ```
    
    - name: "export_story_tickets"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/tickets/STORY_TEMPLATE.md"
      content: |
        # Story: {{story_id}}
        
        ## Story概要
        **タイトル**: {{story_title}}
        **ペルソナ**: {{persona}}
        **説明**: {{story_description}}
        
        ## Story受け入れ基準
        {{#each story_acceptance_criteria}}
        - {{.}}
        {{/each}}
        
        ## Story成功指標
        - **指標**: {{story_success_metrics}}
        - **測定方法**: {{measurement_method}}
        
        ## 配下実装Tasks
        {{#each tasks}}
        ### {{id}}: {{title}}
        - **概要**: {{description}}
        - **受け入れ基準**: {{acceptance_criteria}}
        - **成果物**: {{outputs}}
        - **見積**: {{estimate_h}}時間
        - **依存**: {{depends}}
        - **テスト観点**: {{test_points}}
        {{/each}}
        
        ## 実装ステップ（LLM向け）
        1. Story受け入れ基準を確認
        2. 配下Tasksを依存順で実装
        3. 各Task完了後、Story受け入れ基準に寄与することを確認
        4. Story全体の成功指標を測定可能な状態にする
    
    - name: "export_story_runbook"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_runbook.md"
      content: |
        # ストーリーベースLLM実装Runbook（MVP）
        
        ## 基本方針
        1) **Story単位での実装**を重視
        2) 各Story配下のTasksを依存順で着手
        3) Story受け入れ基準を満たしてから次のStoryへ
        4) MVP Priority 1 → 2 → 3 → 横断機能 → Release1 の順
        
        ## 実装フロー
        1) Story選択（ST001から開始推奨）
        2) Story受け入れ基準・成功指標を確認
        3) 配下Tasks（T001, T002等）を依存順で実装
        4) 各Task完了後、Story受け入れ基準への寄与を確認
        5) Story全体の成功指標を測定可能な状態にする
        6) `integrated_story_map.yaml` との整合性チェック
        7) 次Storyへ進む
        
        ## チェックポイント（各Task共通）
        - 受け入れ基準（AC）を満たす
        - 期待ログ（Console）を確認
          - `[click] ...` / `[state] ...` / `[wf] ...` / `[error] ...`
        - ブラウザ: DevTools Consoleにエラーがない
        - 仕様整合: `total_development_spec.md` の期待状態に合致
        - **Story寄与**: 親Storyの受け入れ基準に寄与している
        
        ## サーバー関連（LLM/WF統合）
        - `dev/server/.env` の `PORT` とキー設定を確認
        - `node index.js` で起動（:8787推奨）
        - `/api/chat`, `/api/workflow/run` のcurlで疎通確認
        - LLMプロキシ・ワークフローエンジンの動作確認
        
        ## Story別優先順位
        ### MVP Priority 1（最重要）
        - ST001: 統合プラットフォーム（田中さん）
        - ST002: セルフサービス型システム（田中さん）
        - ST006: 統合タスク管理（山田さん）
        
        ### MVP Priority 2（重要）
        - ST003: ROI可視化（田中さん）
        - ST014: 承認管理（田中さん）
        - ST017: 外部ワーカー管理（高橋さん）
        
        ### 横断機能・基盤強化
        - ストレージ・データ管理
        - ファイルフロー・コンテキスト管理
        - LLM/ワークフロー統合
        - セキュリティ・運用
    
    - name: "notify_story_completion"
      action: "display"
      content: |
        ✅ ストーリーベース開発タスクを生成しました：
        - 07_開発タスク分解/dev_tasks.yaml（Story親子構造・status付き）
        - 07_開発タスク分解/dev_tasks_mermaid.md（Story依存関係図）
        - 07_開発タスク分解/dev_tasks_order.md（Story優先順位）
        - 07_開発タスク分解/tickets/（Story・Taskテンプレ）
        - 07_開発タスク分解/dev_runbook.md（ストーリーベース実装ガイド）
        - 07_開発タスク分解/total_development_spec.md（総合仕様書）
        
        **次のステップ**:
        1. Story受け入れ基準・成功指標を確認
        2. `08_チケット開始` でStory選択（ST001推奨）
        3. Story配下のTasksを順次実装
        4. Story単位で受け入れ基準達成を確認
```

## 次のコマンド
→ `08_チケット開始` でStory選択・着手タスク選択へ

## 変更点（v2.0 - ストーリーベース対応）
- **ストーリーマップ構造の重視**: integrated_story_map.yaml を最優先参照
- **親子関係の明確化**: Story（親）→ Tasks（子）の階層構造
- **Story受け入れ基準**: 各Storyに story_acceptance_criteria を明記
- **Story成功指標**: 各Storyに story_success_metrics・測定方法を明記
- **MVP/Release1スライス**: リリース計画の明確化
- **ペルソナ連携**: 各StoryにPersona情報を紐づけ
- **技術統合**: HTML/CSS/JS + Node.js Express（LLMプロキシ）をデフォルト化
        →
        =====================================
    
    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正内容で確定成果物（dev_tasks.yaml / dev_tasks_mermaid.md / dev_tasks_order.md / dev_runbook.md / tickets/*.md）をストーリーベース構造で生成します。よろしいですか？（statusは未指定時TODO）"
    
    - name: "export_story_dev_tasks_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks.yaml"
      content: |
        # ストーリーベース開発タスク（親子関係・MVP/Release1スライス対応）
        # integrated_story_map.yamlに基づく構造化タスク分解
        {{story_dev_tasks_final_yaml | default: story_dev.stories_tasks_yaml}}
    
    - name: "export_story_dev_tasks_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_tasks_mermaid.md"
      content: |
        # ストーリーベース依存関係図
        
        ```mermaid
        flowchart TD
        {{story_dev_tasks_mermaid_final | default: story_dev.mermaid}}
        ```
    
    - name: "export_story_tickets"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/tickets/STORY_TEMPLATE.md"
      content: |
        # Story: {{story_id}}
        
        ## Story概要
        **タイトル**: {{story_title}}
        **ペルソナ**: {{persona}}
        **説明**: {{story_description}}
        
        ## Story受け入れ基準
        {{#each story_acceptance_criteria}}
        - {{.}}
        {{/each}}
        
        ## Story成功指標
        - **指標**: {{story_success_metrics}}
        - **測定方法**: {{measurement_method}}
        
        ## 配下実装Tasks
        {{#each tasks}}
        ### {{id}}: {{title}}
        - **概要**: {{description}}
        - **受け入れ基準**: {{acceptance_criteria}}
        - **成果物**: {{outputs}}
        - **見積**: {{estimate_h}}時間
        - **依存**: {{depends}}
        - **テスト観点**: {{test_points}}
        {{/each}}
        
        ## 実装ステップ（LLM向け）
        1. Story受け入れ基準を確認
        2. 配下Tasksを依存順で実装
        3. 各Task完了後、Story受け入れ基準に寄与することを確認
        4. Story全体の成功指標を測定可能な状態にする
    
    - name: "export_story_runbook"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_開発タスク分解/dev_runbook.md"
      content: |
        # ストーリーベースLLM実装Runbook（MVP）
        
        ## 基本方針
        1) **Story単位での実装**を重視
        2) 各Story配下のTasksを依存順で着手
        3) Story受け入れ基準を満たしてから次のStoryへ
        4) MVP Priority 1 → 2 → 3 → 横断機能 → Release1 の順
        
        ## 実装フロー
        1) Story選択（ST001から開始推奨）
        2) Story受け入れ基準・成功指標を確認
        3) 配下Tasks（T001, T002等）を依存順で実装
        4) 各Task完了後、Story受け入れ基準への寄与を確認
        5) Story全体の成功指標を測定可能な状態にする
        6) `integrated_story_map.yaml` との整合性チェック
        7) 次Storyへ進む
        
        ## チェックポイント（各Task共通）
        - 受け入れ基準（AC）を満たす
        - 期待ログ（Console）を確認
          - `[click] ...` / `[state] ...` / `[wf] ...` / `[error] ...`
        - ブラウザ: DevTools Consoleにエラーがない
        - 仕様整合: `total_development_spec.md` の期待状態に合致
        - **Story寄与**: 親Storyの受け入れ基準に寄与している
        
        ## サーバー関連（LLM/WF統合）
        - `dev/server/.env` の `PORT` とキー設定を確認
        - `node index.js` で起動（:8787推奨）
        - `/api/chat`, `/api/workflow/run` のcurlで疎通確認
        - LLMプロキシ・ワークフローエンジンの動作確認
        
        ## Story別優先順位
        ### MVP Priority 1（最重要）
        - ST001: 統合プラットフォーム（田中さん）
        - ST002: セルフサービス型システム（田中さん）
        - ST006: 統合タスク管理（山田さん）
        
        ### MVP Priority 2（重要）
        - ST003: ROI可視化（田中さん）
        - ST014: 承認管理（田中さん）
        - ST017: 外部ワーカー管理（高橋さん）
        
        ### 横断機能・基盤強化
        - ストレージ・データ管理
        - ファイルフロー・コンテキスト管理
        - LLM/ワークフロー統合
        - セキュリティ・運用
    
    - name: "notify_story_completion"
      action: "display"
      content: |
        ✅ ストーリーベース開発タスクを生成しました：
        - 07_開発タスク分解/dev_tasks.yaml（Story親子構造・status付き）
        - 07_開発タスク分解/dev_tasks_mermaid.md（Story依存関係図）
        - 07_開発タスク分解/dev_tasks_order.md（Story優先順位）
        - 07_開発タスク分解/tickets/（Story・Taskテンプレ）
        - 07_開発タスク分解/dev_runbook.md（ストーリーベース実装ガイド）
        - 07_開発タスク分解/total_development_spec.md（総合仕様書）
        
        **次のステップ**:
        1. Story受け入れ基準・成功指標を確認
        2. `08_チケット開始` でStory選択（ST001推奨）
        3. Story配下のTasksを順次実装
        4. Story単位で受け入れ基準達成を確認
```

## 次のコマンド
→ `08_チケット開始` でStory選択・着手タスク選択へ
## 変更点（v2.0 - ストーリーベース対応）
- **ストーリーマップ構造の重視**: integrated_story_map.yaml を最優先参照
- **親子関係の明確化**: Story（親）→ Tasks（子）の階層構造
- **Story受け入れ基準**: 各Storyに story_acceptance_criteria を明記
- **Story成功指標**: 各Storyに story_success_metrics・測定方法を明記
- **MVP/Release1スライス**: リリース計画の明確化
- **ペルソナ連携**: 各StoryにPersona情報を紐づけ
- **技術統合**: HTML/CSS/JS + Node.js Express（LLMプロキシ）をデフォルト化
