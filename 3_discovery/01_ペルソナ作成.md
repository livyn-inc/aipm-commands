# 01 ペルソナ作成（統合版）

## 実行モード
### 🚀 **自動検出モード**（Sense+Focus成果物がある場合）
Sense+Focusフェーズの成果物から複数ペルソナを自動検出し、Opportunity仮説と対応づけた詳細ペルソナ・エクスペリエンスマップを生成

### 📝 **質問ベースモード**（成果物がない場合）
従来の質問形式でペルソナ情報を収集し、仮説として後で検証・修正できる状態に落とし込み

## 質問ベースモード - 事前質問
- 誰のためのアプリ（例: TODOアプリ）を作りますか？（必須）
- 年齢・性別・職業は？（必須）
- その人の1日の流れは？（必須）
- アプリに関する悩みは？（必須）
- 大切にしている価値観は？（任意）

> ヒント: `/aipm_hackathon/persona_candidates_100.md` を見ながら発想してOK！

## 自動検出対象ファイル
- `virtual_interview_log_kakakucom_ax.md` - 実際のインタビューデータ
- `sense_customer_research.md` - 3層ターゲット構造
- `focus_product_definition.md` - プロダクト定義・ターゲット
- `focus_positioning_statement.md` - ポジショニング・メッセージング
- `focus_lean_canvas_mermaid_v2.md` - リーンキャンバス・仮説統合
- `focus_opportunity_hypotheses.yaml` - 優先順位付け済み仮説

