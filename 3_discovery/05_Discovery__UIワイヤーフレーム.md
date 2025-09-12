# 05 仮説駆動: UIワイヤーフレーム（AIPMハッカソン）

## 最初に質問（実行前に回答してください）
- 画面数（最大2画面推奨）（必須）
- メイン画面の要素（上から順）（必須）
- 操作フロー（3-5手順）（必須）
- UI/UXの工夫点（必須）
- 参考デザイン（任意）

## 目的
MVP/Releaseの画面構成をストーリーに対応づけて設計します。画面番号とSTを紐づけ、スクリーンフローをYAMLとMermaidで表現し、各スクリーンをDraw.ioで出力します。

## 実行手順
```yaml
- trigger: "(仮説駆動_UIワイヤーフレーム|HypothesisUIWireframe)"
  priority: high
  steps:
    - name: "prefill_from_yaml"
      action: "display"
      content: |
        🔎 参照
        - story_map.yaml / story_map_mermaid.md
        - spec_mvp.yaml（任意）
    
    - name: "show_ui_patterns"
      action: "display"
      content: |
        🎨 UIパターン例
        - ミニマル型（1タスクだけ）
        - リスト型（チェックボックス）
        - カード型（小さなまとまり）
    
    - name: "collect_ui_info"
      action: "ask_questions_with_template"
      template: |
        === UI設計テンプレート（候補は上に表示） ===
        1. 画面数と役割（既定: 1画面）
        → 【あなたの回答】：
        
        2. メイン画面の要素（上から順に）
        既定候補: タブ(朝/昼/夜) / 主要1アクション / 候補3件 / 3枠表示
        → 【あなたの回答】：
        
        3. 操作フロー（3-5手順）
        既定候補: 表示→今やる→中断3択→完了→3枠更新
        → 【あなたの回答】：
        
        4. UI/UXの工夫
        既定候補: 親指リーチ/静音/遅延<300ms/VoiceOver
        → 【あなたの回答】：
        
        5. 参考デザイン（任意）
        → 【あなたの回答】：
        =====================================
    
    - name: "wait_inputs"
      action: "wait_for_all_answers"
    
    # 自動提案（ストーリー連動の画面番号割当とフロー）
    - name: "auto_propose_screen_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/screen_map_proposed.yaml"
      content: |
        screens:
          - id: SC1
            title: メイン
            links_stories: [ST1, ST2]
          - id: SC2
            title: 詳細/完了
            links_stories: [ST3]
        mapping:
          ST1: SC1
          ST2: SC1
          ST3: SC2
        flows:
          MVP:
            - from: SC1
              action: 今やる
              to: SC2
            - from: SC2
              action: 完了
              to: SC1
          Release1:
            - from: SC1
              action: 時間選択
              to: SC2
        numbering_policy:
          - prefix: SC=Screen
    
    - name: "auto_propose_screen_flow_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/screen_flow_proposed_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          %% MVP
          SC1[SC1 メイン] -->|今やる| SC2[SC2 詳細/完了]
          SC2 -->|完了| SC1
          
          %% Release1
          SC1 -->|時間選択| SC2
        ```
    
    - name: "display_proposal"
      action: "display"
      content: |
        ✍️ 画面番号とストーリーの対応、およびMVP/Releaseごとのスクリーンフロー提案を出力しました。修正点を入力してください（空欄は採用）。
    
    - name: "collect_screen_corrections"
      action: "ask_questions_with_template"
      template: |
        === 修正テンプレ（空欄は採用） ===
        1) スクリーン定義（SCxの追加/名称/links_stories）
        →
        2) ST→SCの対応（例: ST2: SC2）
        →
        3) MVPフロー（from/action/toの追加・変更）
        →
        4) Release1フロー（from/action/toの追加・変更）
        →
        =====================================
    
    - name: "confirm_generate"
      action: "confirm"
      message: "提案＋修正内容で確定成果物（screen_map.yaml / screen_flow.yaml / screen_flow_mermaid.md / design/*.yaml / design/*_wire_aa.md / drawio/*.drawio プレース）を生成します。よろしいですか？"
    
    - name: "export_screen_map_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/screen_map.yaml"
      content: |
        screens:
          {{screens_final_yaml}}
        mapping:
          {{mapping_final_yaml}}
        numbering_policy:
          - prefix: SC=Screen
    
    - name: "export_screen_flow_yaml"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/screen_flow.yaml"
      content: |
        flows:
          MVP:
            {{flow_mvp_final_yaml}}
          Release1:
            {{flow_release1_final_yaml}}
    
    - name: "export_screen_flow_mermaid"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/screen_flow_mermaid.md"
      content: |
        ```mermaid
        flowchart LR
          %% MVP
          SC1[SC1 {{sc1_title | default:"メイン"}}] -->|今やる| SC2[SC2 {{sc2_title | default:"詳細/完了"}}]
          SC2 -->|完了| SC1
          
          %% Release1
          SC1 -->|時間選択| SC2
        ```
    
    - name: "export_screen_design_specs_SC1"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/design/SC1_design.yaml"
      content: |
        screen: SC1
        title: メイン
        tokens:
          color_primary: "#3b82f6"
          color_bg: "#f8fafc"
          spacing: 8
          radius: 12
          font_size_title: 20
        layout:
          header: {x: 24, y: 16, w: 512, h: 28, text: "今日の一番"}
          input:  {x: 24, y: 56, w: 360, h: 36, placeholder: "タスクを入力"}
          addBtn: {x: 392, y: 56, w: 96,  h: 36, text: "追加"}
          list:   {x: 24, y: 104, w: 464, h: 320}
          modeBtn:{x: 24, y: 440, w: 160, h: 36, text: "仕事モード"}
        links_stories: [ST1, ST2]
        accessibility:
          focus_order: [input, addBtn, list, modeBtn]
          aria:
            - element: modeBtn
              role: button
              aria-pressed: true|false
    
    - name: "export_screen_design_specs_SC2"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/design/SC2_design.yaml"
      content: |
        screen: SC2
        title: 詳細/完了
        tokens:
          color_primary: "#3b82f6"
          color_bg: "#ffffff"
          spacing: 8
          radius: 12
          font_size_title: 18
        layout:
          title:   {x: 24, y: 16, w: 512, h: 24, text: "タスク詳細"}
          detail:  {x: 24, y: 56, w: 464, h: 200}
          doneBtn: {x: 24, y: 264, w: 120, h: 36, text: "完了"}
          backBtn: {x: 160, y: 264, w: 120, h: 36, text: "戻る"}
        links_stories: [ST3]
        accessibility:
          focus_order: [doneBtn, backBtn]
          aria: []
    
    - name: "export_screen_design_aa_SC1"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/design/SC1_wire_aa.md"
      content: |
        ```
        +--------------------------------------------------+
        | 今日の一番                                       |
        +------------------------+-----------+------------+
        | [ タスクを入力........ ] | [追加]    | (仕事モード) |
        +------------------------+-----------+------------+
        | • 候補タスク1                                      |
        | • 候補タスク2                                      |
        | • 候補タスク3                                      |
        +--------------------------------------------------+
        ```
    
    - name: "export_screen_design_aa_SC2"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/design/SC2_wire_aa.md"
      content: |
        ```
        +----------------------------------------------+
        | タスク詳細                                   |
        +----------------------------------------------+
        | [内容.....................................]  |
        |                                              |
        | [ 完了 ]    [ 戻る ]                         |
        +----------------------------------------------+
        ```
    
    - name: "generate_drawio_placeholder"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/drawio/SCREENS_PLACEHOLDER.txt"
      content: |
        各スクリーンのDraw.ioファイル（例: SC1.drawio, SC2.drawio）をこのディレクトリに配置してください。
        別コマンド「設計_Drawioスクリーン生成」で自動生成できます。
    
    - name: "generate_wireframe_md"
      action: "create_markdown_file"
      path: "Flow/{{today}}/{{flow_dir}}/05_仮説駆動__UIワイヤーフレーム/ui_wireframe_todo.md"
      content: |
        # アプリ UIワイヤーフレーム
        
        ## 画面一覧とストーリー対応（screen_map.yaml）
        - SC1: ST1, ST2
        - SC2: ST3
        
        ## スクリーンフロー（MVP/Release）
        - MVP/Releaseの画面遷移図は `screen_flow_mermaid.md` を参照（Mermaidコードフェンス付きで出力済み）
        
        ## Draw.io
        - `drawio/SC1.drawio`, `drawio/SC2.drawio` は別コマンドで生成可
    
    - name: "notify_completion"
      action: "display"
      content: |
        ✅ 生成しました：
        - 05_仮説駆動__UIワイヤーフレーム/screen_map.yaml / screen_flow.yaml / screen_flow_mermaid.md
        - 05_仮説駆動__UIワイヤーフレーム/design/SC1_design.yaml / design/SC2_design.yaml
        - 05_仮説駆動__UIワイヤーフレーム/design/SC1_wire_aa.md / design/SC2_wire_aa.md
        - 05_仮説駆動__UIワイヤーフレーム/drawio/ 配下のプレースホルダ
        - 05_仮説駆動__UIワイヤーフレーム/ui_wireframe_todo.md
```

## 次のコマンド
→ `設計_Drawioスクリーン生成` で `SC1.drawio`/`SC2.drawio` を自動生成
→ その後 `仮説駆動_アプリ骨子生成` でスターターコード生成
