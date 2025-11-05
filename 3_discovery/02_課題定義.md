# 02 課題定義（複数ペルソナ・仮説統合版）

## 実行モード
### 🚀 **複数ペルソナモード**（4ペルソナのexperience_mapがある場合）
4ペルソナ（田中・佐藤・山田・高橋）のexperience_mapとOpportunity仮説（FH001-FH014）から課題を自動抽出し、ペルソナ別・フェーズ別の課題マップを生成

### 📝 **単一ペルソナモード**（1ペルソナのみの場合）
従来の質問形式で単一ペルソナの課題を収集し、フェーズ別に構造化

## 複数ペルソナモード - 自動検出対象
- `P001_田中_AI推進リーダー/experience_map_primary_tanaka.yaml`
- `P002_佐藤_経営層/experience_map_secondary_sato.yaml`
- `P003_山田_現場PdM/experience_map_tertiary_yamada.yaml`
- `P004_高橋_コンテンツ運用/experience_map_content_manager.yaml`
- `focus_opportunity_hypotheses.yaml` - FH001-FH014仮説マッピング

## 単一ペルソナモード - 質問項目
- フェーズごとの困りごと（最大3つ、各1行）
  - PH1の困りごと（必須）
  - PH2の困りごと（任意）
  - PH3の困りごと（任意）
- 各困りごとの「なぜ」×3（任意：空欄なら自動補完）
- 各困りごとの影響（時間/頻度/波及）（任意：空欄なら自動補完）
- 現在の対処（任意）

## 目的
複数ペルソナの課題を統合的に分析し、Opportunity仮説と対応づけた課題マップを生成。ペルソナ間の課題の関連性・優先順位を明確化します。

