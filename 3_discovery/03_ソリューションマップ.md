# 03 ソリューションマップ（複数ペルソナ・課題統合版）

## 実行モード
### 🚀 **複数ペルソナモード**（4ペルソナの課題マップがある場合）
4ペルソナ（田中・佐藤・山田・高橋）の課題マップ（PR001-PR106）とOpportunity仮説（FH001-FH020）から解決策を自動生成し、ペルソナ別・統合ソリューションマップを作成

### 📝 **単一ペルソナモード**（1ペルソナのみの場合）
従来の質問形式で単一ペルソナの解決策を収集し、PR→SL→Aの接続で可視化

## 複数ペルソナモード - 自動検出対象
- `P001_田中_雅人/problem_map_P001.yaml` - AI推進リーダー課題
- `P002_佐藤_事業責任者/problem_map_P002.yaml` - 経営層課題
- `P003_山田_健太/problem_map_P003.yaml` - 現場PdM課題
- `P004_高橋_美咲/problem_map_P004.yaml` - コンテンツ運用課題
- `focus_opportunity_hypotheses.yaml` - FH001-FH020仮説マッピング
- `hackathon_insights_additional_problems.yaml` - ハッカソン実証課題

## 単一ペルソナモード - 質問項目
- 対象ペルソナと主要課題群は？（必須・PR基準）
- PR1/PR2/PR3 それぞれへの解決案（必須・1つずつ）
- 各解決案が主に効くフェーズ（任意: 利用前/利用中/利用後）
- 参考となる関連Activity（任意: AID。空欄なら自動候補を採用）

## 目的
複数ペルソナの課題（PR）に対する解決策（SL）を統合的に設計し、Opportunity仮説と対応づけたソリューションマップを生成。ペルソナ間の解決策の関連性・優先順位を明確化します。

