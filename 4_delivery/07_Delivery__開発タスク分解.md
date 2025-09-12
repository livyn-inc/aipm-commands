# 07 実装: 開発タスク分解（AIPMハッカソン）

## 前提
- 直前に `03/04/05/06` を完了済み（problem/solution/story/ui/spec が存在）
- LLMが実装担当（人手ではなく、実装ステップが明確なタスク粒度にする）

## 目的
- MVP/Release1を「受け入れ基準を満たして完全に動く」状態に到達できる開発タスク一覧を生成
- 共通タスク + ストーリー別タスク + 非機能タスクを網羅
- 依存関係・優先度・所要時間・成果物・テスト観点を明記
- 初期状態から各タスクに `status: TODO` を付与（以降のコマンドで DONE/IN_PROGRESS/TODO を更新）

## 実行手順
```yaml
- trigger: "(実装_開発タスク分解|DevTaskBreakdown)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/story_map.yaml','**/solution_map.yaml','**/ui_wireframe_todo.md','**/spec_mvp.yaml']))}}"]
      instructions: |
        スレッド直近の成果物から、対象リリース（MVP/Release1）、優先ストーリー、所要時間の初期値を推定し、display用の箇条書きにまとめてください。
      store_as: "auto_dev_scope"
    - name: "prefill_from_artifacts"
      action: "display"
      content: |
        🔎 読み込み対象
        - story_map.yaml / story_map_mermaid.md
        - spec_mvp.yaml
        - solution_map.yaml
        - ui_wireframe_todo.md
        
        参考サンプル（構造）: .cursor/commands/aipm/dev_tasks_sample.yaml
        
        推定スコープ:
        {{auto_dev_scope}}
    
    - name: "ensure_output_dir"
      action: "execute_shell"
      command: "mkdir -p Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解"

    - name: "ingest_discovery_focus"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/04_仮説駆動__ストーリーマップ/story_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/03_仮説駆動__ソリューションマップ/solution_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/02_仮説駆動__課題定義/problem_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/02_仮説駆動__課題定義/customer_problem_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/05_仮説駆動__UIワイヤーフレーム/ui_wireframe_todo.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/07_sense__オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/spec_mvp.yaml']))}}"
      ]
      instructions: |
        読み込んだ Discovery/Focus/Spec の成果物を解析し、以下方針で「開発タスク案」を生成してください。
        - 共通タスク: 環境準備/ストレージ/ユーティリティ等（重複は統合）。
        - Story別タスク: story_map.yaml の STごとに、受け入れ基準と solution_map.yaml のリンクを参照し、UI/状態/ロジック/テスト観点に分解。
        - 非機能タスク: UIワイヤ/Spec/Senseの示唆をもとにアクセシビリティ/性能等を抽出。
        - 依存関係: 共通→各ST→非機能の順を基本とし、リンク・必要モジュールから依存を推定。
        - 見積: 受け入れ基準項目数と変更範囲に基づき軽量推定（0.5〜3.0h単位）。
        - 出力形式は下記YAML文字列 dev_tasks を返すこと（statusはすべてTODO）。
        さらに Mermaid用の依存エッジ列（例: "T_A --> T_B" の改行連結）を mermaid に出力。
        JSON1件で返す: {"tasks_yaml": "dev_tasks: ...", "mermaid": "T1-->T2\n...", "summary": "要約"}
      store_as: "dev"
    
    - name: "collect_generation_scope"
      action: "ask_questions_with_template"
      template: |
        === 生成スコープ ===
        1) 対象リリース（MVP/Release1/両方）
        →
        2) 想定実装時間の上限（時間/日数）
        →
        3) 優先度付与の基準（例: 受入基準貢献/依存クリティカル）
        →
        =====================================
    
    - name: "wait_scope"
      action: "wait_for_all_answers"
    
    # 自動提案（共通/各ST/非機能）
    - name: "auto_propose_dev_tasks_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/dev_tasks_proposed.yaml"
      content: |
        {{dev.tasks_yaml}}
    
    - name: "auto_propose_dependency_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/dev_tasks_proposed_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
        {{dev.mermaid}}
        ```
    
    - name: "export_dev_tasks_order"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/dev_tasks_order.md"
      content: |
        # 推奨実行順（依存考慮）
        1) Common: T_COMMON_ENV → T_COMMON_STORAGE
        2) Stories: ST2系（T_ST2_MODE → T_ST2_FILTER）
        3) Stories: ST1系（T_ST1_VIEW → T_ST1_PRIORITY）
        4) Stories: ST3系（T_ST3_TIME_MATCH → T_ST3_PROGRESS）
        5) Non-Functional: T_NFR_ACCESSIBILITY → T_NFR_PERF
        
        まずCommonを完了させてからStoriesへ進んでください。
    
    - name: "display_proposal"
      action: "display"
      content: |
        ✍️ 自動提案の開発タスクを出力しました。修正・追加・削除を入力してください（空欄は採用）。
        すべてのタスクには初期 `status: TODO` を付与しています。
    
    - name: "collect_corrections"
      action: "ask_questions_with_template"
      template: |
        === 修正テンプレ ===
        1) 追加したいタスク（id/title/depends/estimate_h/status）
        →
        2) 修正したいタスク（idと変更点）
        →
        3) 削除したいタスクID
        →
        4) 依存の変更
        →
        =====================================
    
    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正内容で確定成果物（dev_tasks.yaml / dev_tasks_mermaid.md / dev_tasks_order.md / dev_runbook.md / tickets/*.md）を生成します。よろしいですか？（statusは未指定時TODO）"
    
    - name: "export_dev_tasks_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/dev_tasks.yaml"
      content: |
        # すべてのタスクは `status` を持ちます（未指定は TODO）
        {{dev_tasks_final_yaml | default: dev.tasks_yaml}}
    
    - name: "export_dev_tasks_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/dev_tasks_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
        {{dev_tasks_mermaid_final | default: dev.mermaid}}
        ```
    
    - name: "export_tickets"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/tickets/TEMPLATE.md"
      content: |
        # チケット: {{task_id}}
        
        ## 概要
        {{title}}
        
        ## 受け入れ基準
        - {{ac1}}
        - {{ac2}}
        - {{ac3}}
        
        ## 実装ステップ（LLM向け）
        1. 変更対象ファイルを列挙
        2. 差分をコードブロックで提示
        3. 保存→再レンダリング→評価観点で自己テスト
        
        ## 依存
        {{depends}}
        
        ## 成果物
        {{outputs}}
    
    - name: "export_runbook"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/07_実装__開発タスク分解/dev_runbook.md"
      content: |
        # LLM実装Runbook（MVP）
        
        1) 共通タスク（T_COMMON_ENV → T_COMMON_STORAGE）を先に完了
        2) 各STのタスクを依存順で着手
        3) 各タスク完了後、受け入れ基準の自己テストを実施
        4) `spec_mvp.yaml` と整合性チェック
        5) すべて完了後に成果物リストを確認
    
    - name: "notify_completion"
      action: "display"
      content: |
        ✅ 生成しました：
        - 07_実装__開発タスク分解/dev_tasks.yaml（status付き） / dev_tasks_mermaid.md / dev_tasks_order.md
        - 07_実装__開発タスク分解/tickets/（テンプレ＋必要に応じて各タスク）
        - 07_実装__開発タスク分解/dev_runbook.md
```

## 次のコマンド
→ `08_実装__チケット開始` で着手タスク選択へ