## 実行手順
```yaml
- trigger: "(課題定義|仮説駆動_課題定義|HypothesisProblem|Problem Definition)"
  priority: high
  steps:
    # Step 1: モード判定（複数ペルソナファイルの存在確認）
    - name: "check_persona_files"
      action: "analyze"
      data: [
        "{{find_files(patterns=['**/P001_*/*.yaml'])}}", 
        "{{find_files(patterns=['**/P002_*/*.yaml'])}}", 
        "{{find_files(patterns=['**/P003_*/*.yaml'])}}", 
        "{{find_files(patterns=['**/P004_*/*.yaml'])}}", 
        "{{find_files(patterns=['**/focus_opportunity_hypotheses.yaml'])}}"
      ]
      instructions: |
        ペルソナファイル存在確認結果から実行モードを判定してください：
        - 3つ以上のペルソナファイル + opportunity仮説ファイルが存在: "multi_persona"
        - それ以下の場合: "single_persona"
        
        結果をJSONで返してください：
        {"mode": "multi_persona|single_persona", "found_personas": ["P001", "P002", ...], "has_hypotheses": true/false}
      store_as: "execution_mode"

    # === 複数ペルソナモード ===
    - name: "load_persona_data"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/P001_*/experience_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P002_*/experience_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P003_*/experience_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/P004_*/experience_map_*.yaml']))}}",
        "{{read_files(find_files(patterns=['**/focus_opportunity_hypotheses.yaml']))}}"
      ]
      instructions: |
        4ペルソナのexperience_mapとOpportunity仮説から課題を抽出し、以下の構造で返してください：
        {
          "personas": [
            {
              "id": "P001",
              "name": "田中 雅人",
              "role": "AI推進リーダー",
              "problems_by_phase": {
                "PH1": [{"id": "PR101", "summary": "課題要約", "linked_hypotheses": ["FH001"], "impact": "影響度", "causes": ["原因1"]}],
                "PH2": [...],
                "PH3": [...]
              }
            }
          ],
          "cross_persona_problems": [
            {"id": "CPR001", "summary": "ペルソナ横断課題", "affected_personas": ["P001", "P003"], "linked_hypotheses": ["FH001", "FH003"]}
          ]
        }
      store_as: "multi_persona_problems"

    # === 単一ペルソナモード ===
    - name: "infer_defaults_from_thread"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['Flow/**/*.yaml','Flow/**/*.md','**/problem_map.yaml','**/customer_problem_map.yaml']))}}"]
      instructions: |
        スレッドと直近生成物から、PH1/PH2/PH3の候補やPR要約の下書きを抽出して提示してください。
        出力は display用テキストとし、空欄は含めずに簡潔に要点のみ列挙。
      store_as: "auto_phase_hints"

    - name: "ingest_sense_focus"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/1_sense/02_顧客調査/sense_customer_research.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/05_インタビュー分析（個別）/draft_interview_analysis_*.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/06_リサーチサマリー（全体）/draft_research_summary.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/07_オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}"
      ]
      instructions: |
        顧客調査・インタビュー分析・全体サマリー・オポチュニティ・プロダクト定義から、フェーズ別課題の候補を抽出し、JSON1件で返してください。
        keys:
          ph1_summary_hint, ph1_whys_hint, ph1_impact_hint,
          ph2_summary_hint, ph2_whys_hint, ph2_impact_hint,
          ph3_summary_hint, ph3_whys_hint, ph3_impact_hint,
          current_solution_hint
      store_as: "sense_focus"

    # === 複数ペルソナモード処理 ===
    - name: "display_detected_problems"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "display"
      content: |
        🎯 検出された課題構造：
        
        {{#multi_persona_problems.personas}}
        **{{name}} ({{role}})**:
        {{#problems_by_phase.PH1}}
        - PH1: {{summary}} (仮説: {{linked_hypotheses | join: ', '}})
        {{/problems_by_phase.PH1}}
        {{#problems_by_phase.PH2}}
        - PH2: {{summary}} (仮説: {{linked_hypotheses | join: ', '}})
        {{/problems_by_phase.PH2}}
        {{#problems_by_phase.PH3}}
        - PH3: {{summary}} (仮説: {{linked_hypotheses | join: ', '}})
        {{/problems_by_phase.PH3}}
        
        {{/multi_persona_problems.personas}}
        
        **ペルソナ横断課題**:
        {{#multi_persona_problems.cross_persona_problems}}
        - {{summary}} (対象: {{affected_personas | join: ', '}}, 仮説: {{linked_hypotheses | join: ', '}})
        {{/multi_persona_problems.cross_persona_problems}}

    - name: "confirm_multi_persona_generation"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "confirm"
      message: |
        🚀 複数ペルソナモード: 検出された{{multi_persona_problems.personas | size}}個のペルソナ課題で全成果物を生成します。
        Opportunity仮説（FH001-FH014）との対応づけも含まれます。
        よろしいですか？
    # === 単一ペルソナモード処理 ===
    - name: "prefill_from_yaml"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "display"
      content: |
        📝 単一ペルソナモードで実行します
        
        🔎 候補（自動抽出）
        {{auto_phase_hints}}
        
        🔎 Sense/Focus 由来の候補
        - PH1: {{sense_focus.ph1_summary_hint}}
          - Why候補: {{sense_focus.ph1_whys_hint}}
          - Impact候補: {{sense_focus.ph1_impact_hint}}
        - PH2: {{sense_focus.ph2_summary_hint}}
          - Why候補: {{sense_focus.ph2_whys_hint}}
          - Impact候補: {{sense_focus.ph2_impact_hint}}
        - PH3: {{sense_focus.ph3_summary_hint}}
          - Why候補: {{sense_focus.ph3_whys_hint}}
          - Impact候補: {{sense_focus.ph3_impact_hint}}
        - 現在の対処候補: {{sense_focus.current_solution_hint}}
    
    - name: "collect_phase_problems"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "ask_questions_with_template"
      template: |
        === フェーズ別課題入力テンプレ ===
        1. PH1の困りごと（1行）
        （候補: {{sense_focus.ph1_summary_hint}}）
        →
        1-why: なぜ① / なぜ② / なぜ③（任意）
        （候補: {{sense_focus.ph1_whys_hint}}）
        1-impact: 影響（任意）
        （候補: {{sense_focus.ph1_impact_hint}}）
        
        2. PH2の困りごと（1行・任意）
        （候補: {{sense_focus.ph2_summary_hint}}）
        →
        2-why: なぜ① / なぜ② / なぜ③（任意）
        （候補: {{sense_focus.ph2_whys_hint}}）
        2-impact: 影響（任意）
        （候補: {{sense_focus.ph2_impact_hint}}）
        
        3. PH3の困りごと（1行・任意）
        （候補: {{sense_focus.ph3_summary_hint}}）
        →
        3-why: なぜ① / なぜ② / なぜ③（任意）
        （候補: {{sense_focus.ph3_whys_hint}}）
        3-impact: 影響（任意）
        （候補: {{sense_focus.ph3_impact_hint}}）
        
        4. 現在の対処（任意）
        （候補: {{sense_focus.current_solution_hint}}）
        →
        =====================================
    
    - name: "wait_inputs"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "wait_for_all_answers"

    # === 共通処理（複数ペルソナ成果物生成） ===
    - name: "generate_multi_persona_problem_maps"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "loop"
      items: "{{multi_persona_problems.personas}}"
      steps:
        - name: "create_persona_problem_subfolder"
          action: "execute_shell"
          command: "mkdir -p {{patterns.flow_date}}/3_discovery/02_課題定義/{{item.id}}_{{item.name | slugify}}"
          
        - name: "generate_persona_problem_yaml"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/02_課題定義/{{item.id}}_{{item.name | slugify}}/problem_map_{{item.id}}.yaml"
          content: |
            problem_map:
              persona_ref: {{item.id}}
              persona_name: "{{item.name}}"
              persona_role: "{{item.role}}"
              
              problems:
            {{#item.problems_by_phase.PH1}}
                - id: {{id}}
                  summary: "{{summary}}"
                  causes: [{{causes | join: '", "'}}]
                  impact: "{{impact}}"
                  linked_actions: [{{linked_actions | join: ', '}}]
                  linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
                  occurs_in_phases: [PH1]
                  priority: "{{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}critical{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}high{{else}}medium{{/if}}{{/if}}"
            {{/item.problems_by_phase.PH1}}
            {{#item.problems_by_phase.PH2}}
                - id: {{id}}
                  summary: "{{summary}}"
                  causes: [{{causes | join: '", "'}}]
                  impact: "{{impact}}"
                  linked_actions: [{{linked_actions | join: ', '}}]
                  linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
                  occurs_in_phases: [PH2]
                  priority: "{{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}critical{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}high{{else}}medium{{/if}}{{/if}}"
            {{/item.problems_by_phase.PH2}}
            {{#item.problems_by_phase.PH3}}
                - id: {{id}}
                  summary: "{{summary}}"
                  causes: [{{causes | join: '", "'}}]
                  impact: "{{impact}}"
                  linked_actions: [{{linked_actions | join: ', '}}]
                  linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
                  occurs_in_phases: [PH3]
                  priority: "{{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}critical{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}high{{else}}medium{{/if}}{{/if}}"
            {{/item.problems_by_phase.PH3}}
              
              numbering_policy:
                - "prefix: PR=Problem, A=Action, FH=Focus Hypothesis"
                - "{{item.id}}: {{item.role}}"
                - "Priority: critical(FH001-004) > high(FH005-007) > medium(others)"

        - name: "generate_persona_problem_mermaid"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/02_課題定義/{{item.id}}_{{item.name | slugify}}/problem_map_{{item.id}}_mermaid.md"
          content: |
            # {{item.name}}（{{item.role}}）課題マップ
            
            ## Mermaid図
            
            ```mermaid
            flowchart TD
                {{item.id}}[{{item.name}}<br/>{{item.role}}]
                
            {{#item.problems_by_phase.PH1}}
                {{item.id}} --> PH1_{{../item.id}}[PH1: 戦略・計画フェーズ]
                PH1_{{../item.id}} --> {{id}}[{{id}}: {{summary}}]
                {{id}} -.対応仮説.-> {{#linked_hypotheses}}{{.}}_{{../id}}[{{.}}]{{/linked_hypotheses}}
            {{/item.problems_by_phase.PH1}}
            
            {{#item.problems_by_phase.PH2}}
                {{item.id}} --> PH2_{{../item.id}}[PH2: 実行・導入フェーズ]
                PH2_{{../item.id}} --> {{id}}[{{id}}: {{summary}}]
                {{id}} -.対応仮説.-> {{#linked_hypotheses}}{{.}}_{{../id}}[{{.}}]{{/linked_hypotheses}}
            {{/item.problems_by_phase.PH2}}
            
            {{#item.problems_by_phase.PH3}}
                {{item.id}} --> PH3_{{../item.id}}[PH3: 評価・改善フェーズ]
                PH3_{{../item.id}} --> {{id}}[{{id}}: {{summary}}]
                {{id}} -.対応仮説.-> {{#linked_hypotheses}}{{.}}_{{../id}}[{{.}}]{{/linked_hypotheses}}
            {{/item.problems_by_phase.PH3}}
                
                %% スタイリング
                classDef personaStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
                classDef phaseStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
                classDef criticalProblem fill:#ffcdd2,stroke:#d32f2f,stroke-width:3px
                classDef highProblem fill:#c8e6c9,stroke:#388e3c,stroke-width:2px
                classDef mediumProblem fill:#fff3c4,stroke:#f57c00,stroke-width:2px
                classDef hypothesisStyle fill:#e1f5fe,stroke:#0277bd,stroke-width:1px
                
                class {{item.id}} personaStyle
                class PH1_{{item.id}},PH2_{{item.id}},PH3_{{item.id}} phaseStyle
            {{#item.problems_by_phase.PH1}}
                class {{id}} {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}criticalProblem{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}highProblem{{else}}mediumProblem{{/if}}{{/if}}
            {{/item.problems_by_phase.PH1}}
            {{#item.problems_by_phase.PH2}}
                class {{id}} {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}criticalProblem{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}highProblem{{else}}mediumProblem{{/if}}{{/if}}
            {{/item.problems_by_phase.PH2}}
            {{#item.problems_by_phase.PH3}}
                class {{id}} {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}criticalProblem{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}highProblem{{else}}mediumProblem{{/if}}{{/if}}
            {{/item.problems_by_phase.PH3}}
            ```
            
            ## Opportunity仮説との対応
            
            {{#item.problems_by_phase.PH1}}
            ### {{id}}: {{summary}}
            **対応仮説**: {{linked_hypotheses | join: ', '}}
            **優先度**: {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}🔴 Critical{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}🟡 High{{else}}🔵 Medium{{/if}}{{/if}}
            {{/item.problems_by_phase.PH1}}
            
            {{#item.problems_by_phase.PH2}}
            ### {{id}}: {{summary}}
            **対応仮説**: {{linked_hypotheses | join: ', '}}
            **優先度**: {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}🔴 Critical{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}🟡 High{{else}}🔵 Medium{{/if}}{{/if}}
            {{/item.problems_by_phase.PH2}}
            
            {{#item.problems_by_phase.PH3}}
            ### {{id}}: {{summary}}
            **対応仮説**: {{linked_hypotheses | join: ', '}}
            **優先度**: {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}🔴 Critical{{else}}{{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}🟡 High{{else}}🔵 Medium{{/if}}{{/if}}
            {{/item.problems_by_phase.PH3}}
            
            ---
            **作成日**: {{today}}  
            **対応仮説**: {{item.problems_by_phase.PH1.0.linked_hypotheses | join: ', '}}{{item.problems_by_phase.PH2.0.linked_hypotheses | join: ', '}}{{item.problems_by_phase.PH3.0.linked_hypotheses | join: ', '}}  
            **検証計画**: Discovery フェーズ段階的検証
    
    # 統合課題マップ生成（複数ペルソナ）
    - name: "generate_integrated_problem_map"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/02_課題定義/integrated_problem_map.yaml"
      content: |
        integrated_problem_map:
          meta:
            created_date: "{{today}}"
            project: "Palma MVP0"
            phase: "Discovery - 課題定義"
            description: "4ペルソナ統合課題マップ・Opportunity仮説対応"
            version: "v1.0"
          
          personas_problems:
        {{#multi_persona_problems.personas}}
            {{id}}:
              name: "{{name}}"
              role: "{{role}}"
              problems:
        {{#problems_by_phase.PH1}}
                - {{id}}: "{{summary}}"
        {{/problems_by_phase.PH1}}
        {{#problems_by_phase.PH2}}
                - {{id}}: "{{summary}}"
        {{/problems_by_phase.PH2}}
        {{#problems_by_phase.PH3}}
                - {{id}}: "{{summary}}"
        {{/problems_by_phase.PH3}}
        {{/multi_persona_problems.personas}}
          
          cross_persona_problems:
        {{#multi_persona_problems.cross_persona_problems}}
            - id: {{id}}
              summary: "{{summary}}"
              affected_personas: [{{affected_personas | join: ', '}}]
              linked_hypotheses: [{{linked_hypotheses | join: ', '}}]
              priority: "{{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}critical{{else}}high{{/if}}"
        {{/multi_persona_problems.cross_persona_problems}}
          
          hypothesis_problem_mapping:
        {{#multi_persona_problems.personas}}
        {{#problems_by_phase.PH1}}
        {{#linked_hypotheses}}
            {{.}}:
              problems: [{{../id}}]
              persona: {{../../id}}
              phase: PH1
        {{/linked_hypotheses}}
        {{/problems_by_phase.PH1}}
        {{#problems_by_phase.PH2}}
        {{#linked_hypotheses}}
            {{.}}:
              problems: [{{../id}}]
              persona: {{../../id}}
              phase: PH2
        {{/linked_hypotheses}}
        {{/problems_by_phase.PH2}}
        {{#problems_by_phase.PH3}}
        {{#linked_hypotheses}}
            {{.}}:
              problems: [{{../id}}]
              persona: {{../../id}}
              phase: PH3
        {{/linked_hypotheses}}
        {{/problems_by_phase.PH3}}
        {{/multi_persona_problems.personas}}

    - name: "generate_integrated_problem_mermaid"
      condition: "{{execution_mode.mode == 'multi_persona'}}"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/02_課題定義/integrated_problem_map_mermaid.md"
      content: |
        # 統合課題マップ - 4ペルソナ・Opportunity仮説対応
        
        ## 全体課題構造
        
        ```mermaid
        graph TD
            subgraph "🔴 Critical Problems (FH001-004)"
        {{#multi_persona_problems.personas}}
        {{#problems_by_phase.PH1}}
        {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/if}}
        {{/problems_by_phase.PH1}}
        {{#problems_by_phase.PH2}}
        {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/if}}
        {{/problems_by_phase.PH2}}
        {{#problems_by_phase.PH3}}
        {{#if (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/if}}
        {{/problems_by_phase.PH3}}
        {{/multi_persona_problems.personas}}
            end
            
            subgraph "🟡 High Problems (FH005-007)"
        {{#multi_persona_problems.personas}}
        {{#problems_by_phase.PH1}}
        {{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/if}}
        {{/problems_by_phase.PH1}}
        {{#problems_by_phase.PH2}}
        {{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/if}}
        {{/problems_by_phase.PH2}}
        {{#problems_by_phase.PH3}}
        {{#if (any linked_hypotheses 'FH005' 'FH006' 'FH007')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/if}}
        {{/problems_by_phase.PH3}}
        {{/multi_persona_problems.personas}}
            end
            
            subgraph "🔵 Medium Problems (Others)"
        {{#multi_persona_problems.personas}}
        {{#problems_by_phase.PH1}}
        {{#unless (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004' 'FH005' 'FH006' 'FH007')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/unless}}
        {{/problems_by_phase.PH1}}
        {{#problems_by_phase.PH2}}
        {{#unless (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004' 'FH005' 'FH006' 'FH007')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/unless}}
        {{/problems_by_phase.PH2}}
        {{#problems_by_phase.PH3}}
        {{#unless (any linked_hypotheses 'FH001' 'FH002' 'FH003' 'FH004' 'FH005' 'FH006' 'FH007')}}
                {{id}}[{{../../../name}}: {{summary}}]
        {{/unless}}
        {{/problems_by_phase.PH3}}
        {{/multi_persona_problems.personas}}
            end
            
            %% ペルソナ横断課題
        {{#multi_persona_problems.cross_persona_problems}}
            {{id}}[ペルソナ横断: {{summary}}]
        {{/multi_persona_problems.cross_persona_problems}}
            
            classDef criticalStyle fill:#ffcdd2,stroke:#d32f2f,stroke-width:3px
            classDef highStyle fill:#c8e6c9,stroke:#388e3c,stroke-width:2px
            classDef mediumStyle fill:#fff3c4,stroke:#f57c00,stroke-width:2px
            classDef crossStyle fill:#e1f5fe,stroke:#0277bd,stroke-width:3px
        ```

    # === 単一ペルソナモード処理 ===
    - name: "auto_propose_customer_problem_map_yaml"
      condition: "{{execution_mode.mode == 'single_persona'}}"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/02_課題定義/customer_problem_map_proposed.yaml"
      content: |
        customer_problem_map:
          persona_ref: P001
          phases:
            - id: PH1
              actions:
                - id: A101
                  thoughts: [{{auto_ph1_thoughts}}]
                  emotions: [{{auto_ph1_emotions}}]
                  issues: [PR1]
            - id: PH2
              actions:
                - id: A201
                  thoughts: [{{auto_ph2_thoughts}}]
                  emotions: [{{auto_ph2_emotions}}]
                  issues: [PR2]
            - id: PH3
              actions:
                - id: A301
                  thoughts: [{{auto_ph3_thoughts}}]
                  emotions: [{{auto_ph3_emotions}}]
                  issues: [PR3]
          problems:
            - id: PR1
              summary: {{ph1_problem_summary}}
              linked_actions: [A101]
              occurs_in_phases: [PH1]
            - id: PR2
              summary: {{ph2_problem_summary | default:""}}
              linked_actions: [A201]
              occurs_in_phases: [PH2]
            - id: PR3
              summary: {{ph3_problem_summary | default:""}}
              linked_actions: [A301]
              occurs_in_phases: [PH3]
          numbering_policy:
            - prefix: PH=Phase, A=Action, PR=Problem
    
    - name: "auto_propose_customer_problem_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_課題定義/customer_problem_map_proposed_mermaid.md"
      content: |
        ```mermaid
        flowchart TD
          subgraph PH1[フェーズ1]
            A101[行動: A101]
            A101 --> PR1[PR1 {{ph1_problem_summary}}]
          end
          subgraph PH2[フェーズ2]
            A201[行動: A201]
            A201 --> PR2[PR2 {{ph2_problem_summary}}]
          end
          subgraph PH3[フェーズ3]
            A301[行動: A301]
            A301 --> PR3[PR3 {{ph3_problem_summary}}]
          end
        ```
    
    - name: "display_proposal"
      action: "display"
      content: |
        ✍️ 自動推測（PR1/PR2/PR3）を出力しました。修正点を入力してください（空欄は採用）。
    
    - name: "collect_mapping_corrections"
      action: "ask_questions_with_template"
      template: |
        === 追加/修正テンプレ（空欄は採用） ===
        PH1: 要約/why×3/impact/リンクAID
        →
        PH2: 要約/why×3/impact/リンクAID
        →
        PH3: 要約/why×3/impact/リンクAID
        →
        =====================================
    
    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正で最終成果物を生成します（phase別PRを含む）。よろしいですか？"
    
    - name: "export_customer_problem_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_課題定義/customer_problem_map.yaml"
      content: |
        customer_problem_map:
          persona_ref: P001
          phases:
            - id: PH1
              actions:
                - id: A101
                  thoughts: [{{ph1_thoughts_final | default:auto_ph1_thoughts}}]
                  emotions: [{{ph1_emotions_final | default:auto_ph1_emotions}}]
                  issues: [PR1]
            - id: PH2
              actions:
                - id: A201
                  thoughts: [{{ph2_thoughts_final | default:auto_ph2_thoughts}}]
                  emotions: [{{ph2_emotions_final | default:auto_ph2_emotions}}]
                  issues: [PR2]
            - id: PH3
              actions:
                - id: A301
                  thoughts: [{{ph3_thoughts_final | default:auto_ph3_thoughts}}]
                  emotions: [{{ph3_emotions_final | default:auto_ph3_emotions}}]
                  issues: [PR3]
          problems:
            - id: PR1
              summary: {{ph1_problem_summary_final | default:ph1_problem_summary}}
              causes: [{{ph1_why1}}, {{ph1_why2}}, {{ph1_why3}}]
              impact: {{ph1_impact}}
              linked_actions: [{{ph1_linked_actions_final | default:"A101"}}]
              occurs_in_phases: [PH1]
            - id: PR2
              summary: {{ph2_problem_summary_final | default:ph2_problem_summary}}
              causes: [{{ph2_why1}}, {{ph2_why2}}, {{ph2_why3}}]
              impact: {{ph2_impact}}
              linked_actions: [{{ph2_linked_actions_final | default:"A201"}}]
              occurs_in_phases: [PH2]
            - id: PR3
              summary: {{ph3_problem_summary_final | default:ph3_problem_summary}}
              causes: [{{ph3_why1}}, {{ph3_why2}}, {{ph3_why3}}]
              impact: {{ph3_impact}}
              linked_actions: [{{ph3_linked_actions_final | default:"A301"}}]
              occurs_in_phases: [PH3]
          numbering_policy:
            - prefix: PH=Phase, A=Action, PR=Problem
    
    - name: "export_customer_problem_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_課題定義/customer_problem_map_mermaid.md"
      content: |
        ```mermaid
        flowchart TD
          subgraph PH1[フェーズ1]
            A101[行動: A101]
            A101 --> PR1[PR1 {{ph1_problem_summary_final | default:ph1_problem_summary}}]
          end
          subgraph PH2[フェーズ2]
            A201[行動: A201]
            A201 --> PR2[PR2 {{ph2_problem_summary_final | default:ph2_problem_summary}}]
          end
          subgraph PH3[フェーズ3]
            A301[行動: A301]
            A301 --> PR3[PR3 {{ph3_problem_summary_final | default:ph3_problem_summary}}]
          end
        ```
    
    - name: "generate_problem_statement"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_課題定義/problem_todo.md"
      content: |
        # アプリ 課題定義（仮説）
        
        ## フェーズ別の主要課題
        - PH1 → PR1: {{ph1_problem_summary_final | default:ph1_problem_summary}}
        - PH2 → PR2: {{ph2_problem_summary_final | default:ph2_problem_summary}}
        - PH3 → PR3: {{ph3_problem_summary_final | default:ph3_problem_summary}}
        
        ## 現状の対処と限界
        {{current_solution}}
    
    - name: "export_problem_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_課題定義/problem_map.yaml"
      content: |
        problem_map:
          persona_ref: P001
          problems:
            - id: PR1
              summary: {{ph1_problem_summary_final | default:ph1_problem_summary}}
              causes: [{{ph1_why1}}, {{ph1_why2}}, {{ph1_why3}}]
              impact: {{ph1_impact}}
              linked_actions: [{{ph1_linked_actions_final | default:"A101"}}]
              occurs_in_phases: [PH1]
            - id: PR2
              summary: {{ph2_problem_summary_final | default:ph2_problem_summary}}
              causes: [{{ph2_why1}}, {{ph2_why2}}, {{ph2_why3}}]
              impact: {{ph2_impact}}
              linked_actions: [{{ph2_linked_actions_final | default:"A201"}}]
              occurs_in_phases: [PH2]
            - id: PR3
              summary: {{ph3_problem_summary_final | default:ph3_problem_summary}}
              causes: [{{ph3_why1}}, {{ph3_why2}}, {{ph3_why3}}]
              impact: {{ph3_impact}}
              linked_actions: [{{ph3_linked_actions_final | default:"A301"}}]
              occurs_in_phases: [PH3]
          numbering_policy:
            - prefix: PR=Problem, A=Action
    
    - name: "export_problem_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_課題定義/problem_map_mermaid.md"
      content: |
        ```mermaid
        flowchart TD
          subgraph PH1[フェーズ1]
            A101[行動: A101]
            A101 --> PR1[PR1 {{ph1_problem_summary_final | default:ph1_problem_summary}}]
          end
          subgraph PH2[フェーズ2]
            A201[行動: A201]
            A201 --> PR2[PR2 {{ph2_problem_summary_final | default:ph2_problem_summary}}]
          end
          subgraph PH3[フェーズ3]
            A301[行動: A301]
            A301 --> PR3[PR3 {{ph3_problem_summary_final | default:ph3_problem_summary}}]
          end
          classDef phase fill:#f9d71c
          classDef action fill:#fff9c4
          classDef problem fill:#5fb3d4
          class PH1,PH2,PH3 phase
          class A101,A201,A301 action
          class PR1,PR2,PR3 problem
        ```
    
    # === 完了通知（両モード） ===
    - name: "notify_completion"
      action: "display"
      content: |
        {{#if (eq execution_mode.mode 'multi_persona')}}
        ✅ 複数ペルソナモード完了：
        
        **ペルソナ別課題マップ**:
        {{#multi_persona_problems.personas}}
        - 📁 {{id}}_{{name | slugify}}/
          - problem_map_{{id}}.yaml
          - problem_map_{{id}}_mermaid.md
        {{/multi_persona_problems.personas}}
        
        **統合ファイル**:
        - integrated_problem_map.yaml
        - integrated_problem_map_mermaid.md
        
        🚀 **Opportunity仮説対応完了**: FH001-FH014との完全対応
        🔴 Critical: {{multi_persona_problems.personas | map: 'problems_by_phase' | map: 'PH1' | flatten | where: 'linked_hypotheses', 'contains', 'FH001' | size}}件
        🟡 High: {{multi_persona_problems.personas | map: 'problems_by_phase' | map: 'PH2' | flatten | where: 'linked_hypotheses', 'contains', 'FH005' | size}}件
        
        次のステップ: ソリューション定義・仮説検証計画
        {{else}}
        ✅ 単一ペルソナモード完了：
        - 02_課題定義/customer_problem_map_proposed.yaml / customer_problem_map_proposed_mermaid.md
        - 02_課題定義/customer_problem_map.yaml
        - 02_課題定義/problem_map.yaml / problem_map_mermaid.md
        - 02_課題定義/problem_todo.md
        
        📝 **質問ベースモード完了**: 仮説は未設定
        次のステップ: 
        1. 実際のユーザーインタビューで課題検証
        2. Opportunity仮説の設定・対応づけ
        3. ソリューション定義
        {{/if}}
```

## 生成される成果物

### 複数ペルソナモード
#### ペルソナ別ファイル（各ペルソナ×2ファイル）
- `{{persona_id}}_{{name}}/problem_map_{{persona_id}}.yaml` - ペルソナ別課題マップYAML
- `{{persona_id}}_{{name}}/problem_map_{{persona_id}}_mermaid.md` - ペルソナ別課題Mermaid図

#### 統合ファイル
- `integrated_problem_map.yaml` - 4ペルソナ統合課題マップ
- `integrated_problem_map_mermaid.md` - 統合課題可視化

### 単一ペルソナモード
- `customer_problem_map_proposed.yaml` / `customer_problem_map_proposed_mermaid.md`
- `customer_problem_map.yaml` / `problem_map.yaml` / `problem_map_mermaid.md`
- `problem_todo.md`

### 特徴

#### 🚀 複数ペルソナモード（4ペルソナのexperience_mapがある場合）
- **自動課題抽出**: 4ペルソナのexperience_mapから課題を自動検出
- **Opportunity仮説対応**: FH001-FH014との完全対応づけ
- **優先度自動判定**: Critical(FH001-004) > High(FH005-007) > Medium(others)
- **ペルソナ横断課題**: 複数ペルソナに影響する共通課題の特定
- **サブフォルダ構成**: ペルソナ別に整理された課題マップ

#### 📝 単一ペルソナモード（experience_mapが少ない場合）
- **質問収集**: 従来の質問形式で課題情報を収集
- **Sense/Focus連携**: 既存調査結果からの候補自動提示
- **仮説準備**: 後続の仮説設定・検証に向けた構造化

#### 共通機能
- **モード自動判定**: ペルソナファイル存在確認で適切なモードを自動選択
- **仮説統合**: Opportunity仮説との対応づけによる検証計画策定
- **優先度管理**: 仮説の重要度に基づく課題優先順位付け

## 次のコマンド
→ `ソリューション定義` で解決策を構造化  
→ `仮説検証計画` で検証方法を策定