## 実行手順
```yaml
- trigger: "(ソリューションマップ|仮説駆動_ソリューションマップ|HypothesisSolutionMap|Solution Map)"
  priority: high
  steps:
    # Step 1: モード判定（複数ペルソナ課題マップの存在確認）
    - name: "check_problem_maps"
      action: "analyze"
      data: [
        "{{find_files(patterns=['**/P001_*/problem_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/P002_*/problem_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/P003_*/problem_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/P004_*/problem_map_*.yaml'])}}", 
        "{{find_files(patterns=['**/focus_opportunity_hypotheses.yaml'])}}"
      ]
      instructions: |
        課題マップファイル存在確認結果から実行モードを判定してください：
        - 3つ以上のペルソナ課題マップ + opportunity仮説ファイルが存在: "multi_persona"
        - それ以下の場合: "single_persona"
        
        結果をJSONで返してください：
        {"mode": "multi_persona|single_persona", "found_problem_maps": ["P001", "P002", ...], "has_hypotheses": true/false}
      store_as: "execution_mode"

    # === 複数ペルソナモード ===
    - name: "load_problem_data"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/P001_*/problem_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P002_*/problem_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P003_*/problem_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P004_*/problem_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/focus_opportunity_hypotheses.yaml']))}}",
        "{{read_files(find_files(patterns=['**/hackathon_insights_additional_problems.yaml']))}}"
      ]
      instructions: |
        4ペルソナの課題マップとOpportunity仮説から解決策を生成し、以下の構造で返してください：
        {
          "personas": [
            {
              "id": "P001",
              "name": "田中 雅人",
              "role": "AI推進リーダー",
              "solutions_by_priority": {
                "critical": [{"problem_id": "PR101", "solution": "統合UI/UX設計", "linked_hypotheses": ["FH001"], "mvp_priority": 1}],
                "high": [...],
                "medium": [...]
              }
            }
          ],
          "cross_persona_solutions": [
            {"id": "CSL001", "solution": "統合プラットフォーム", "target_problems": ["P001_PR101", "P003_PR102"], "linked_hypotheses": ["FH001"]}
          ]
        }
      store_as: "multi_persona_solutions"

    # === 単一ペルソナモード ===
    - name: "infer_defaults_from_thread"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/problem_map.yaml','**/customer_problem_map.yaml','**/story_map.yaml']))}}"]
      instructions: |
        直近のPR/Activity/Storyの対応から、PR1/PR2/PR3に対する既定ソリューション案を抽出してください。
        出力は display 用テキストで、PRx: 候補… の形式で簡潔に。
      store_as: "auto_solution_hints"

    - name: "ingest_sense_focus"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/1_sense/07_オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/06_リサーチサマリー（全体）/draft_research_summary.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/02_顧客調査/sense_customer_research.md']))}}"
      ]
      instructions: |
        Senseのオポチュニティ/サマリー/顧客調査、およびFocusの定義から、PRごとの解決案候補・効くフェーズ・関連A候補を抽出し、JSON1件で返してください。
        keys:
          pr1_solution_hint, pr1_phase_hint, pr1_actions_hint,
          pr2_solution_hint, pr2_phase_hint, pr2_actions_hint,
          pr3_solution_hint, pr3_phase_hint, pr3_actions_hint,
          persona_and_problem_hint
      store_as: "sense_focus"

    # === 複数ペルソナモード処理 ===
    - name: "display_detected_solutions"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "display"
      content: |
        🎯 検出された解決策構造：
        
        {{#multi_persona_solutions.personas}}
        **{{name}} ({{role}})**:
        {{#solutions_by_priority.critical}}
        - 🔴 {{problem_id}}: {{solution}} (仮説: {{linked_hypotheses | join: ', '}}, MVP優先度: {{mvp_priority}})
        {{/solutions_by_priority.critical}}
        {{#solutions_by_priority.high}}
        - 🟡 {{problem_id}}: {{solution}} (仮説: {{linked_hypotheses | join: ', '}}, MVP優先度: {{mvp_priority}})
        {{/solutions_by_priority.high}}
        {{#solutions_by_priority.medium}}
        - 🔵 {{problem_id}}: {{solution}} (仮説: {{linked_hypotheses | join: ', '}}, MVP優先度: {{mvp_priority}})
        {{/solutions_by_priority.medium}}
        
        {{/multi_persona_solutions.personas}}
        
        **ペルソナ横断ソリューション**:
        {{#multi_persona_solutions.cross_persona_solutions}}
        - {{solution}} (対象: {{target_problems | join: ', '}}, 仮説: {{linked_hypotheses | join: ', '}})
        {{/multi_persona_solutions.cross_persona_solutions}}

    - name: "confirm_multi_persona_generation"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "confirm"
      message: |
        🚀 複数ペルソナモード: 検出された{{multi_persona_solutions.personas | size}}個のペルソナ解決策で全成果物を生成します。
        Opportunity仮説（FH001-FH020）との対応づけも含まれます。
        よろしいですか？

    # === 単一ペルソナモード処理 ===
    - name: "infer_defaults_from_thread"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/problem_map.yaml','**/customer_problem_map.yaml','**/story_map.yaml']))}}"]
      instructions: |
        直近のPR/Activity/Storyの対応から、PR1/PR2/PR3に対する既定ソリューション案を抽出してください。
        出力は display 用テキストで、PRx: 候補… の形式で簡潔に。
      store_as: "auto_solution_hints"
    - name: "prefill_from_yaml"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "display"
      content: |
        📝 単一ペルソナモードで実行します
        
        🔎 候補（自動抽出）
        {{auto_solution_hints}}
        
        🔎 Sense/Focus 由来の候補
        - PR1: {{sense_focus.pr1_solution_hint}}（フェーズ: {{sense_focus.pr1_phase_hint}}、A: {{sense_focus.pr1_actions_hint}}）
        - PR2: {{sense_focus.pr2_solution_hint}}（フェーズ: {{sense_focus.pr2_phase_hint}}、A: {{sense_focus.pr2_actions_hint}}）
        - PR3: {{sense_focus.pr3_solution_hint}}（フェーズ: {{sense_focus.pr3_phase_hint}}、A: {{sense_focus.pr3_actions_hint}}）

    # === 共通処理（複数ペルソナソリューション生成） ===
    - name: "generate_multi_persona_solution_maps"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "loop"
      items: "{{multi_persona_solutions.personas}}"
      steps:
        - name: "create_persona_solution_subfolder"
          action: "execute_shell"
          command: "mkdir -p {{patterns.flow_date}}/3_discovery/03_ソリューションマップ/{{item.id}}_{{item.name | slugify}}"
          
        - name: "generate_persona_solution_yaml"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/03_ソリューションマップ/{{item.id}}_{{item.name | slugify}}/solution_map_{{item.id}}.yaml"
          content: |
            solution_map:
              persona_ref: {{item.id}}
              persona_name: "{{item.name}}"
              persona_role: "{{item.role}}"
              
              solutions:
            {{#item.solutions_by_priority.critical}}
                - id: SL{{@index_plus_1}}01
                  problem_ref: {{problem_id}}
                  summary: "{{solution}}"
                  linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
                  mvp_priority: {{mvp_priority}}
                  priority: "critical"
                  phase: "{{phase | default: 'PH2'}}"
                  implementation_complexity: "{{complexity | default: 'high'}}"
            {{/item.solutions_by_priority.critical}}
            {{#item.solutions_by_priority.high}}
                - id: SL{{@index_plus_1}}02
                  problem_ref: {{problem_id}}
                  summary: "{{solution}}"
                  linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
                  mvp_priority: {{mvp_priority}}
                  priority: "high"
                  phase: "{{phase | default: 'PH2'}}"
                  implementation_complexity: "{{complexity | default: 'medium'}}"
            {{/item.solutions_by_priority.high}}
            {{#item.solutions_by_priority.medium}}
                - id: SL{{@index_plus_1}}03
                  problem_ref: {{problem_id}}
                  summary: "{{solution}}"
                  linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
                  mvp_priority: {{mvp_priority}}
                  priority: "medium"
                  phase: "{{phase | default: 'PH3'}}"
                  implementation_complexity: "{{complexity | default: 'low'}}"
            {{/item.solutions_by_priority.medium}}
              
              numbering_policy:
                - "prefix: SL=Solution, PR=Problem, FH=Focus Hypothesis"
                - "{{item.id}}: {{item.role}}"
                - "Priority: critical(MVP1-3) > high(MVP4-6) > medium(MVP7-9)"

        - name: "generate_persona_solution_mermaid"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/03_ソリューションマップ/{{item.id}}_{{item.name | slugify}}/solution_map_{{item.id}}_mermaid.md"
          content: |
            # {{item.name}}（{{item.role}}）ソリューションマップ
            
            ## Mermaid図
            
            ```mermaid
            flowchart TD
                {{item.id}}[{{item.name}}<br/>{{item.role}}]
                
                {{item.id}} --> CRITICAL[🔴 Critical Solutions]
                {{item.id}} --> HIGH[🟡 High Solutions]
                {{item.id}} --> MEDIUM[🔵 Medium Solutions]
                
            {{#item.solutions_by_priority.critical}}
                CRITICAL --> {{problem_id}}[{{problem_id}}]
                {{problem_id}} --> SL{{@index_plus_1}}01[SL{{@index_plus_1}}01: {{solution}}]
                SL{{@index_plus_1}}01 -.対応仮説.-> {{#linked_hypotheses}}{{.}}_{{../problem_id}}[{{.}}]{{/linked_hypotheses}}
            {{/item.solutions_by_priority.critical}}
            
            {{#item.solutions_by_priority.high}}
                HIGH --> {{problem_id}}[{{problem_id}}]
                {{problem_id}} --> SL{{@index_plus_1}}02[SL{{@index_plus_1}}02: {{solution}}]
                SL{{@index_plus_1}}02 -.対応仮説.-> {{#linked_hypotheses}}{{.}}_{{../problem_id}}[{{.}}]{{/linked_hypotheses}}
            {{/item.solutions_by_priority.high}}
            
            {{#item.solutions_by_priority.medium}}
                MEDIUM --> {{problem_id}}[{{problem_id}}]
                {{problem_id}} --> SL{{@index_plus_1}}03[SL{{@index_plus_1}}03: {{solution}}]
                SL{{@index_plus_1}}03 -.対応仮説.-> {{#linked_hypotheses}}{{.}}_{{../problem_id}}[{{.}}]{{/linked_hypotheses}}
            {{/item.solutions_by_priority.medium}}
                
                %% スタイリング
                classDef personaStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
                classDef priorityStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
                classDef criticalSolution fill:#c8e6c9,stroke:#2e7d32,stroke-width:3px
                classDef highSolution fill:#fff3c4,stroke:#f57c00,stroke-width:2px
                classDef mediumSolution fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
                classDef problemStyle fill:#ffcdd2,stroke:#d32f2f,stroke-width:2px
                classDef hypothesisStyle fill:#e1f5fe,stroke:#0277bd,stroke-width:1px
                
                class {{item.id}} personaStyle
                class CRITICAL,HIGH,MEDIUM priorityStyle
            {{#item.solutions_by_priority.critical}}
                class SL{{@index_plus_1}}01 criticalSolution
                class {{problem_id}} problemStyle
            {{/item.solutions_by_priority.critical}}
            {{#item.solutions_by_priority.high}}
                class SL{{@index_plus_1}}02 highSolution
                class {{problem_id}} problemStyle
            {{/item.solutions_by_priority.high}}
            {{#item.solutions_by_priority.medium}}
                class SL{{@index_plus_1}}03 mediumSolution
                class {{problem_id}} problemStyle
            {{/item.solutions_by_priority.medium}}
            ```
            
            ## MVP優先度・実装計画
            
            {{#item.solutions_by_priority.critical}}
            ### SL{{@index_plus_1}}01: {{solution}}
            **対象課題**: {{problem_id}}  
            **対応仮説**: {{linked_hypotheses | join: ', '}}  
            **MVP優先度**: {{mvp_priority}}  
            **優先度**: 🔴 Critical
            {{/item.solutions_by_priority.critical}}
            
            {{#item.solutions_by_priority.high}}
            ### SL{{@index_plus_1}}02: {{solution}}
            **対象課題**: {{problem_id}}  
            **対応仮説**: {{linked_hypotheses | join: ', '}}  
            **MVP優先度**: {{mvp_priority}}  
            **優先度**: 🟡 High
            {{/item.solutions_by_priority.high}}
            
            {{#item.solutions_by_priority.medium}}
            ### SL{{@index_plus_1}}03: {{solution}}
            **対象課題**: {{problem_id}}  
            **対応仮説**: {{linked_hypotheses | join: ', '}}  
            **MVP優先度**: {{mvp_priority}}  
            **優先度**: 🔵 Medium
            {{/item.solutions_by_priority.medium}}
            
            ---
            **作成日**: {{today}}  
            **対応仮説**: {{item.solutions_by_priority.critical.0.linked_hypotheses | join: ', '}}{{item.solutions_by_priority.high.0.linked_hypotheses | join: ', '}}{{item.solutions_by_priority.medium.0.linked_hypotheses | join: ', '}}  
            **MVP計画**: Critical→High→Mediumの段階的実装
    
    # 統合ソリューションマップ生成（複数ペルソナ）
    - name: "generate_integrated_solution_map"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/03_ソリューションマップ/integrated_solution_map.yaml"
      content: |
        integrated_solution_map:
          meta:
            created_date: "{{today}}"
            project: "Palma MVP0"
            phase: "Discovery - ソリューションマップ"
            description: "4ペルソナ統合ソリューションマップ・Opportunity仮説対応"
            version: "v1.0"
          
          personas_solutions:
        {{#multi_persona_solutions.personas}}
            {{id}}:
              name: "{{name}}"
              role: "{{role}}"
              critical_solutions:
        {{#solutions_by_priority.critical}}
                - {{problem_id}}: "{{solution}}"
        {{/solutions_by_priority.critical}}
              high_solutions:
        {{#solutions_by_priority.high}}
                - {{problem_id}}: "{{solution}}"
        {{/solutions_by_priority.high}}
              medium_solutions:
        {{#solutions_by_priority.medium}}
                - {{problem_id}}: "{{solution}}"
        {{/solutions_by_priority.medium}}
        {{/multi_persona_solutions.personas}}
          
          cross_persona_solutions:
        {{#multi_persona_solutions.cross_persona_solutions}}
            - id: {{id}}
              solution: "{{solution}}"
              target_problems: [{{target_problems | join: ', '}}]
              linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
              mvp_priority: {{mvp_priority | default: 1}}
              implementation_scope: "全ペルソナ対応"
        {{/multi_persona_solutions.cross_persona_solutions}}
          
          mvp_roadmap:
            phase_1_critical:
              solutions: "統合UI/UX、ROI可視化、人間-AI協働、セルフサービス型"
              target_hypotheses: [FH001, FH002, FH003, FH004, FH018]
              target_personas: [P001, P002, P003, P004]
              implementation_period: "1-3ヶ月"
              
            phase_2_high:
              solutions: "ビジネス職種特化、自動コンテキスト収集、カスタマイズ機能"
              target_hypotheses: [FH005, FH006, FH007, FH015, FH017, FH020]
              target_personas: [P001, P003, P004]
              implementation_period: "4-6ヶ月"
              
            phase_3_medium:
              solutions: "段階的導入、ナレッジ共有、既存業務適用"
              target_hypotheses: [FH008, FH009, FH013, FH014, FH016, FH019]
              target_personas: [P001, P003, P004]
              implementation_period: "7-12ヶ月"

    - name: "generate_integrated_solution_mermaid"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/03_ソリューションマップ/integrated_solution_map_mermaid.md"
      content: |
        # 統合ソリューションマップ - 4ペルソナ・Opportunity仮説対応
        
        ## MVP実装ロードマップ
        
        ```mermaid
        gantt
            title Palma MVP0 ソリューション実装ロードマップ
            dateFormat X
            axisFormat %s
            
            section Phase 1 Critical
            統合UI/UX設計           :crit, ui_ux, 0, 3
            ROI可視化ダッシュボード  :crit, roi_dash, 0, 3
            人間-AI協働機能         :crit, human_ai, 1, 3
            セルフサービス型設計    :crit, self_service, 2, 3
            
            section Phase 2 High
            ビジネス職種特化UI      :high, biz_ui, 3, 6
            自動コンテキスト収集    :high, auto_context, 3, 6
            カスタマイズ機能        :high, customize, 4, 6
            実用化支援システム      :high, practical, 4, 6
            
            section Phase 3 Medium
            段階的導入プロセス      :medium, gradual, 6, 12
            ナレッジ共有機能        :medium, knowledge, 7, 12
            既存業務適用機能        :medium, existing, 8, 12
        ```
        
        ## ペルソナ別ソリューション優先度
        
        ```mermaid
        graph TD
            subgraph "🔴 Phase 1: Critical Solutions (1-3ヶ月)"
        {{#multi_persona_solutions.personas}}
        {{#solutions_by_priority.critical}}
                {{../id}}_{{problem_id}}[{{../../name}}: {{solution}}]
        {{/solutions_by_priority.critical}}
        {{/multi_persona_solutions.personas}}
            end
            
            subgraph "🟡 Phase 2: High Solutions (4-6ヶ月)"
        {{#multi_persona_solutions.personas}}
        {{#solutions_by_priority.high}}
                {{../id}}_{{problem_id}}[{{../../name}}: {{solution}}]
        {{/solutions_by_priority.high}}
        {{/multi_persona_solutions.personas}}
            end
            
            subgraph "🔵 Phase 3: Medium Solutions (7-12ヶ月)"
        {{#multi_persona_solutions.personas}}
        {{#solutions_by_priority.medium}}
                {{../id}}_{{problem_id}}[{{../../name}}: {{solution}}]
        {{/solutions_by_priority.medium}}
        {{/multi_persona_solutions.personas}}
            end
            
            %% ペルソナ横断ソリューション
        {{#multi_persona_solutions.cross_persona_solutions}}
            {{id}}[ペルソナ横断: {{solution}}]
        {{/multi_persona_solutions.cross_persona_solutions}}
            
            classDef criticalStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:3px
            classDef highStyle fill:#fff3c4,stroke:#f57c00,stroke-width:2px
            classDef mediumStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
            classDef crossStyle fill:#f1f8e9,stroke:#33691e,stroke-width:3px
        ```

    # === 単一ペルソナモード処理 ===
    - name: "show_map_examples"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "display"
      content: |
        📝 単一ペルソナモードで実行します
        
        🗺️ PR基準の設計例（A→PR→SL）
        - A101 → PR1 → SL1
        - A201 → PR2 → SL2
        - A301 → PR3 → SL3
    
    - name: "collect_solution_map"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "ask_questions_with_template"
      template: |
        === ソリューションマップ入力テンプレート（PR基準） ===
        1. ペルソナと課題群要約
        既定: P001 / PR1・PR2・PR3（problem_map.yaml）
        Sense/Focus候補: {{sense_focus.persona_and_problem_hint}}
        → 【あなたの回答】：
        
        2. PR1 向けの解決案（効くフェーズ/関連A候補 任意）
        既定候補: PR1=朝の判断迷い / A候補: A101
        Sense/Focus候補: 解決案={{sense_focus.pr1_solution_hint}} / フェーズ={{sense_focus.pr1_phase_hint}} / A={{sense_focus.pr1_actions_hint}}
        → 【解決案】：
        → 【効くフェーズ（任意: 利用前/中/後）】：
        → 【関連A（任意: AIDカンマ区切り）】：
        
        3. PR2 向けの解決案（効くフェーズ/関連A候補 任意）
        既定候補: PR2=昼の処理待ち / A候補: A201
        Sense/Focus候補: 解決案={{sense_focus.pr2_solution_hint}} / フェーズ={{sense_focus.pr2_phase_hint}} / A={{sense_focus.pr2_actions_hint}}
        → 【解決案】：
        → 【効くフェーズ（任意）】：
        → 【関連A（任意）】：
        
        4. PR3 向けの解決案（効くフェーズ/関連A候補 任意）
        既定候補: PR3=夜の達成感不足 / A候補: A301
        Sense/Focus候補: 解決案={{sense_focus.pr3_solution_hint}} / フェーズ={{sense_focus.pr3_phase_hint}} / A={{sense_focus.pr3_actions_hint}}
        → 【解決案】：
        → 【効くフェーズ（任意）】：
        → 【関連A（任意）】：
        =====================================
    
    - name: "wait_inputs"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "wait_for_all_answers"
    
    - name: "spot_mvp_candidates"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "interactive_dialog"
      message: |
        いいですね！ 3つの解決案の中から、MVPでまず検証すべき順序を1→3で指定してください（例: SL2, SL1, SL3）。
    
    # ここから：自動提案 → 修正収集 → 確定生成（単一ペルソナモード）
    - name: "auto_propose_solution_map_yaml"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/03_ソリューションマップ/solution_map_proposed.yaml"
      content: |
        solution_map:
          persona_ref: P001
          problems: [PR1, PR2, PR3]
          phases:
            - id: PH1
              name: 利用前
              solutions:
                - id: SL1
                  label: {{sl1_label | default:pr1_solution}}
                  links: [PR1, {{sl1_actions | default:"A101"}}]
            - id: PH2
              name: 利用中
              solutions:
                - id: SL2
                  label: {{sl2_label | default:pr2_solution}}
                  links: [PR2, {{sl2_actions | default:"A201"}}]
            - id: PH3
              name: 利用後
              solutions:
                - id: SL3
                  label: {{sl3_label | default:pr3_solution}}
                  links: [PR3, {{sl3_actions | default:"A301"}}]
          numbering_policy:
            - prefix: SL=Solution, PH=Phase
    
    - name: "auto_propose_solution_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/03_ソリューションマップ/solution_map_proposed_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          {{sl1_actions | default:"A101"}} --> PR1:::problem
          {{sl2_actions | default:"A201"}} --> PR2:::problem
          {{sl3_actions | default:"A301"}} --> PR3:::problem
          PR1:::problem --> SL1[SL1 {{sl1_label | default:pr1_solution}}]:::solution
          PR2:::problem --> SL2[SL2 {{sl2_label | default:pr2_solution}}]:::solution
          PR3:::problem --> SL3[SL3 {{sl3_label | default:pr3_solution}}]:::solution
          classDef problem fill:#ffebee
          classDef solution fill:#e8f5e8
          classDef action fill:#fff3e0
        ```
    
    - name: "display_proposal"
      action: "display"
      content: |
        ✍️ 自動提案（PR1/PR2/PR3）を `solution_map_proposed.*` に出力しました。必要な修正を入力してください（空欄は採用）。
    
    - name: "collect_solution_corrections"
      action: "ask_questions_with_template"
      template: |
        === 追加/修正テンプレート（空欄は採用） ===
        SL1(PR1) ラベル/リンク修正
        → 【ラベル】：
        → 【リンク（例: PR1,A101）】：
        
        SL2(PR2) ラベル/リンク修正
        → 【ラベル】：
        → 【リンク】：
        
        SL3(PR3) ラベル/リンク修正
        → 【ラベル】：
        → 【リンク】：
        =====================================
    
    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正内容で最終成果物（solution_map.md / solution_map.yaml / solution_map_mermaid.md / ソリューション仮説文）を生成します。よろしいですか？"
    
    - name: "generate_solution_map"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/03_ソリューションマップ/solution_map_todo.md"
      content: |
        # アプリ ソリューションマップ（仮説）
        
        ## 対象と課題（PR基準）
        {{persona_and_problem}}
        
        ## PR別の解決案
        - PR1: {{sl1_label_final | default:sl1_label | default:pr1_solution}}
        - PR2: {{sl2_label_final | default:sl2_label | default:pr2_solution}}
        - PR3: {{sl3_label_final | default:sl3_label | default:pr3_solution}}
        
        ## MVP候補（検証優先の順）
        1. {{mvp_candidate_1}}
        2. {{mvp_candidate_2}}
        3. {{mvp_candidate_3}}
        
        ## ソリューション仮説（Statement）
        > {{solution}} することで、より多くのお客様が {{behavior_change}} できるようになると信じています。
    
    - name: "export_solution_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/03_ソリューションマップ/solution_map.yaml"
      content: |
        solution_map:
          persona_ref: P001
          problems: [PR1, PR2, PR3]
          phases:
            - id: PH1
              name: 利用前
              solutions:
                - id: SL1
                  label: {{sl1_label_final | default:sl1_label | default:pr1_solution}}
                  links: [{{sl1_links_final | default:"PR1,A101"}}]
            - id: PH2
              name: 利用中
              solutions:
                - id: SL2
                  label: {{sl2_label_final | default:sl2_label | default:pr2_solution}}
                  links: [{{sl2_links_final | default:"PR2,A201"}}]
            - id: PH3
              name: 利用後
              solutions:
                - id: SL3
                  label: {{sl3_label_final | default:sl3_label | default:pr3_solution}}
                  links: [{{sl3_links_final | default:"PR3,A301"}}]
          numbering_policy:
            - prefix: SL=Solution, PH=Phase
    
    - name: "export_solution_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/03_ソリューションマップ/solution_map_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          {{sl1_actions | default:"A101"}} --> PR1:::problem
          {{sl2_actions | default:"A201"}} --> PR2:::problem
          {{sl3_actions | default:"A301"}} --> PR3:::problem
          PR1:::problem --> SL1[SL1 {{sl1_label_final | default:sl1_label | default:pr1_solution}}]:::solution
          PR2:::problem --> SL2[SL2 {{sl2_label_final | default:sl2_label | default:pr2_solution}}]:::solution
          PR3:::problem --> SL3[SL3 {{sl3_label_final | default:sl3_label | default:pr3_solution}}]:::solution
          classDef problem fill:#ffebee
          classDef solution fill:#e8f5e8
          classDef action fill:#fff3e0
        ```
    
    # === 完了通知（両モード） ===
    - name: "notify_completion"
      action: "display"
      content: |
        {{#if (eq execution_mode.mode 'multi_persona')}}
        ✅ 複数ペルソナモード完了：
        
        **ペルソナ別ソリューションマップ**:
        {{#multi_persona_solutions.personas}}
        - 📁 {{id}}_{{name | slugify}}/
          - solution_map_{{id}}.yaml
          - solution_map_{{id}}_mermaid.md
        {{/multi_persona_solutions.personas}}
        
        **統合ファイル**:
        - integrated_solution_map.yaml
        - integrated_solution_map_mermaid.md
        
        🚀 **Opportunity仮説対応完了**: FH001-FH020との完全対応
        🔴 Critical Solutions: {{multi_persona_solutions.personas | map: 'solutions_by_priority.critical' | flatten | size}}件
        🟡 High Solutions: {{multi_persona_solutions.personas | map: 'solutions_by_priority.high' | flatten | size}}件
        
        次のステップ: MVP設計・プロトタイプ開発計画
        {{else}}
        ✅ 単一ペルソナモード完了：
        - （提案）03_ソリューションマップ/solution_map_proposed.yaml / solution_map_proposed_mermaid.md
        - （確定）03_ソリューションマップ/solution_map_todo.md / solution_map.yaml / solution_map_mermaid.md / ソリューション仮説文
        
        📝 **質問ベースモード完了**: 仮説は未設定
        次のステップ: 
        1. 実際のユーザーテストでソリューション検証
        2. MVP機能の優先順位決定
        3. プロトタイプ設計
        {{/if}}
```

## 生成される成果物

### 複数ペルソナモード
#### ペルソナ別ファイル（各ペルソナ×2ファイル）
- `{{persona_id}}_{{name}}/solution_map_{{persona_id}}.yaml` - ペルソナ別ソリューションマップYAML
- `{{persona_id}}_{{name}}/solution_map_{{persona_id}}_mermaid.md` - ペルソナ別ソリューションMermaid図

#### 統合ファイル
- `integrated_solution_map.yaml` - 4ペルソナ統合ソリューションマップ
- `integrated_solution_map_mermaid.md` - 統合ソリューション可視化・MVP実装ロードマップ

### 単一ペルソナモード
- `solution_map_proposed.yaml` / `solution_map_proposed_mermaid.md`
- `solution_map_todo.md` / `solution_map.yaml` / `solution_map_mermaid.md`

### 特徴

#### 🚀 複数ペルソナモード（4ペルソナの課題マップがある場合）
- **自動ソリューション生成**: 4ペルソナの課題マップから解決策を自動生成
- **Opportunity仮説対応**: FH001-FH020との完全対応づけ
- **MVP優先度判定**: Critical(1-3) > High(4-6) > Medium(7-9)の自動優先順位
- **ペルソナ横断ソリューション**: 複数ペルソナに効果的な共通解決策
- **実装ロードマップ**: Ganttチャートによる段階的実装計画

#### 📝 単一ペルソナモード（課題マップが少ない場合）
- **質問収集**: 従来の質問形式でソリューション情報を収集
- **Sense/Focus連携**: 既存調査結果からの解決策候補自動提示
- **MVP候補選定**: 手動による優先順位決定

#### 共通機能
- **モード自動判定**: 課題マップファイル存在確認で適切なモードを自動選択
- **仮説統合**: Opportunity仮説との対応づけによるMVP計画策定
- **優先度管理**: 仮説の重要度に基づくソリューション優先順位付け

## 次のコマンド
→ `MVP設計` で最優先ソリューションの詳細設計  
→ `プロトタイプ計画` で検証用プロトタイプ開発計画策定
