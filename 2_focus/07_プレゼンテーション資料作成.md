# プレゼンテーション資料作成コマンド

## 概要
Sense & Focusフェーズの成果を統合し、企画の蓋然性を説明するプロフェッショナルなプレゼンテーション資料を作成します。

## トリガー
- "プレゼン資料作成"
- "社内プレゼン作成"
- "企画説明資料作成"
- "Sense Focus プレゼン"

## 実行フロー

### 1. 情報収集フェーズ
```yaml
questions:
  - key: project_id
    question: "プロジェクトIDを入力してください"
    required: true
  
  - key: presentation_title
    question: "プレゼンテーションのタイトルを入力してください"
    default: "{{project_id}} - 企画の蓋然性説明"
    required: true
  
  - key: presentation_purpose
    question: "プレゼンテーションの目的を選択してください"
    options:
      - "社内承認・意思決定"
      - "ステークホルダー説明"
      - "投資家ピッチ"
      - "チーム共有"
    required: true
  
  - key: target_audience
    question: "ターゲットオーディエンスを入力してください"
    default: "社内ステークホルダー"
    required: true
  
  - key: presentation_length
    question: "プレゼンテーションの長さを選択してください"
    options:
      - "短め（10-15スライド）"
      - "標準（15-20スライド）"
      - "詳細（20-25スライド）"
    default: "標準（15-20スライド）"
    required: true
  
  - key: include_appendix
    question: "付録スライドを含めますか？"
    options:
      - "はい"
      - "いいえ"
    default: "はい"
    required: true
  
  - key: export_formats
    question: "エクスポート形式を選択してください（複数選択可）"
    options:
      - "PDF"
      - "HTML"
      - "PowerPoint"
    default: "PDF"
    required: true
```

### 2. 参照資料収集
```yaml
reference_sources:
  sense_phase:
    - "1_sense/01_競合調査/"
    - "1_sense/02_顧客調査/"
    - "1_sense/03_インタビュー設計/"
  
  focus_phase:
    - "2_focus/04_オポチュニティ仮説抽出/"
    - "2_focus/05_プロダクト定義/"
    - "2_focus/07_市場規模推定/"
    - "2_focus/08_ラフロードマップ作成/"
    - "2_focus/09_OKR作成/"
    - "2_focus/10_リーンキャンバス作成/"
    - "2_focus/12_アクションプラン作成/"
  
  reference_materials:
    - "00_参考資料/"
```

