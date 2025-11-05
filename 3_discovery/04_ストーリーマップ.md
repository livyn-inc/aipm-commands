# 04 ストーリーマップ（複数ペルソナ・ソリューション統合版）

## 実行モード
### 🚀 **複数ペルソナモード**（4ペルソナのソリューションマップがある場合）
4ペルソナ（田中・佐藤・山田・高橋）のソリューションマップ（SL001-SL039）とOpportunity仮説（FH001-FH020）からユーザーストーリーを自動生成し、MVP優先度に基づくストーリーマップを作成

### 📝 **単一ペルソナモード**（1ペルソナのみの場合）
従来の質問形式で単一ペルソナのユーザーストーリーを収集し、A→PR→SL→STの流れで可視化

## 複数ペルソナモード - 自動検出対象
- `P001_田中_雅人/solution_map_P001.yaml` - AI推進リーダーソリューション
- `P002_佐藤_事業責任者/solution_map_P002.yaml` - 経営層ソリューション
- `P003_山田_健太/solution_map_P003.yaml` - 現場PdMソリューション
- `P004_高橋_美咲/solution_map_P004.yaml` - コンテンツ運用ソリューション
- `integrated_solution_map.yaml` - 統合ソリューション・MVP優先度
- `focus_opportunity_hypotheses.yaml` - FH001-FH020仮説マッピング

## 単一ペルソナモード - 質問項目
- ペルソナ（既定: P001）（必須）
- PR1/PR2/PR3 それぞれに対するユーザーストーリー（As a / I want / So that）（必須）
- 各ストーリーの受け入れ基準（必須）
- リリース区分（MVP or Release1）（必須）
- 実装順序（任意）

## 目的
複数ペルソナのソリューション（SL）に対するユーザーストーリー（ST）を統合的に設計し、MVP優先度に基づくリリース計画・実装順序を明確化します。A→PR→SL→STの完全なトレーサビリティを確保。

