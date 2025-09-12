# Focus: 市場規模推定（TAM/SAM/SOM + Sheets）

## 目的
定義したプロダクト案の市場性を素早く見積もり、ビジネスインパクトの仮説を置きます。

## 実行手順（Rules Steps）
```yaml
- trigger: "(focus_市場規模推定|Market Size|TAM|SAM|SOM)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}", "{{read_files(find_files(patterns=['Flow/**/*.md','Flow/**/*.mdx','**/submit_*/README.md']))}}"]
      instructions: |
        スレッド内の文脈から、以下の推定値を返してください（なければ空）。
        keys: project_name, region, company_threshold, enterprise_companies, seats_per_company, seat_price_month, platform_fee_year, adoption_rates, scenario_seats
        出力はJSONのオブジェクト1件のみ。
      store_as: "prefill"
    - name: "collect_market_inputs"
      action: "ask_questions"
      questions:
        - key: "project_name"
          question: "調査対象のプロジェクト名を入力してください"
          required: true
          default: "{{prefill.project_name}}"
        - key: "region"
          question: "対象地域を入力してください（例：日本、北米、グローバル）"
          required: true
          default: "{{prefill.region}}"
        - key: "company_threshold"
          question: "企業規模の閾値（例：従業者500名以上）"
          required: false
          default: "{{prefill.company_threshold}}"
        - key: "enterprise_companies"
          question: "該当企業数（分かる範囲で、未入力可）"
          required: false
          default: "{{prefill.enterprise_companies}}"
        - key: "seats_per_company"
          question: "初期対象席数/社（例：20）"
          required: false
          default: 20
        - key: "seat_price_month"
          question: "席課金（月額/席）（例：5000）"
          required: false
          default: 5000
        - key: "platform_fee_year"
          question: "プラットフォーム年額（SSO/監査/ガバナンス等）（例：1000000）"
          required: false
          default: 1000000
        - key: "adoption_rates"
          question: "シナリオ別採用率（保守/標準/強気をカンマ区切りで、例：0.03,0.07,0.15）"
          required: false
          default: "0.03,0.07,0.15"
        - key: "scenario_seats"
          question: "シナリオ別席数/社（保守/標準/強気をカンマ区切りで、例：8,15,30）"
          required: false
          default: "8,15,30"
      store_as: "ms_params"

    - name: "confirm_market_calc"
      action: "confirm"
      message: |
        以下の設定で市場規模推定を実行します：
        プロジェクト: {{ms_params.project_name}}
        地域: {{ms_params.region}}
        企業閾値: {{ms_params.company_threshold}}
        企業数(仮): {{ms_params.enterprise_companies}}
        席/社(初期): {{ms_params.seats_per_company}}
        席課金(月/席): {{ms_params.seat_price_month}}
        PF年額: {{ms_params.platform_fee_year}}
        採用率(保守/標準/強気): {{ms_params.adoption_rates}}
        席数/社(保守/標準/強気): {{ms_params.scenario_seats}}

    - name: "create_market_size_draft"
      action: "edit_file"
      path: "{{patterns.flow_date}}/focus_market_size_estimation.md"
      message: "TAM/SAM/SOMの枠組みと式をFlowに出力します"
      content: |
        ---
        doc_targets: [market_size_estimation, pricing, roadmap]
        importance: 5
        program: {{program_id}}
        project: {{ms_params.project_name}}
        date: {{today}}
        ---

        # 市場規模推定（TAM/SAM/SOM） - {{ms_params.project_name}}

        ## 入力パラメータ
        - 地域: {{ms_params.region}}
        - 企業閾値: {{ms_params.company_threshold}}
        - 企業数(仮): {{ms_params.enterprise_companies}}
        - 席/社(初期): {{ms_params.seats_per_company}}
        - 席課金(月/席): {{ms_params.seat_price_month}}
        - PF年額: {{ms_params.platform_fee_year}}
        - 採用率(保守,標準,強気): {{ms_params.adoption_rates}}
        - 席数/社(保守,標準,強気): {{ms_params.scenario_seats}}

        ## 式
        - 年間/社 = (席/社 × 席課金 × 12) + PF年額
        - TAM(ウェッジ) = 企業数 × 年間/社（初期席数）
        - TAM(拡張例) = 企業数 × ((200席 × 席課金 × 12) + PF年額)
        - SAM = TAM(ウェッジ) × 到達率（例: 40%）
        - SOM(シナリオ別) = 採用社数 × 年間/社（各シナリオ席数）

        ※ Sheets用CSVと操作手順は同フォルダ内 `gsheets/` に出力します。

    - name: "create_gsheets_folder"
      action: "execute_shell"
      command: "mkdir -p {{patterns.flow_date}}/gsheets"

    - name: "create_params_csv"
      action: "edit_file"
      path: "{{patterns.flow_date}}/gsheets/market_size_params.csv"
      content: |
        param,value
        region,{{ms_params.region}}
        enterprise_companies,{{ms_params.enterprise_companies}}
        seats_per_company,{{ms_params.seats_per_company}}
        seat_price_month,{{ms_params.seat_price_month}}
        platform_fee_year,{{ms_params.platform_fee_year}}
        months,12

    - name: "create_scenarios_csv"
      action: "edit_file"
      path: "{{patterns.flow_date}}/gsheets/market_size_scenarios.csv"
      content: |
        scenario,adoption_rate,seats_per_company
        conservative,{{ms_params.adoption_rates | split: ',' | first }},{{ms_params.scenario_seats | split: ',' | first }}
        base,{{ms_params.adoption_rates | split: ',' | nth: 1 }},{{ms_params.scenario_seats | split: ',' | nth: 1 }}
        bull,{{ms_params.adoption_rates | split: ',' | last }},{{ms_params.scenario_seats | split: ',' | last }}

    - name: "create_sheet_instructions"
      action: "edit_file"
      path: "{{patterns.flow_date}}/gsheets/PATTERN_A_SHEETS_INSTRUCTIONS.md"
      content: |
        # Pattern A: Googleスプレッドシート再計算（手動）
        1) `market_size_params.csv` と `market_size_scenarios.csv` を新しいシートとしてインポート（`params` / `scenarios`）
        2) `calc` シートで以下を設定：
           - A1: enterprise_companies / B1: =INDEX(params!B:B, MATCH("enterprise_companies", params!A:A, 0))
           - A2: seat_price_month / B2: =INDEX(params!B:B, MATCH("seat_price_month", params!A:A, 0))
           - A3: platform_fee_year / B3: =INDEX(params!B:B, MATCH("platform_fee_year", params!A:A, 0))
           - A4: months / B4: 12
        3) `scenarios` に列追加：
           - D: enterprise_companies (=calc!B1)
           - E: annual_per_company (=C×calc!B2×calc!B4 + calc!B3)
           - F: adopted_companies (=ROUND(D×B))
           - G: annual_total (=F×E)
           - H: annual_total_million_yen (=G/1000000)
        4) `summary` シート：
           - A1: TAM_wedge / B1: =calc!B1 * ({{ms_params.seats_per_company}}*calc!B2*calc!B4 + calc!B3)
           - A2: TAM_expanded / B2: =calc!B1 * (200*calc!B2*calc!B4 + calc!B3)
           - A3: SAM(40%) / B3: =B1*0.4

    - name: "create_apps_script"
      action: "edit_file"
      path: "{{patterns.flow_date}}/gsheets/PATTERN_B_APPS_SCRIPT.md"
      content: |
        # Pattern B: Apps Script自動生成
        ```javascript
        function setupMarketSheet() {
          const ss = SpreadsheetApp.getActiveSpreadsheet();
          ['params','scenarios','calc','summary'].forEach(n=>{const s=ss.getSheetByName(n); if(s) ss.deleteSheet(s); ss.insertSheet(n)});
          const params = ss.getSheetByName('params');
          const p = [
            ['param','value'],
            ['enterprise_companies', {{ms_params.enterprise_companies}}],
            ['seat_price_month', {{ms_params.seat_price_month}}],
            ['platform_fee_year', {{ms_params.platform_fee_year}}],
            ['months', 12]
          ];
          params.getRange(1,1,p.length,p[0].length).setValues(p);
          const scenarios = ss.getSheetByName('scenarios');
          const s = [
            ['scenario','adoption_rate','seats_per_company'],
            ['conservative', {{ms_params.adoption_rates | split: ',' | first }}, {{ms_params.scenario_seats | split: ',' | first }}],
            ['base', {{ms_params.adoption_rates | split: ',' | nth: 1 }}, {{ms_params.scenario_seats | split: ',' | nth: 1 }}],
            ['bull', {{ms_params.adoption_rates | split: ',' | last }}, {{ms_params.scenario_seats | split: ',' | last }}]
          ];
          scenarios.getRange(1,1,s.length,s[0].length).setValues(s);
          const calc = ss.getSheetByName('calc');
          calc.getRange('A1').setValue('enterprise_companies');
          calc.getRange('B1').setFormula('=INDEX(params!B:B, MATCH("enterprise_companies", params!A:A, 0))');
          calc.getRange('A2').setValue('seat_price_month');
          calc.getRange('B2').setFormula('=INDEX(params!B:B, MATCH("seat_price_month", params!A:A, 0))');
          calc.getRange('A3').setValue('platform_fee_year');
          calc.getRange('B3').setFormula('=INDEX(params!B:B, MATCH("platform_fee_year", params!A:A, 0))');
          calc.getRange('A4').setValue('months');
          calc.getRange('B4').setValue(12);
          const sc = ss.getSheetByName('scenarios');
          sc.getRange('D1').setValue('enterprise_companies');
          sc.getRange('E1').setValue('annual_per_company');
          sc.getRange('F1').setValue('adopted_companies');
          sc.getRange('G1').setValue('annual_total');
          sc.getRange('H1').setValue('annual_total_million_yen');
          sc.getRange('D2').setFormula('=calc!B1');
          sc.getRange('E2').setFormula('=scenarios!C2*calc!B2*calc!B4 + calc!B3');
          sc.getRange('F2').setFormula('=ROUND(scenarios!D2*scenarios!B2)');
          sc.getRange('G2').setFormula('=scenarios!F2*scenarios!E2');
          sc.getRange('H2').setFormula('=G2/1000000');
          sc.getRange('D2:H2').copyTo(sc.getRange('D3:H4'));
          const summary = ss.getSheetByName('summary');
          summary.getRange('A1').setValue('TAM_wedge');
          summary.getRange('B1').setFormula('=calc!B1 * ({{ms_params.seats_per_company}}*calc!B2*calc!B4 + calc!B3)');
          summary.getRange('A2').setValue('TAM_expanded');
          summary.getRange('B2').setFormula('=calc!B1 * (200*calc!B2*calc!B4 + calc!B3)');
          summary.getRange('A3').setValue('SAM(40%)');
          summary.getRange('B3').setFormula('=B1*0.4');
        }
        ```

    - name: "validate_assumptions_web"
      action: "web_search"
      search_term: "e-Stat 従業者規模別 企業数 500人以上 / Microsoft Copilot 価格 円 / Google Workspace Gemini Enterprise 価格 / ChatGPT Team 価格"
      explanation: "企業数と価格相場の一次・公式ソース確認"
      store_as: "validation_sources"

    - name: "create_validation_memo"
      action: "edit_file"
      path: "{{patterns.flow_date}}/assumptions_validation.md"
      content: |
        ---
        doc_targets: [market_size_estimation, pricing]
        importance: 5
        program: {{program_id}}
        project: {{ms_params.project_name}}
        date: {{today}}
        ---
        # 仮置き数値の妥当性検証メモ
        - 企業数（{{ms_params.company_threshold}}）: e-Stat 経済センサスの該当表で置換推奨
        - 席課金の参考: Copilot/Gemini/ChatGPT Team 公式価格（$20〜$30/user/mo 目安）
        - PF年額の参考: SSO/監査を含むEnterprise最小コミットの一般的レンジ（0.8〜1.5百万円/年）
        - 推奨: 席課金 4,000〜6,000円/月、PF年額 0.8〜1.5百万円/年、席数 10〜30席/社
        - 参考リンク: {{validation_sources}}

    - name: "notify_output"
      action: "notify"
      message: |
        ✅ 市場規模推定キットを作成しました：
        - ドラフト: {{patterns.flow_date}}/focus_market_size_estimation.md
        - Sheets用CSV/手順: {{patterns.flow_date}}/gsheets/
        - 仮置き検証メモ: {{patterns.flow_date}}/assumptions_validation.md
```

## 次に実行
- `03_focus__ラフロードマップ作成`
- `04_focus__OKR作成`