### 3. Marpプレゼンテーション作成
```yaml
marp_template: |
  ---
  marp: true
  theme: explaza
  paginate: true
  header: "**{{presentation_title}}**"
  footer: "© {{current_year}} {{company_name}} | {{project_phase}} フェーズ統合報告"
  
  ---
  
  <!-- _class: lead -->
  # **{{project_id}}**
  ## {{presentation_subtitle}}
  ### {{project_phase}} フェーズ統合報告
  
  **発表者**: {{presenter_name}}  
  **日付**: {{current_date}}  
  **対象**: {{target_audience}}
  
  ---
  
  ## **アジェンダ**
  
  1. **エグゼクティブサマリー** - 企画概要と市場機会
  2. **Senseフェーズ成果** - 市場調査・仮説検証
  3. **Focusフェーズ成果** - 戦略策定・仮説構造化
  4. **検証結果** - 仮説検証の決定的証拠
  5. **重要アップデート** - ビジネスモデル・重要機能
  6. **アクションプラン** - 具体的実行計画
  7. **次のステップ** - Discovery移行準備
  
  ---
  
  ## **エグゼクティブサマリー**
  
  <div class="grid">
  <div>
  
  ### **企画コンセプト**
  **{{value_proposition}}**
  
  - **{{key_benefit_1}}**: {{benefit_1_description}}  
    → {{benefit_1_impact}}
  - **{{key_benefit_2}}**: {{benefit_2_description}}  
    → {{benefit_2_impact}}
  - **{{key_benefit_3}}**: {{benefit_3_description}}  
    → {{benefit_3_impact}}
  
  </div>
  <div>
  
  ### **市場機会**
  - **<span class="success">TAM</span>**: {{tam_value}}
  - **<span class="success">SAM</span>**: {{sam_value}}  
  - **<span class="success">SOM</span>**: {{som_value}}（Year 3）
  
  ### **競合優位性**
  - {{competitive_advantage_1}}
  - {{competitive_advantage_2}}
  - {{competitive_advantage_3}}
  
  </div>
  </div>
  
  ---
  
  ## **Sense フェーズ：徹底的市場調査**
  
  <div class="grid">
  <div>
  
  ### **調査範囲**
  - **競合調査**: {{competitor_count}}社詳細分析
    - {{key_competitors}}
  - **顧客調査**: {{customer_segment_count}}層ターゲット分析
  - **インタビュー**: {{interview_source}}実証
  
  ### **調査期間**
  {{research_period}}
  
  </div>
  <div>
  
  ### **{{key_findings_count}}つの重要発見**
  
  #### **1. {{finding_1_title}}**
  {{finding_1_description}}
  
  #### **2. {{finding_2_title}}**  
  {{finding_2_description}}
  
  #### **3. {{finding_3_title}}**
  {{finding_3_description}}
  
  </div>
  </div>
  
  ---
  
  ## **Focus フェーズ：{{hypothesis_count}}仮説の構造化**
  
  ### **仮説優先順位マトリクス**
  
  <div class="center">
  
  | 優先度 | 仮説ID | 内容 | インパクト | 確実性 |
  |--------|--------|------|-----------|--------|
  | **<span class="danger">最高</span>** | {{high_priority_hypotheses}} | {{high_priority_content}} | {{high_priority_impact}} | {{high_priority_certainty}} |
  | **<span class="warning">高</span>** | {{medium_priority_hypotheses}} | {{medium_priority_content}} | {{medium_priority_impact}} | {{medium_priority_certainty}} |
  | **<span class="success">即座</span>** | {{immediate_hypotheses}} | {{immediate_content}} | {{immediate_impact}} | {{immediate_certainty}} |
  
  </div>
  
  ### **Discovery重点検証（{{critical_hypotheses}}）**
  {{#each critical_hypothesis_list}}
  - **{{this.id}}**: {{this.description}} → {{this.expected_outcome}}
  {{/each}}
  
  ---
  
  ## **ビジネスモデル革新**
  
  <div class="grid">
  <div>
  
  ### **{{pricing_model_name}}**
  
  #### **{{plan_1_name}}**
  - **{{plan_1_price}}**
  - {{plan_1_features}}
  
  #### **{{plan_2_name}}**
  - **{{plan_2_price}}**
  - {{plan_2_features}}
  
  </div>
  <div>
  
  #### **{{plan_3_name}}**
  - **{{plan_3_price}}**
  - {{plan_3_features}}
  
  ### **市場規模インパクト**
  - **従来**: {{old_model_revenue}}（Year 3）
  - **新モデル**: {{new_model_revenue}}（Year 3）
  - **<span class="highlight">成長倍率: {{growth_multiplier}}倍</span>**
  
  </div>
  </div>
  
  ---
  
  ## **{{validation_event_name}}：仮説検証の決定的証拠**
  
  ### **イベント概要（{{event_date}}）**
  - **参加者**: <span class="highlight">{{participant_count}}名</span>（{{participant_roles}}）
  - **内容**: {{event_description}}
  - **会場**: {{event_venue}}
  
  ### **仮説検証結果**
  
  <div class="grid">
  <div>
  
  #### **{{validation_1_title}}**
  > **{{testimonial_1_author}}**  
  > 「{{testimonial_1_quote}}」
  
  > **{{testimonial_2_author}}**  
  > 「{{testimonial_2_quote}}」
  
  </div>
  <div>
  
  #### **{{validation_2_title}}**
  - **{{validation_metric_1}}**: {{validation_result_1}}
  - **{{validation_metric_2}}**: {{validation_result_2}}
  - **{{validation_metric_3}}**: {{validation_result_3}}
  
  #### **{{validation_3_title}}**
  {{validation_3_description}}
  
  </div>
  </div>
  
  ---
  
  ## **今週の{{update_count}}つの戦略的アップデート**
  
  <div class="center">
  
  ### **1. <span class="highlight">{{update_1_title}}</span>**
  {{update_1_description}}
  **{{update_1_impact}}**
  
  ### **2. <span class="highlight">{{update_2_title}}</span>**  
  {{update_2_description}}
  **{{update_2_impact}}**
  
  ### **3. <span class="highlight">{{update_3_title}}</span>**
  {{update_3_description}}
  **{{update_3_impact}}**
  
  </div>
  
  ---
  
  ## **競合優位性：{{differentiation_count}}つの差別化**
  
  <div class="grid">
  <div>
  
  ### **vs {{competitor_1_name}}**
  | {{competitor_1_name}} | {{project_id}} |
  |--------|--------|
  | {{competitor_1_weakness_1}} | <span class="success">{{our_strength_1}}</span> |
  | {{competitor_1_weakness_2}} | <span class="success">{{our_strength_2}}</span> |
  | {{competitor_1_weakness_3}} | <span class="success">{{our_strength_3}}</span> |
  
  </div>
  <div>
  
  ### **vs {{competitor_2_name}}**
  | {{competitor_2_name}} | {{project_id}} |
  |--------|--------|
  | {{competitor_2_weakness_1}} | <span class="success">{{our_advantage_1}}</span> |
  | {{competitor_2_weakness_2}} | <span class="success">{{our_advantage_2}}</span> |
  | {{competitor_2_weakness_3}} | <span class="success">{{our_advantage_3}}</span> |
  
  </div>
  </div>
  
  ### **<span class="highlight">Unfair Advantage</span>**
  {{unfair_advantage_description}}
  
  ---
  
  ## **{{action_period}}アクション：{{action_focus_count}}つの重点施策**
  
  ### **1. <span class="highlight">{{action_1_title}}</span>**
  - **{{action_1_owner}}**: {{action_1_description}}
  - **{{action_1_team}}**: {{action_1_team_tasks}}
  - **{{action_1_timeline}}**: {{action_1_schedule}}
  
  ### **2. <span class="highlight">{{action_2_title}}</span>**
  - **{{action_2_focus}}**: {{action_2_description}}
  - **{{action_2_tools}}**: {{action_2_implementation}}
  - **{{action_2_automation}}**: {{action_2_efficiency}}
  
  ---
  
  ## **{{research_expansion_title}}**
  
  <div class="grid">
  <div>
  
  ### **調査対象企業（{{target_company_count}}社）**
  {{#each target_companies}}
  {{@index}}. **{{this.company}}**: {{this.contact}}（{{this.role}}）
  {{/each}}
  
  ### **調査スケジュール**
  - **{{schedule_phase_1}}**: {{schedule_phase_1_tasks}}
  - **{{schedule_phase_2}}**: {{schedule_phase_2_tasks}}
  - **{{schedule_phase_3}}**: {{schedule_phase_3_tasks}}
  
  </div>
  <div>
  
  ### **インタビュー内容**
  #### **基本版**（{{basic_interview_duration}}分）
  - {{basic_interview_focus_1}}
  - {{basic_interview_focus_2}}
  - {{basic_interview_focus_3}}
  
  #### **{{validation_interview_type}}版**
  - {{validation_interview_focus_1}}
  - {{validation_interview_focus_2}}
  - {{validation_interview_focus_3}}
  
  #### **デモ版**（実装後）
  - {{demo_interview_focus_1}}
  - {{demo_interview_focus_2}}
  
  </div>
  </div>
  
  ---
  
  ## **{{implementation_title}}**
  
  ### **実装スケジュール（{{implementation_duration}}）**
  
  <div class="grid">
  <div>
  
  #### **第1週（{{week1_period}}）**
  - **{{week1_focus_1}}**
  - **{{week1_focus_2}}**
  - **{{week1_focus_3}}**
  
  #### **第2週（{{week2_period}}）**
  - **{{week2_focus_1}}**
  - **{{week2_focus_2}}**
  - **{{week2_focus_3}}**
  
  </div>
  <div>
  
  ### **実装優先度**
  1. **<span class="highlight">{{impl_priority_1}}</span>**: {{impl_priority_1_desc}}
  2. **<span class="highlight">{{impl_priority_2}}</span>**: {{impl_priority_2_desc}}
  3. **<span class="highlight">{{impl_priority_3}}</span>**: {{impl_priority_3_desc}}
  4. **{{impl_priority_4}}**: {{impl_priority_4_desc}}
  
  ### **デモシナリオ**
  - **シナリオ1**: {{demo_scenario_1}}（{{demo_duration_1}}分）
  - **シナリオ2**: {{demo_scenario_2}}（{{demo_duration_2}}分）
  - **シナリオ3**: {{demo_scenario_3}}（{{demo_duration_3}}分）
  
  </div>
  </div>
  
  ---
  
  ## **週別詳細スケジュール**
  
  ### **第1週（{{detailed_week1_period}}）: {{week1_theme}}**
  
  | 日付 | AM | PM |
  |------|----|----|
  | **{{day1_date}}** | {{day1_am}} | {{day1_pm}} |
  | **{{day2_date}}** | {{day2_am}} | {{day2_pm}} |
  | **{{day3_date}}** | {{day3_am}} | {{day3_pm}} |
  | **{{day4_date}}** | {{day4_am}} | {{day4_pm}} |
  | **{{day5_date}}** | {{day5_am}} | {{day5_pm}} |
  
  ### **第2週（{{detailed_week2_period}}）: {{week2_theme}}**
  
  | 日付 | AM | PM |
  |------|----|----|
  | **{{day6_date}}** | {{day6_am}} | {{day6_pm}} |
  | **{{day7_date}}** | {{day7_am}} | {{day7_pm}} |
  | **{{day8_date}}** | {{day8_am}} | {{day8_pm}} |
  | **{{day9_date}}** | {{day9_am}} | {{day9_pm}} |
  | **{{day10_date}}** | {{day10_am}} | {{day10_pm}} |
  
  ---
  
  ## **成果物・KPI**
  
  <div class="grid">
  <div>
  
  ### **定量的成果物**
  - **{{deliverable_1}}**: <span class="highlight">{{deliverable_1_target}}</span>
  - **{{deliverable_2}}**: <span class="highlight">{{deliverable_2_target}}</span>
  - **{{deliverable_3}}**: <span class="highlight">{{deliverable_3_target}}</span>
  - **{{deliverable_4}}**: <span class="highlight">{{deliverable_4_target}}</span>
  
  ### **KPI設定**
  - **{{kpi_1}}**: {{kpi_1_target}}（{{kpi_1_measure}}）
  - **{{kpi_2}}**: {{kpi_2_target}}（{{kpi_2_measure}}）
  - **{{kpi_3}}**: {{kpi_3_target}}（{{kpi_3_measure}}）
  - **{{kpi_4}}**: {{kpi_4_target}}（{{kpi_4_measure}}）
  
  </div>
  <div>
  
  ### **定性的成果物**
  - **{{qualitative_1}}**: {{qualitative_1_description}}
  - **{{qualitative_2}}**: {{qualitative_2_description}}
  - **{{qualitative_3}}**: {{qualitative_3_description}}
  - **{{qualitative_4}}**: {{qualitative_4_description}}
  
  ### **期待される効果**
  - **{{effect_1}}**: {{effect_1_improvement}}（{{effect_1_reason}}）
  - **{{effect_2}}**: {{effect_2_improvement}}（{{effect_2_reason}}）
  - **{{effect_3}}**: {{effect_3_improvement}}（{{effect_3_reason}}）
  
  </div>
  </div>
  
  ---
  
  ## **リスク・対策**
  
  <div class="grid">
  <div>
  
  ### **主要リスク**
  
  #### **<span class="danger">{{risk_1_category}}リスク</span>**
  - **リスク**: {{risk_1_description}}
  - **対策**: {{risk_1_mitigation}}
  
  #### **<span class="warning">{{risk_2_category}}リスク</span>**
  - **リスク**: {{risk_2_description}}
  - **対策**: {{risk_2_mitigation}}
  
  #### **<span class="warning">{{risk_3_category}}リスク</span>**
  - **リスク**: {{risk_3_description}}
  - **対策**: {{risk_3_mitigation}}
  
  </div>
  <div>
  
  ### **対策詳細**
  
  #### **{{mitigation_1_category}}対策**
  - {{mitigation_1_action_1}}
  - {{mitigation_1_action_2}}
  - {{mitigation_1_action_3}}
  
  #### **{{mitigation_2_category}}対策**
  - {{mitigation_2_action_1}}
  - {{mitigation_2_action_2}}
  - {{mitigation_2_action_3}}
  
  #### **{{mitigation_3_category}}対策**
  - {{mitigation_3_action_1}}
  - {{mitigation_3_action_2}}
  - {{mitigation_3_action_3}}
  
  </div>
  </div>
  
  ---
  
  ## **{{next_phase_period}}{{next_phase_name}}移行準備**
  
  <div class="center">
  
  ### **{{current_phase_end}}完了目標**
  - **<span class="success">{{goal_1_category}}</span>**: {{goal_1_description}}
  - **<span class="success">{{goal_2_category}}</span>**: {{goal_2_description}}  
  - **<span class="success">{{goal_3_category}}</span>**: {{goal_3_description}}
  - **<span class="success">{{goal_4_category}}</span>**: {{goal_4_description}}
  
  ### **{{next_phase_period}}{{next_phase_name}}開始内容**
  1. **{{next_phase_activity_1}}**: {{next_phase_activity_1_desc}}
  2. **{{next_phase_activity_2}}**: {{next_phase_activity_2_desc}}
  3. **{{next_phase_activity_3}}**: {{next_phase_activity_3_desc}}
  4. **{{next_phase_activity_4}}**: {{next_phase_activity_4_desc}}
  
  </div>
  
  ---
  
  ## **結論：企画の蓋然性実証完了**
  
  <div class="grid">
  <div>
  
  ### **{{validation_category_1}}の実証**
  - **{{validation_evidence_1}}**: {{validation_evidence_1_desc}}
  - **{{validation_evidence_2}}**: {{validation_evidence_2_desc}}
  - **{{validation_evidence_3}}**: {{validation_evidence_3_desc}}
  
  ### **{{validation_category_2}}の確立**
  - **{{competitive_element_1}}**: {{competitive_element_1_desc}}
  - **{{competitive_element_2}}**: {{competitive_element_2_desc}}
  - **{{competitive_element_3}}**: {{competitive_element_3_desc}}
  
  </div>
  <div>
  
  ### **{{validation_category_3}}の向上**
  - **{{business_element_1}}**: {{business_element_1_desc}}
  - **{{business_element_2}}**: {{business_element_2_desc}}
  - **{{business_element_3}}**: {{business_element_3_desc}}
  
  ### **{{validation_category_4}}の確認**
  - **{{feasibility_element_1}}**: {{feasibility_element_1_desc}}
  - **{{feasibility_element_2}}**: {{feasibility_element_2_desc}}
  - **{{feasibility_element_3}}**: {{feasibility_element_3_desc}}
  
  </div>
  </div>
  
  ---
  
  ## **推奨アクション**
  
  <div class="center large-text">
  
  ### **1. {{recommendation_1_title}}**
  {{recommendation_1_description}}
  
  ### **2. {{recommendation_2_title}}**  
  {{recommendation_2_description}}
  
  ### **3. {{recommendation_3_title}}**
  {{recommendation_3_description}}
  
  ### **4. {{recommendation_4_title}}**
  {{recommendation_4_description}}
  
  </div>
  
  ---
  
  <!-- _class: lead -->
  ## **承認・質疑応答**
  
  <div class="center">
  
  ### **ご質問・ご意見をお聞かせください**
  
  <div style="margin-top: 60px; font-size: 24px;">
  <span class="highlight">企画の蓋然性は実証されました</span><br>
  次は<span class="success">{{next_phase_name}}実行フェーズ</span>への移行です
  </div>
  
  </div>
  
  {{#if include_appendix}}
  ---
  
  ## **付録：参考資料**
  
  ### **作成資料一覧**
  {{#each created_documents}}
  - `{{this.filename}}` - {{this.description}}
  {{/each}}
  
  ### **データソース**
  {{#each data_sources}}
  - {{this.source}}（{{this.count}}）
  {{/each}}
  
  ---
  {{/if}}
  
  **作成**: {{creation_date}} | **作成者**: {{creator_name}}  
  **ステータス**: {{current_status}} | **次**: {{next_phase_name}}開始
```

