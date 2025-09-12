# 02 仮説駆動: 課題定義（AIPMハッカソン）

## 最初に質問（実行前に回答してください）
- フェーズごとの困りごと（最大3つ、各1行）
  - PH1の困りごと（必須）
  - PH2の困りごと（任意）
  - PH3の困りごと（任意）
- 各困りごとの「なぜ」×3（任意：空欄なら自動補完）
- 各困りごとの影響（時間/頻度/波及）（任意：空欄なら自動補完）
- 現在の対処（任意）

> `experience_map.yaml` の PHx/Axxx を参照し、自動で行動候補を引き当てます。

## 目的
コア課題の検討に加え、フェーズ別に分解した複数のProblemをActivityに紐づけて可視化します。

## 実行手順
```yaml
- trigger: "(仮説駆動_課題定義|HypothesisProblem)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['Flow/**/*.yaml','Flow/**/*.md','**/problem_map.yaml','**/customer_problem_map.yaml']))}}"]
      instructions: |
        スレッドと直近生成物から、PH1/PH2/PH3の候補やPR要約の下書きを抽出して提示してください。
        出力は display用テキストとし、空欄は含めずに簡潔に要点のみ列挙。
      store_as: "auto_phase_hints"

    - name: "ingest_sense_focus"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/1_sense/02_sense__顧客調査/sense_customer_research.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/05_sense__インタビュー分析（個別）/draft_interview_analysis_*.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/06_sense__リサーチサマリー（全体）/draft_research_summary.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/07_sense__オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
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
    - name: "prefill_from_yaml"
      action: "display"
      content: |
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
      action: "wait_for_all_answers"
    
    # 提案（フェーズ別PR）
    - name: "auto_propose_customer_problem_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/customer_problem_map_proposed.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/customer_problem_map_proposed_mermaid.md"
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
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/customer_problem_map.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/customer_problem_map_mermaid.md"
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
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/problem_todo.md"
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
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/problem_map.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/02_仮説駆動__課題定義/problem_map_mermaid.md"
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
    
    - name: "notify_completion"
      action: "display"
      content: |
        ✅ 生成しました：
        - 02_仮説駆動__課題定義/customer_problem_map_proposed.yaml / customer_problem_map_proposed_mermaid.md
        - 02_仮説駆動__課題定義/customer_problem_map.yaml
        - 02_仮説駆動__課題定義/problem_map.yaml / problem_map_mermaid.md
        - 02_仮説駆動__課題定義/problem_todo.md
```

## 次のコマンド
→ `仮説駆動_ソリューションマップ` で解決策を構造化します