## 実行手順
```yaml
- trigger: "(ペルソナ作成|Persona Creation|仮説駆動_ペルソナ作成)"
  priority: high
  steps:
    # Step 1: モード判定（Sense+Focus成果物の存在確認）
    - name: "check_sense_focus_files"
      action: "analyze"
      data: [
        "{{find_files(patterns=['**/virtual_interview_log_kakakucom_ax.md'])}}", 
        "{{find_files(patterns=['**/sense_customer_research.md'])}}", 
        "{{find_files(patterns=['**/focus_product_definition.md'])}}", 
        "{{find_files(patterns=['**/focus_positioning_statement.md'])}}", 
        "{{find_files(patterns=['**/focus_lean_canvas_mermaid_v2.md'])}}", 
        "{{find_files(patterns=['**/focus_opportunity_hypotheses.yaml'])}}"
      ]
      instructions: |
        ファイル存在確認結果から実行モードを判定してください：
        - 3つ以上のファイルが存在する場合: "auto_detect"
        - それ以下の場合: "question_based"
        
        結果をJSONで返してください：
        {"mode": "auto_detect|question_based", "found_files": ["file1", "file2", ...]}
      store_as: "execution_mode"

    # === 自動検出モード ===
    - name: "auto_detect_personas"
      condition: "{{execution_mode.mode == 'auto_detect'}}"
      action: "analyze"
      data: [
        "{{read_files(find_files(patterns=['**/virtual_interview_log_kakakucom_ax.md']))}}",
        "{{read_files(find_files(patterns=['**/sense_customer_research.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_product_definition.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_positioning_statement.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_lean_canvas_mermaid_v2.md']))}}",
        "{{read_files(find_files(patterns=['**/focus_opportunity_hypotheses.yaml']))}}"
      ]
      instructions: |
        Sense+Focusフェーズの成果物から複数ペルソナを検出し、以下の構造で返してください：
        {
          "personas": [
            {
              "id": "P001",
              "type": "primary|secondary|tertiary",
              "name": "仮名",
              "age_gender": "年齢・性別",
              "occupation": "職業・役職",
              "company_context": "企業・組織背景",
              "daily_activities": ["活動1", "活動2", ...],
              "pain_points": ["課題1", "課題2", ...],
              "values": ["価値観1", "価値観2", ...],
              "opportunity_hypotheses": ["FH001", "FH002", ...],
              "phases": [
                {
                  "id": "PH1",
                  "name": "フェーズ名",
                  "description": "説明",
                  "actions": [{"id": "A101", "label": "行動", "notes": ["メモ"]}]
                }
              ]
            }
          ]
        }
      store_as: "detected_personas"

    # === 質問ベースモード ===
    - name: "show_persona_examples"
      condition: "{{execution_mode.mode == 'question_based'}}"
      action: "display"
      content: |
        📝 **質問ベースモード**で実行します
        
        💡 ペルソナの例（参考）
        - 働くママ（35）: 家事と仕事のタスクが混在
        - 大学生（20）: バイトや課題でスケジュール混乱
        - フリーランス（28）: 締切管理と優先度が苦手
        - シニア（68）: 小さい文字が見づらい
        
        さらに → `/aipm_hackathon/persona_candidates_100.md`

    - name: "collect_persona_info"
      condition: "{{execution_mode.mode == 'question_based'}}"
      action: "ask_questions"
      questions:
        - key: "persona_who"
          question: "誰のためのアプリ（例: TODOアプリ）を作りますか？"
          required: true
        - key: "age_gender"
          question: "年齢・性別は？"
          required: true
        - key: "occupation"
          question: "職業は？"
          required: true
        - key: "daily_routine"
          question: "その人の1日の流れは？（時系列で3-6項目）"
          required: true
        - key: "app_problem"
          question: "アプリに関する悩みは？"
          required: true
        - key: "values"
          question: "大切にしている価値観は？（任意）"
          required: false
      store_as: "persona_answers"

    - name: "refine_problem_brief"
      condition: "{{execution_mode.mode == 'question_based'}}"
      action: "ask_question"
      question: |
        ペルソナの悩みを1行で要約してください（仮説として）。
        例）「選択肢が多すぎて、最初の一歩が出ない」
      store_as: "problem_brief"

    - name: "generate_single_persona_structure"
      condition: "{{execution_mode.mode == 'question_based'}}"
      action: "analyze"
      data: ["{{persona_answers}}", "{{problem_brief}}"]
      instructions: |
        質問ベースの回答から単一ペルソナ構造を生成してください：
        {
          "personas": [
            {
              "id": "P001",
              "type": "primary",
              "name": "{{persona_answers.age_gender}}の{{persona_answers.occupation}}",
              "age_gender": "{{persona_answers.age_gender}}",
              "occupation": "{{persona_answers.occupation}}",
              "company_context": "一般的な{{persona_answers.occupation}}の環境",
              "daily_activities": [{{persona_answers.daily_routine}}を配列化],
              "pain_points": [{{persona_answers.app_problem}}を詳細化],
              "values": [{{persona_answers.values}}を配列化],
              "opportunity_hypotheses": [],
              "phases": [
                {
                  "id": "PH1",
                  "name": "準備・計画フェーズ",
                  "description": "{{persona_answers.persona_who}}を使う前の準備段階",
                  "actions": [{"id": "A101", "label": "基本行動", "notes": ["詳細メモ"]}]
                },
                {
                  "id": "PH2", 
                  "name": "実行・利用フェーズ",
                  "description": "実際に{{persona_answers.persona_who}}を使用する段階",
                  "actions": [{"id": "A201", "label": "メイン行動", "notes": ["詳細メモ"]}]
                },
                {
                  "id": "PH3",
                  "name": "完了・振り返りフェーズ", 
                  "description": "{{persona_answers.persona_who}}使用後の振り返り段階",
                  "actions": [{"id": "A301", "label": "完了行動", "notes": ["詳細メモ"]}]
                }
              ]
            }
          ]
        }
      store_as: "detected_personas"

    # === 共通処理（両モード） ===
    - name: "display_detected_personas"
      action: "display"
      content: |
        🎯 検出されたペルソナ：
        {{#detected_personas.personas}}
        - **{{type}}**: {{name}} ({{age_gender}}, {{occupation}})
        {{#if opportunity_hypotheses}}
          - 対応仮説: {{opportunity_hypotheses | join: ', '}}
        {{else}}
          - モード: 質問ベース（仮説なし）
        {{/if}}
        {{/detected_personas.personas}}
    
    - name: "confirm_generate_all"
      action: "confirm"
      message: |
        {{#if (eq execution_mode.mode 'auto_detect')}}
        🚀 自動検出モード: 検出された{{detected_personas.personas | size}}個のペルソナで全成果物を生成します。
        {{else}}
        📝 質問ベースモード: 作成されたペルソナで全成果物を生成します。
        {{/if}}
        よろしいですか？

    # 複数ペルソナの一括生成
    - name: "generate_all_personas"
      action: "loop"
      items: "{{detected_personas.personas}}"
      steps:
        - name: "create_persona_subfolder"
          action: "execute_shell"
          command: "mkdir -p {{patterns.flow_date}}/3_discovery/01_ペルソナ作成/{{item.id}}_{{item.name | slugify}}_{{item.type}}"
          
        - name: "generate_persona_md"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/01_ペルソナ作成/{{item.id}}_{{item.name | slugify}}_{{item.type}}/persona_{{item.type}}_{{item.name | slugify}}.md"
          content: |
            # {{item.type | title}}ペルソナ - {{item.name}}
            
            ## 基本情報
            - **ID**: {{item.id}}
            - **名前**: {{item.name}}
            - **年齢・性別**: {{item.age_gender}}
            - **職業・役職**: {{item.occupation}}
            - **企業・組織**: {{item.company_context}}
            
            ## 日常活動・行動パターン
            {{#item.daily_activities}}
            - {{.}}
            {{/item.daily_activities}}
            
            ## 課題・ペインポイント
            {{#item.pain_points}}
            - {{.}}
            {{/item.pain_points}}
            
            ## 価値観・重視する要素
            {{#item.values}}
            - {{.}}
            {{/item.values}}
            
            ## 対応するOpportunity仮説
            {{#if item.opportunity_hypotheses}}
            {{#item.opportunity_hypotheses}}
            - **{{.}}**: {{lookup ../focus_opportunity_hypotheses .}}
            {{/item.opportunity_hypotheses}}
            {{else}}
            - 質問ベースモードで作成されたため、仮説は未設定です
            - Discovery フェーズで仮説を設定・検証してください
            {{/if}}
            
            ## エクスペリエンスフェーズ
            {{#item.phases}}
            ### {{id}}: {{name}}
            {{description}}
            
            #### 主要アクション
            {{#actions}}
            - **{{id}}**: {{label}}
            {{#notes}}
              - {{.}}
            {{/notes}}
            {{/actions}}
            {{/item.phases}}
            
            ---
            **作成日**: {{today}}  
            **ベース**: Sense+Focusフェーズ統合分析  
            **検証予定**: Discovery フェーズ

        - name: "generate_experience_yaml"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/01_ペルソナ作成/{{item.id}}_{{item.name | slugify}}_{{item.type}}/experience_map_{{item.type}}_{{item.name | slugify}}.yaml"
          content: |
            persona:
              id: {{item.id}}
              name: "{{item.name}}"
              profile:
                age_gender: "{{item.age_gender}}"
                occupation: "{{item.occupation}}"
                company_context: "{{item.company_context}}"
              behaviors: 
            {{#item.daily_activities}}
                - "{{.}}"
            {{/item.daily_activities}}
              values:
            {{#item.values}}
                - "{{.}}"
            {{/item.values}}
            
            experience_map:
              phases:
            {{#item.phases}}
                - id: {{id}}
                  name: "{{name}}"
                  description: "{{description}}"
                  actions:
            {{#actions}}
                    - id: {{id}}
                      label: "{{label}}"
                      notes:
            {{#notes}}
                        - "{{.}}"
            {{/notes}}
            {{/actions}}
            {{/item.phases}}
            
            opportunity_hypotheses_mapping:
            {{#if item.opportunity_hypotheses}}
            {{#item.phases}}
              {{id}}:
            {{#../item.opportunity_hypotheses}}
                - "{{.}}: {{lookup ../../focus_opportunity_hypotheses .}}"
            {{/../item.opportunity_hypotheses}}
            {{/item.phases}}
            {{else}}
              # 質問ベースモードで作成されたため、仮説マッピングは未設定
              # Discovery フェーズで仮説を設定・検証してください
            {{/if}}
            
            numbering_policy:
              - "prefix: P=Persona, PH=Phase, A=Action"
              - "以後の課題(PR)/解決策(SL)/ストーリー(ST)と連番で紐付けます"
              - "{{item.id}}: {{item.type | title}}ペルソナ（{{item.occupation}}）"
            {{#item.phases}}
              - "{{id}}: {{name}}"
            {{/item.phases}}

        - name: "generate_experience_mermaid"
          action: "create_markdown_file"
          path: "{{patterns.flow_date}}/3_discovery/01_ペルソナ作成/{{item.id}}_{{item.name | slugify}}_{{item.type}}/experience_map_{{item.type}}_{{item.name | slugify}}_mermaid.md"
          content: |
            # {{item.type | title}}ペルソナ エクスペリエンスマップ - {{item.name}}
            
            ## Mermaid図
            
            ```mermaid
            flowchart TD
                {{item.id}}[ペルソナ: {{item.name}}<br/>{{item.occupation}}<br/>{{item.age_gender}}]
                
            {{#item.phases}}
                {{item.id}} --> {{id}}[{{id}}: {{name}}]
            {{/item.phases}}
                
            {{#item.phases}}
            {{#actions}}
                {{../id}} --> {{id}}[{{id}}: {{label}}]
            {{/actions}}
            {{/item.phases}}
                
                %% フェーズ間の流れ
            {{#item.phases}}
            {{#unless @last}}
                {{id}} -.次フェーズ.-> {{lookup ../item.phases @index_plus_1 'id'}}
            {{/unless}}
            {{/item.phases}}
                
                %% 課題・ペインポイント
            {{#item.phases}}
            {{#actions}}
                {{id}} -.課題.-> PR{{@index_plus_1}}[PR{{@index_plus_1}}: {{notes.0}}]
            {{/actions}}
            {{/item.phases}}
                
                %% スタイリング
                classDef personaStyle fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
                classDef phaseStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
                classDef actionStyle fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
                classDef problemStyle fill:#ffebee,stroke:#d32f2f,stroke-width:2px
                
                class {{item.id}} personaStyle
            {{#item.phases}}
                class {{id}} phaseStyle
            {{/item.phases}}
            {{#item.phases}}
            {{#actions}}
                class {{id}} actionStyle
            {{/actions}}
            {{/item.phases}}
            ```
            
            ## Opportunity仮説との対応
            
            {{#if item.opportunity_hypotheses}}
            {{#item.phases}}
            ### {{id}}: {{name}}
            {{#../item.opportunity_hypotheses}}
            - **{{.}}**: {{lookup ../../focus_opportunity_hypotheses .}}
            {{/../item.opportunity_hypotheses}}
            {{/item.phases}}
            {{else}}
            ### 質問ベースモード
            - 仮説は未設定です
            - Discovery フェーズで仮説を設定・検証してください
            {{/if}}
            
            ## 検証ポイント
            
            {{#item.phases}}
            1. **{{name}}**: {{description}}での検証項目
            {{/item.phases}}
            
            ---
            **作成日**: {{today}}  
            **対応仮説**: {{item.opportunity_hypotheses | join: ', '}}  
            **検証計画**: Discovery フェーズ段階的検証

    # 統合エクスペリエンスマップ生成
    - name: "generate_integrated_experience_map"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/01_ペルソナ作成/experience_map_integrated.yaml"
      content: |
        integrated_experience_map:
          meta:
            created_date: "{{today}}"
            project: "{{project_name | default: 'Product'}}"
            phase: "Discovery - ペルソナ作成"
            description: "{{detected_personas.personas | size}}層ペルソナの統合エクスペリエンスマップ"
          
          personas:
        {{#detected_personas.personas}}
            {{type}}:
              id: {{id}}
              name: "{{name}}"
              role: "{{occupation}}"
              focus: "{{pain_points.0}}"
        {{/detected_personas.personas}}

          integrated_phases:
        {{#detected_personas.personas.0.phases}}
            - id: IPH{{@index_plus_1}}
              name: "{{name}}"
              description: "{{description}}"
              persona_mapping:
        {{#../../detected_personas.personas}}
                {{id}}: "{{lookup phases ../../../@index 'id'}}: {{lookup phases ../../../@index 'name'}}"
        {{/../../detected_personas.personas}}
              opportunity_hypotheses:
        {{#../../detected_personas.personas}}
        {{#opportunity_hypotheses}}
                - "{{.}}"
        {{/opportunity_hypotheses}}
        {{/../../detected_personas.personas}}
        {{/detected_personas.personas.0.phases}}

          validation_priorities:
            critical:
        {{#detected_personas.personas}}
        {{#opportunity_hypotheses}}
        {{#if (eq . 'FH001' 'FH002' 'FH003' 'FH004')}}
              - "{{.}}: {{lookup ../../focus_opportunity_hypotheses .}}"
        {{/if}}
        {{/opportunity_hypotheses}}
        {{/detected_personas.personas}}
            
            high:
        {{#detected_personas.personas}}
        {{#opportunity_hypotheses}}
        {{#if (eq . 'FH005' 'FH006' 'FH007')}}
              - "{{.}}: {{lookup ../../focus_opportunity_hypotheses .}}"
        {{/if}}
        {{/opportunity_hypotheses}}
        {{/detected_personas.personas}}

    # サマリー生成
    - name: "generate_personas_summary"
      action: "create_markdown_file"
      path: "{{patterns.flow_date}}/3_discovery/01_ペルソナ作成/personas_summary.md"
      content: |
        # ペルソナサマリー - {{detected_personas.personas | size}}層構造
        
        ## 概要
        
        Sense+Focusフェーズの成果物から{{detected_personas.personas | size}}個のペルソナを検出し、
        Opportunity仮説（FH001-FH014）と対応づけた詳細エクスペリエンスマップを作成しました。
        
        ## ペルソナ一覧
        
        {{#detected_personas.personas}}
        ### {{@index_plus_1}}. {{type | title}}ペルソナ: {{name}}
        
        - **ID**: {{id}}
        - **プロフィール**: {{age_gender}}, {{occupation}}
        - **企業背景**: {{company_context}}
        - **主要課題**: {{pain_points.0}}
        - **対応仮説**: {{opportunity_hypotheses | join: ', '}}
        
        #### エクスペリエンスフェーズ
        {{#phases}}
        - **{{id}}**: {{name}} - {{description}}
        {{/phases}}
        
        {{/detected_personas.personas}}
        
        ## 仮説マッピング
        
        {{#if (any detected_personas.personas 'opportunity_hypotheses')}}
        ### Discovery重点検証（FH001-FH004）
        {{#detected_personas.personas}}
        {{#opportunity_hypotheses}}
        {{#if (eq . 'FH001' 'FH002' 'FH003' 'FH004')}}
        - **{{.}}** ({{../name}}): {{lookup ../../focus_opportunity_hypotheses .}}
        {{/if}}
        {{/opportunity_hypotheses}}
        {{/detected_personas.personas}}
        
        ### Discovery検証（FH005-FH007）
        {{#detected_personas.personas}}
        {{#opportunity_hypotheses}}
        {{#if (eq . 'FH005' 'FH006' 'FH007')}}
        - **{{.}}** ({{../name}}): {{lookup ../../focus_opportunity_hypotheses .}}
        {{/if}}
        {{/opportunity_hypotheses}}
        {{/detected_personas.personas}}
        {{else}}
        ### 質問ベースモード
        - 仮説マッピングは未設定です
        - Discovery フェーズで以下を実施してください：
          1. 仮説の設定・定義
          2. ペルソナとの対応づけ
          3. 段階的検証計画の策定
        {{/if}}
        
        ## 検証計画
        
        {{#if (any detected_personas.personas 'opportunity_hypotheses')}}
        ### Phase 1: 重点検証（1ヶ月目）
        - 統合UI/UX・セルフサービス・コンテキスト管理・ROI可視化
        
        ### Phase 2: 標準検証（2ヶ月目）  
        - 価格受容性・職種特化・自動収集
        
        ### Phase 3: 後期検証（3ヶ月目）
        - 段階導入・組織カスタマイズ
        {{else}}
        ### 質問ベースモード検証計画
        
        ### Phase 1: ペルソナ検証（1ヶ月目）
        - 作成したペルソナの実在性・代表性確認
        - 実際のユーザーインタビュー実施
        
        ### Phase 2: 課題・ニーズ検証（2ヶ月目）
        - 想定した課題・ペインポイントの妥当性確認
        - ソリューション仮説の設定・検証
        
        ### Phase 3: エクスペリエンス検証（3ヶ月目）
        - エクスペリエンスマップの精度向上
        - プロトタイプでの実証実験
        {{/if}}
        
        ---
        **作成日**: {{today}}  
        **検出ペルソナ数**: {{detected_personas.personas | size}}  
        **対応仮説数**: {{detected_personas.personas | map: 'opportunity_hypotheses' | flatten | uniq | size}}

    - name: "notify_completion"
      action: "display"
      content: |
        ✅ {{detected_personas.personas | size}}個のペルソナで全成果物を生成しました：
        
        {{#detected_personas.personas}}
        **{{type | title}}ペルソナ ({{name}})**:
        - 📁 {{id}}_{{name | slugify}}_{{type}}/
          - persona_{{type}}_{{name | slugify}}.md
          - experience_map_{{type}}_{{name | slugify}}.yaml  
          - experience_map_{{type}}_{{name | slugify}}_mermaid.md
        
        {{/detected_personas.personas}}
        **統合ファイル**:
        - experience_map_integrated.yaml
        - personas_summary.md
        
        {{#if (eq execution_mode.mode 'auto_detect')}}
        🚀 **自動検出モード完了**: Opportunity仮説との対応完了
        次のステップ: カスタマージャーニーマップ作成
        {{else}}
        📝 **質問ベースモード完了**: 仮説は未設定
        次のステップ: 
        1. 実際のユーザーインタビューでペルソナ検証
        2. 課題・ニーズの仮説設定
        3. カスタマージャーニーマップ作成
        {{/if}}
```

## 生成される成果物

### 個別ペルソナファイル（各ペルソナ×3ファイル）
- `persona_{type}_{name}.md` - 詳細ペルソナ定義
- `experience_map_{type}_{name}.yaml` - エクスペリエンスマップYAML
- `experience_map_{type}_{name}_mermaid.md` - Mermaidフロー図

### 統合ファイル
- `experience_map_integrated.yaml` - 全ペルソナ統合マップ
- `personas_summary.md` - サマリー・検証計画

### 特徴

#### 🚀 自動検出モード（Sense+Focus成果物がある場合）
- **自動検出**: Sense+Focus成果物から複数ペルソナを自動抽出
- **仮説対応**: FH001-FH014のOpportunity仮説と完全対応
- **PH1-PH3定義**: 各ペルソナのエクスペリエンスフェーズを詳細化
- **検証計画**: Discovery フェーズでの段階的検証計画

#### 📝 質問ベースモード（成果物がない場合）
- **質問収集**: 従来の質問形式でペルソナ情報を収集
- **仮説生成**: 回答から単一ペルソナ構造を生成
- **PH1-PH3定義**: 準備・実行・完了の3フェーズを自動設定
- **検証準備**: 後続のインタビュー・検証に向けた仮説として整理

#### 共通機能
- **モード自動判定**: ファイル存在確認で適切なモードを自動選択
- **統一出力**: 両モードとも同じ形式でファイル生成
- **柔軟対応**: プロジェクトの進行状況に応じて適切な手法を選択

## 次のコマンド
→ `カスタマージャーニーマップ作成` でジャーニー詳細化
→ `課題定義` で課題を深掘り

    - name: "collect_persona_info"
      action: "ask_questions_with_template"
      template: |
        === ペルソナ情報入力テンプレート ===
        1. 誰のためのアプリ（例: TODOアプリ）？
        （候補: {{prefill.persona_who}} / Sense: {{sense_hints.persona_who_hint}}）
        → 【あなたの回答】：
        
        2. 年齢・性別・職業は？
        （候補: {{prefill.age_gender}} / {{prefill.occupation}} / Sense: {{sense_hints.age_gender_hint}} / {{sense_hints.occupation_hint}}）
        → 【あなたの回答】：
        
        3. 1日の流れ（時系列で3-6項目）
        （候補: {{prefill.daily_routine}} / Sense: {{sense_hints.daily_routine_hint}}）
        → 【あなたの回答】：
        
        4. アプリに関する悩み
        （候補: {{prefill.app_problem}} / Sense: {{sense_hints.app_problem_hint}}）
        → 【あなたの回答】：
        
        5. 大切にしている価値観（任意）
        （候補: {{prefill.values}} / Sense: {{sense_hints.values_hint}}）
        → 【あなたの回答】：
        =====================================
    
    - name: "wait_inputs"
      action: "wait_for_all_answers"
    
    - name: "refine_brief"
      action: "interactive_dialog"
      message: |
        いいですね！
        ペルソナの悩みを1行で要約してください（仮説として）。
        例）「選択肢が多すぎて、最初の一歩が出ない」
    
    - name: "confirm_generate"
      action: "confirm"
      message: "収集した内容で全成果物（persona.md / experience_map.yaml / mermaid）を生成します。よろしいですか？"
    
    - name: "generate_persona"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/01_ペルソナ作成/persona_todo.md"
      content: |
        # アプリ ペルソナ仮説（例: TODOアプリ）
        
        ## 基本情報
        - 名前（任意）: {{generated_name}}
        - 年齢・性別: {{age_gender}}
        - 職業: {{occupation}}
        
        ## 1日の流れ
        {{daily_routine}}
        
        ## アプリに関する課題（仮説）
        - 要約: {{problem_brief}}
        - 具体例: {{specific_problems}}
        
        ## 価値観・重視
        {{values}}
        
        ## 注意
        - 本文書は仮説です。検証前提で更新します。
        - 思い込みの可能性: {{assumptions}}
    
    - name: "export_experience_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/01_ペルソナ作成/experience_map.yaml"
      content: |
        persona:
          id: P001
          name: {{generated_name}}
          profile:
            age_gender: {{age_gender}}
            occupation: {{occupation}}
          behaviors: {{daily_routine | to_list}}
          values: {{values}}
        experience_map:
          phases:
            - id: PH1
              name: "買い物前の準備"
              actions:
                - id: A101
                  label: "例: 連絡帳を書く"
                  notes: []
            - id: PH2
              name: "レジ/作業中"
              actions:
                - id: A201
                  label: "例: その場で5分タスクに着手"
                  notes: []
            - id: PH3
              name: "完了後の見直し"
              actions:
                - id: A301
                  label: "例: 今日の3枠を確認"
                  notes: []
        numbering_policy:
          - prefix: P=Persona, PH=Phase, A=Action
          - 以後の課題(PR)/解決策(SL)/ストーリー(ST)と連番で紐付けます
    
    - name: "export_experience_map_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/01_ペルソナ作成/experience_map_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          P001[ペルソナ: {{generated_name}}]
          P001 --> PH1[PH1 買い物前の準備]
          P001 --> PH2[PH2 レジ/作業中]
          P001 --> PH3[PH3 完了後の見直し]
          PH1 --> A101[A101 行動]
          PH2 --> A201[A201 行動]
          PH3 --> A301[A301 行動]
        ```
    
    - name: "notify_completion"
      action: "display"
      content: |
        ✅ 生成しました：
        - 01_ペルソナ作成/persona_todo.md
        - 01_ペルソナ作成/experience_map.yaml
        - 01_ペルソナ作成/experience_map_mermaid.md
```

## 次のコマンド
→ `仮説駆動_課題定義` で課題を深掘りします