### 4. 変数設定・データ収集
```yaml
data_collection:
  - name: "sense_data_extraction"
    source: "sense_phase_documents"
    extract:
      - competitor_analysis_results
      - customer_research_findings
      - interview_insights
      - market_validation_data
  
  - name: "focus_data_extraction"
    source: "focus_phase_documents"
    extract:
      - opportunity_hypotheses
      - product_definition
      - market_sizing
      - roadmap_timeline
      - okr_objectives
      - lean_canvas_elements
      - action_plans
  
  - name: "validation_data_extraction"
    source: "reference_materials"
    extract:
      - event_feedback
      - user_testimonials
      - validation_metrics
      - proof_points
```

### 5. ファイル生成・エクスポート
```yaml
file_generation:
  - name: "marp_presentation"
    template: "marp_template"
    output: "{{patterns.flow_date}}/{{project_id}}_presentation_marp.md"
    
  - name: "presentation_guide"
    template: "presentation_guide_template"
    output: "{{patterns.flow_date}}/{{project_id}}_presentation_guide.md"
    
  - name: "export_pdf"
    command: "npx @marp-team/marp-cli {{project_id}}_presentation_marp.md --pdf"
    condition: "export_formats contains 'PDF'"
    
  - name: "export_html"
    command: "npx @marp-team/marp-cli {{project_id}}_presentation_marp.md --html"
    condition: "export_formats contains 'HTML'"
    
  - name: "export_pptx"
    command: "npx @marp-team/marp-cli {{project_id}}_presentation_marp.md --pptx"
    condition: "export_formats contains 'PowerPoint'"
```

