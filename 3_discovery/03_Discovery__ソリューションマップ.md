# 03 仮説駆動: ソリューションマップ（AIPMハッカソン）

## 最初に質問（実行前に回答してください）
- 対象ペルソナと主要課題群は？（必須・PR基準）
- PR1/PR2/PR3 それぞれへの解決案（必須・1つずつ）
- 各解決案が主に効くフェーズ（任意: 利用前/利用中/利用後）
- 参考となる関連Activity（任意: AID。空欄なら自動候補を採用）

## 目的
Problem（PR）基準で解決案を設計し、PR→SL→A の接続で可視化します。

## 実行手順
```yaml
- trigger: "(仮説駆動_ソリューションマップ|HypothesisSolutionMap)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/problem_map.yaml','**/customer_problem_map.yaml','**/story_map.yaml']))}}"]
      instructions: |
        直近のPR/Activity/Storyの対応から、PR1/PR2/PR3に対する既定ソリューション案を抽出してください。
        出力は display 用テキストで、PRx: 候補… の形式で簡潔に。
      store_as: "auto_solution_hints"

    - name: "ingest_sense_focus"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/1_sense/07_sense__オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/06_sense__リサーチサマリー（全体）/draft_research_summary.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/02_sense__顧客調査/sense_customer_research.md']))}}"
      ]
      instructions: |
        Senseのオポチュニティ/サマリー/顧客調査、およびFocusの定義から、PRごとの解決案候補・効くフェーズ・関連A候補を抽出し、JSON1件で返してください。
        keys:
          pr1_solution_hint, pr1_phase_hint, pr1_actions_hint,
          pr2_solution_hint, pr2_phase_hint, pr2_actions_hint,
          pr3_solution_hint, pr3_phase_hint, pr3_actions_hint,
          persona_and_problem_hint
      store_as: "sense_focus"
    - name: "prefill_from_yaml"
      action: "display"
      content: |
        🔎 候補（自動抽出）
        {{auto_solution_hints}}
        
        🔎 Sense/Focus 由来の候補
        - PR1: {{sense_focus.pr1_solution_hint}}（フェーズ: {{sense_focus.pr1_phase_hint}}、A: {{sense_focus.pr1_actions_hint}}）
        - PR2: {{sense_focus.pr2_solution_hint}}（フェーズ: {{sense_focus.pr2_phase_hint}}、A: {{sense_focus.pr2_actions_hint}}）
        - PR3: {{sense_focus.pr3_solution_hint}}（フェーズ: {{sense_focus.pr3_phase_hint}}、A: {{sense_focus.pr3_actions_hint}}）
    
    - name: "show_map_examples"
      action: "display"
      content: |
        🗺️ PR基準の設計例（A→PR→SL）
        - A101 → PR1 → SL1
        - A201 → PR2 → SL2
        - A301 → PR3 → SL3
    
    - name: "collect_solution_map"
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
      action: "wait_for_all_answers"
    
    - name: "spot_mvp_candidates"
      action: "interactive_dialog"
      message: |
        いいですね！ 3つの解決案の中から、MVPでまず検証すべき順序を1→3で指定してください（例: SL2, SL1, SL3）。
    
    # ここから：自動提案 → 修正収集 → 確定生成
    - name: "auto_propose_solution_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/03_仮説駆動__ソリューションマップ/solution_map_proposed.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/03_仮説駆動__ソリューションマップ/solution_map_proposed_mermaid.md"
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
      path: "Flow/{{today}}/{{flow_dir}}/03_仮説駆動__ソリューションマップ/solution_map_todo.md"
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
      path: "Flow/{{today}}/{{flow_dir}}/03_仮説駆動__ソリューションマップ/solution_map.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/03_仮説駆動__ソリューションマップ/solution_map_mermaid.md"
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
    
    - name: "notify_completion"
      action: "display"
      content: |
        ✅ 生成しました：
        - （提案）03_仮説駆動__ソリューションマップ/solution_map_proposed.yaml / solution_map_proposed_mermaid.md
        - （確定）03_仮説駆動__ソリューションマップ/solution_map_todo.md / solution_map.yaml / solution_map_mermaid.md / ソリューション仮説文
```

## 次のコマンド
→ `仮説駆動_ストーリーマップ` でMVPストーリー化します