## 実行手順
```yaml
- trigger: "(ストーリーマップ|仮説駆動_ストーリーマップ|HypothesisStoryMap|Story Map)"
  priority: high
  steps:
    # Step 1: モード判定（複数ペルソナソリューションマップの存在確認）
    - name: "check_solution_maps"
      action: "analyze"
      data: [
        "{{find_files(patterns=['**/P001_*/solution_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/P002_*/solution_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/P003_*/solution_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/P004_*/solution_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/integrated_solution_map.yaml'])}}"
      ]
      instructions: |
        ソリューションマップファイル存在確認結果から実行モードを判定してください：
        - 3つ以上のペルソナソリューションマップ + 統合ファイルが存在: "multi_persona"
        - それ以下の場合: "single_persona"
        
        結果をJSONで返してください：
        {"mode": "multi_persona|single_persona", "found_solution_maps": ["P001", "P002", ...], "has_integrated": true/false}
      store_as: "execution_mode"

    # === 複数ペルソナモード ===
    - name: "load_solution_data"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/P001_*/solution_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P003_*/solution_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P004_*/solution_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/integrated_solution_map.yaml']))}}",
        "{{read_files(find_files(patterns=['**/focus_opportunity_hypotheses.yaml']))}}"
      ]
      instructions: |
        4ペルソナのソリューションマップと統合ソリューションからユーザーストーリーを生成し、以下の構造で返してください：
        {
          "project_name": "プロジェクト名（任意なら 'Untitled'）",
          "activities": [
            {"id":"ACT-ONBOARD","name":"Onboarding"},
            {"id":"ACT-TASKS","name":"Tasks"}
          ],
          "backbones": [
            {"id":"BB-SIGNUP","name":"Sign up","sequence":1,"activity_id":"ACT-ONBOARD"},
            {"id":"BB-FLOW","name":"Create Task","sequence":2,"activity_id":"ACT-TASKS"}
          ],
          "personas": [
            {"key":"P001","name":"田中 雅人","stories":[{"id":"ST-001","story":"...","backbone_id":"BB-SIGNUP","version":"MVP","status":"TODO"}]} 
          ],
          "cross_persona_stories": [
            {"id":"CST-001","story":"横断ストーリー","backbone_id":"BB-FLOW","version":"V1","status":"TODO"}
          ],
          "display_order": ["BB-SIGNUP","BB-FLOW"],
          "story_mapping": {"ST-001": {"backbone_id":"BB-SIGNUP","sequence":1}},
          "traceability": {
            "problem_refs": ["PR1","PR2"],
            "links": [ {"chain": ["A101","PR1","SL1","ST-001"]} ]
          }
        }
      store_as: "multi_persona_stories"

    # === 単一ペルソナモード ===
    - name: "infer_defaults_from_thread"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/solution_map.yaml','**/problem_map.yaml','**/customer_problem_map.yaml']))}}"]
      instructions: |
        直近のPR/SL/Activity対応から、PR→SL→STの初期チェーン（ST1/2/3）を推定して表示用テキストにまとめてください。
      store_as: "auto_story_hints"

    - name: "ingest_sense_focus"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/1_sense/07_オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/06_リサーチサマリー（全体）/draft_research_summary.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/02_顧客調査/sense_customer_research.md']))}}",
        "{{read_files(find_files(patterns=['**/03_ソリューションマップ/solution_map.yaml']))}}"
      ]
      instructions: |
        Sense/Focusと確定済みソリューションから、PR→SL→STの候補ストーリー（As/I want/So that）、受け入れ基準の雛形、推奨リリース（MVP/Release1）を抽出し、JSON1件で返してください。
        keys:
          st1_summary_hint, st1_ac_hint, st1_release_hint,
          st2_summary_hint, st2_ac_hint, st2_release_hint,
          st3_summary_hint, st3_ac_hint, st3_release_hint
      store_as: "sense_focus"
    - name: "prefill_from_yaml"
      action: "display"
      content: |
        🔎 候補（自動抽出）
        {{auto_story_hints}}
        
        🔎 Sense/Focus 由来の候補
        - ST1: {{sense_focus.st1_summary_hint}} / AC: {{sense_focus.st1_ac_hint}} / リリース: {{sense_focus.st1_release_hint}}
        - ST2: {{sense_focus.st2_summary_hint}} / AC: {{sense_focus.st2_ac_hint}} / リリース: {{sense_focus.st2_release_hint}}
        - ST3: {{sense_focus.st3_summary_hint}} / AC: {{sense_focus.st3_ac_hint}} / リリース: {{sense_focus.st3_release_hint}}
    
    - name: "show_story_examples"
      action: "display"
      content: |
        📝 ユーザーストーリー書式
        As a [誰が], I want [何をしたい], So that [なぜ/価値]
        例: As a 共働きママ, I want 「今やる1つ」だけを見る, So that 迷わず着手できる
    
    - name: "collect_story_inputs"
      action: "ask_questions_with_template"
      template: |
        === PR連動ストーリー入力テンプレート ===
        1) PR1 → ST1
        - ST1（As/I want/So that）
        （候補: {{sense_focus.st1_summary_hint}}）
        →
        - 受け入れ基準（箇条書き）
        （候補: {{sense_focus.st1_ac_hint}}）
        →
        - リリース（MVP/Release1）
        （候補: {{sense_focus.st1_release_hint}}）
        →
        
        2) PR2 → ST2
        - ST2（As/I want/So that）
        （候補: {{sense_focus.st2_summary_hint}}）
        →
        - 受け入れ基準
        （候補: {{sense_focus.st2_ac_hint}}）
        →
        - リリース（MVP/Release1）
        （候補: {{sense_focus.st2_release_hint}}）
        →
        
        3) PR3 → ST3
        - ST3（As/I want/So that）
        （候補: {{sense_focus.st3_summary_hint}}）
        →
        - 受け入れ基準
        （候補: {{sense_focus.st3_ac_hint}}）
        →
        - リリース（MVP/Release1）
        （候補: {{sense_focus.st3_release_hint}}）
        →
        
        4) 実装順序（任意。例: ST1→ST2→ST3）
        →
        =====================================
    
    - name: "wait_inputs"
      action: "wait_for_all_answers"
    
    # ここから：Viewer対応の統合YAML + 既存のトレーサビリティYAMLを生成
    - name: "export_viewer_integrated_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/viewer_story_map.yaml"
      content: |
        integrated_story_map:
          meta:
            project_name: {{project_name | default:"Untitled Project"}}
          version_definitions:
            order: [MVP, V1, V2]
          story_map_structure:
            activities:
              - id: ACT-ONBOARD
                name: Onboarding
              - id: ACT-TASKS
                name: Tasks
            backbones:
              - id: BB-SIGNUP
                name: Sign up
                sequence: 1
                activity_id: ACT-ONBOARD
              - id: BB-FLOW
                name: Create Task
                sequence: 2
                activity_id: ACT-TASKS
          display_order:
            backbones: [BB-SIGNUP, BB-FLOW]
          personas_stories:
            P001:
              name: 主要ペルソナ
              stories:
                - id: ST-001
                  story: {{st1_summary | default:"I want to register"}}
                  backbone_id: BB-SIGNUP
                  version: MVP
                  status: TODO
                  backbone_x_version_sort: 1
                - id: ST-002
                  story: {{st2_summary | default:"I want to login"}}
                  backbone_id: BB-SIGNUP
                  version: MVP
                  status: TODO
                  backbone_x_version_sort: 2
            P002:
              name: セカンダリ
              stories:
                - id: ST-003
                  story: {{st3_summary | default:"I want to create a task"}}
                  backbone_id: BB-FLOW
                  version: V1
                  status: TODO
                  backbone_x_version_sort: 1
          cross_persona_stories:
            - id: ST-004
              story: I want to see dashboard
              backbone_id: BB-FLOW
              version: V2
              status: TODO
              backbone_x_version_sort: 2
          story_mapping:
            ST-001: { backbone_id: BB-SIGNUP, sequence: 1 }
            ST-002: { backbone_id: BB-SIGNUP, sequence: 2 }
            ST-003: { backbone_id: BB-FLOW, sequence: 1 }
            ST-004: { backbone_id: BB-FLOW, sequence: 2 }

    # 既存のトレーサビリティ表現（Problem/Links）は維持
    - name: "auto_propose_story_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/story_map_proposed.yaml"
      content: |
        story_map:
          persona_ref: {{persona_id | default:"P001"}}
          problem_refs: [PR1, PR2, PR3]
          links:
            - chain: [A101, PR1, SL1, ST1]
            - chain: [A201, PR2, SL2, ST2]
            - chain: [A301, PR3, SL3, ST3]
          stories:
            - id: ST1
              summary: {{st1_summary}}
              acceptance: [{{st1_ac_1}}, {{st1_ac_2}}, {{st1_ac_3}}]
              release: {{st1_release | default:"MVP"}}
              links: [PR1, SL1, A101]
            - id: ST2
              summary: {{st2_summary}}
              acceptance: [{{st2_ac_1}}, {{st2_ac_2}}, {{st2_ac_3}}]
              release: {{st2_release | default:"MVP"}}
              links: [PR2, SL2, A201]
            - id: ST3
              summary: {{st3_summary}}
              acceptance: [{{st3_ac_1}}, {{st3_ac_2}}, {{st3_ac_3}}]
              release: {{st3_release | default:"Release1"}}
              links: [PR3, SL3, A301]
          releases:
            - name: MVP
              includes: [ST1, ST2]
            - name: Release1
              includes: [ST3]
          numbering_policy:
            - prefix: ST=Story

    - name: "auto_propose_story_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/story_map_proposed_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          A101:::action --> PR1:::problem --> SL1:::solution --> ST1[ST1]:::story
          A201:::action --> PR2:::problem --> SL2:::solution --> ST2[ST2]:::story
          A301:::action --> PR3:::problem --> SL3:::solution --> ST3[ST3]:::story
          classDef action fill:#fff3e0
          classDef problem fill:#ffebee
          classDef solution fill:#e8f5e8
          classDef story fill:#e3f2fd
        ```

    - name: "display_proposal"
      action: "display"
      content: |
        ✍️ 自動提案を `viewer_story_map.yaml` / `story_map_proposed.yaml` / `story_map_proposed_mermaid.md` に出力しました。修正事項を入力してください（空欄は採用）。

    - name: "collect_story_corrections"
      action: "ask_questions_with_template"
      template: |
        === 修正テンプレ（空欄は採用） ===
        ST1 要約/AC/リリース/リンク（例: PR1,SL1,A101）
        →
        ST2 要約/AC/リリース/リンク
        →
        ST3 要約/AC/リリース/リンク
        →
        実装順序（任意）：
        →
        =====================================

    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正内容で最終成果物（viewer_story_map.yaml / story_map_todo.md / story_map.yaml / story_map_mermaid.md）を生成し、検証します。よろしいですか？"

    # Viewer互換の最終YAML（上書き）
    - name: "export_viewer_integrated_yaml_final"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/viewer_story_map.yaml"
      content: |
        integrated_story_map:
          meta:
            project_name: {{project_name | default:"Untitled Project"}}
          version_definitions:
            order: [MVP, V1, V2]
          story_map_structure:
            activities:
              - id: ACT-ONBOARD
                name: Onboarding
              - id: ACT-TASKS
                name: Tasks
            backbones:
              - id: BB-SIGNUP
                name: Sign up
                sequence: 1
                activity_id: ACT-ONBOARD
              - id: BB-FLOW
                name: Create Task
                sequence: 2
                activity_id: ACT-TASKS
          display_order:
            backbones: [BB-SIGNUP, BB-FLOW]
          personas_stories:
            P001:
              name: 主要ペルソナ
              stories:
                - id: ST-001
                  story: {{st1_summary | default:"I want to register"}}
                  backbone_id: BB-SIGNUP
                  version: MVP
                  status: TODO
                  backbone_x_version_sort: 1
                - id: ST-002
                  story: {{st2_summary | default:"I want to login"}}
                  backbone_id: BB-SIGNUP
                  version: MVP
                  status: TODO
                  backbone_x_version_sort: 2
            P002:
              name: セカンダリ
              stories:
                - id: ST-003
                  story: {{st3_summary | default:"I want to create a task"}}
                  backbone_id: BB-FLOW
                  version: V1
                  status: TODO
                  backbone_x_version_sort: 1
          cross_persona_stories:
            - id: ST-004
              story: I want to see dashboard
              backbone_id: BB-FLOW
              version: V2
              status: TODO
              backbone_x_version_sort: 2
          story_mapping:
            ST-001: { backbone_id: BB-SIGNUP, sequence: 1 }
            ST-002: { backbone_id: BB-SIGNUP, sequence: 2 }
            ST-003: { backbone_id: BB-FLOW, sequence: 1 }
            ST-004: { backbone_id: BB-FLOW, sequence: 2 }

    # === 検証ステップ（SchemaValidator.js を使用） ===
    - name: "validate_viewer_yaml"
      action: "execute_shell"
      command: |
        node -e "(async()=>{const fs=require('fs');const yaml=require('js-yaml');const path=require('path');const {default:Validator}=await import('file:///Users/daisukemiyata/aipm_v3/Stock/programs/Tools/projects/story_map_viewer/modules/SchemaValidator.js');const p=process.argv[2];const text=fs.readFileSync(p,'utf-8');const data=yaml.load(text);const errs=Validator.validate(data);console.log(JSON.stringify({errors:errs},null,2));process.exit(0);})( )" Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/viewer_story_map.yaml | cat
      message: "Viewer用YAMLのスキーマ検証を実行します（エラーがあれば表示します）"

    - name: "generate_story_map_md"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/story_map_todo.md"
      content: |
        # アプリ MVPストーリーマップ
        
        ## 優先度順ユーザーストーリー（PR連動）
        1. {{st1_summary}}（受け入れ基準: {{st1_ac_1}} / {{st1_ac_2}} / {{st1_ac_3}}、リリース: {{st1_release | default:"MVP"}}）
        2. {{st2_summary}}（受け入れ基準: {{st2_ac_1}} / {{st2_ac_2}} / {{st2_ac_3}}、リリース: {{st2_release | default:"MVP"}}）
        3. {{st3_summary}}（受け入れ基準: {{st3_ac_1}} / {{st3_ac_2}} / {{st3_ac_3}}、リリース: {{st3_release | default:"Release1"}}）
        
        ## 実装順序
        {{impl_order | default:"ST1→ST2→ST3"}}

    - name: "export_story_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/story_map.yaml"
      content: |
        story_map:
          persona_ref: {{persona_id | default:"P001"}}
          problem_refs: [PR1, PR2, PR3]
          links:
            - chain: [A101, PR1, SL1, ST1]
            - chain: [A201, PR2, SL2, ST2]
            - chain: [A301, PR3, SL3, ST3]
          stories:
            - id: ST1
              summary: {{st1_summary}}
              acceptance: [{{st1_ac_1}}, {{st1_ac_2}}, {{st1_ac_3}}]
              release: {{st1_release | default:"MVP"}}
              links: [PR1, SL1, A101]
            - id: ST2
              summary: {{st2_summary}}
              acceptance: [{{st2_ac_1}}, {{st2_ac_2}}, {{st2_ac_3}}]
              release: {{st2_release | default:"MVP"}}
              links: [PR2, SL2, A201]
            - id: ST3
              summary: {{st3_summary}}
              acceptance: [{{st3_ac_1}}, {{st3_ac_2}}, {{st3_ac_3}}]
              release: {{st3_release | default:"Release1"}}
              links: [PR3, SL3, A301]
          releases:
            - name: MVP
              includes: [ST1, ST2]
            - name: Release1
              includes: [ST3]
          numbering_policy:
            - prefix: ST=Story

    - name: "export_story_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_ストーリーマップ/story_map_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          A101:::action --> PR1:::problem --> SL1:::solution --> ST1[ST1]:::story
          A201:::action --> PR2:::problem --> SL2:::solution --> ST2[ST2]:::story
          A301:::action --> PR3:::problem --> SL3:::solution --> ST3[ST3]:::story
          classDef action fill:#fff3e0
          classDef problem fill:#ffebee
          classDef solution fill:#e8f5e8
          classDef story fill:#e3f2fd
        ```

    - name: "notify_completion"
      action: "display"
      content: |
        ✅ 生成しました：
        - viewer_story_map.yaml（Viewer対応）
        - story_map_proposed.yaml / story_map_proposed_mermaid.md（トレーサビリティ用）
        - story_map_todo.md / story_map.yaml / story_map_mermaid.md
```

## 次のコマンド
→ `仮説駆動_UIワイヤーフレーム` で画面設計へ