## 使用方法

### 基本実行
```bash
# コマンド実行
プレゼン資料作成

# または
社内プレゼン作成 Palma MVP0
```

### 高度な実行
```bash
# 特定の目的で実行
企画説明資料作成 --purpose="投資家ピッチ" --length="詳細"

# 複数フォーマットでエクスポート
Sense Focus プレゼン --export="PDF,HTML,PowerPoint"
```

## 出力ファイル

### 生成されるファイル
1. **`{project_id}_presentation_marp.md`** - Marpプレゼンテーション本体
2. **`{project_id}_presentation_guide.md`** - 使用ガイド・説明書
3. **`{project_id}_presentation_marp.pdf`** - PDF版（選択時）
4. **`{project_id}_presentation_marp.html`** - HTML版（選択時）
5. **`{project_id}_presentation_marp.pptx`** - PowerPoint版（選択時）

### ファイル配置
```
Flow/YYYYMM/YYYY-MM-DD/{project_id}/2_focus/11_プレゼンテーション資料作成/
├── {project_id}_presentation_marp.md
├── {project_id}_presentation_guide.md
├── {project_id}_presentation_marp.pdf
├── {project_id}_presentation_marp.html
└── {project_id}_presentation_marp.pptx
```

## 特徴

### デザイン仕様
- **テーマ**: Explaza専用テーマ
- **絵文字**: 使用しない（プロフェッショナル重視）
- **カラーパレット**: Explazaブランドカラー
- **レイアウト**: グリッドシステム・レスポンシブ対応

