# 04 仮説駆動: ストーリーマップ（AIPMハッカソン）

## 最初に質問（実行前に回答してください）
- ペルソナ（既定: P001）（必須）
- PR1/PR2/PR3 それぞれに対するユーザーストーリー（As a / I want / So that）（必須）
- 各ストーリーの受け入れ基準（必須）
- リリース区分（MVP or Release1）（必須）
- 実装順序（任意）

## 目的
Problem（PR）とSolution（SL）に連動し、行動（A）から派生するユーザーストーリー（ST）をA→PR→SL→STの流れで可視化します。

## 実行手順
```yaml
- trigger: "(仮説駆動_ストーリーマップ|HypothesisStoryMap)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['**/solution_map.yaml','**/problem_map.yaml','**/customer_problem_map.yaml']))}}"]
      instructions: |
        直近のPR/SL/Activity対応から、PR→SL→STの初期チェーン（ST1/2/3）を推定して表示用テキストにまとめてください。
      store_as: "auto_story_hints"

    - name: "ingest_sense_focus"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/1_sense/07_sense__オポチュニティ仮説抽出/sense_opportunities.yaml']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/06_sense__リサーチサマリー（全体）/draft_research_summary.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md','**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/1_sense/02_sense__顧客調査/sense_customer_research.md']))}}",
        "{{read_files(find_files(patterns=['**/03_仮説駆動__ソリューションマップ/solution_map.yaml']))}}"
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
    
    # ここから：自動提案（A→PR→SL→ST） → 修正収集 → 確定生成
    - name: "auto_propose_story_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_仮説駆動__ストーリーマップ/story_map_proposed.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/04_仮説駆動__ストーリーマップ/story_map_proposed_mermaid.md"
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
        ✍️ 自動提案を `story_map_proposed.yaml` / `story_map_proposed_mermaid.md` に出力しました。修正事項を入力してください（空欄は採用）。
    
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
      message: "提案＋修正内容で最終成果物（story_map_todo.md / story_map.yaml / story_map_mermaid.md）を生成します。よろしいですか？"
    
    - name: "generate_story_map_md"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/04_仮説駆動__ストーリーマップ/story_map_todo.md"
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
      path: "Flow/{{today}}/{{flow_dir}}/04_仮説駆動__ストーリーマップ/story_map.yaml"
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
      path: "Flow/{{today}}/{{flow_dir}}/04_仮説駆動__ストーリーマップ/story_map_mermaid.md"
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
        - 04_仮説駆動__ストーリーマップ/story_map_proposed.yaml / story_map_proposed_mermaid.md
        - 04_仮説駆動__ストーリーマップ/story_map_todo.md / story_map.yaml / story_map_mermaid.md
```

## 次のコマンド
→ `仮説駆動_UIワイヤーフレーム` で画面設計へ