### 内容構成
- **17-20スライド**: 標準的なプレゼンテーション長
- **データ統合**: Sense & Focus全成果物を自動統合
- **証拠重視**: 検証データ・フィードバックを重点配置
- **アクション指向**: 具体的な次のステップを明示

### 自動化機能
- **データ抽出**: 既存ドキュメントから自動データ収集
- **変数置換**: テンプレート変数の自動置換
- **複数エクスポート**: PDF・HTML・PowerPoint同時出力
- **ガイド生成**: 使用方法・カスタマイズガイド自動作成

## 注意事項

1. **事前準備**: Sense & Focusフェーズの成果物が完成していること
2. **Marp環境**: `@marp-team/marp-cli`がインストールされていること
3. **テーマファイル**: Explazaテーマが利用可能であること
4. **データ整合性**: 参照するデータファイルの形式が統一されていること

## カスタマイズ

### テンプレート修正
- `marp_template`セクションを編集
- 変数名・構成を調整可能

### スタイル調整
- Explazaテーマファイルを修正
- CSS変数で色・フォント調整

### 出力形式追加
- `file_generation`セクションに新しいエクスポート形式追加
- 対応するコマンドを定義

---

**作成日**: 2025-09-16  
**バージョン**: 1.0  
**対応フェーズ**: Sense & Focus  
**依存関係**: Marp CLI, Explazaテーマ
