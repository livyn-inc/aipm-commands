# 14 GoogleDrive提出（AIPMハッカソン）

## 目的
GitHubアカウントを持たない参加者向けに、Stock側の提出フォルダをZIP圧縮して、指定のGoogleDriveフォルダにアップロードするための手順を提供します。

## 事前条件
- 成果物パッケージング（`@成果物パッケージング`）を実行済みであること
- Googleアカウントでログイン可能であること
- 提出用GoogleDriveフォルダへのアクセス権限があること

## 実行手順
```yaml
- trigger: "(提出_GoogleDrive提出|SubmitGoogleDrive)"
  priority: high
  steps:
    - name: "infer_defaults_from_thread"
      action: "analyze"
      data: ["{{thread_messages}}","{{read_files(find_files(patterns:['Stock/**/submissions/**/README_submission.md','**/project_init.yaml']))}}"]
      instructions: |
        Program/Project/submit_folder/participant_name の初期値候補を抽出し、displayで提示してください。
      store_as: "auto_drive_submit_seed"
    
    - name: "prefill_drive_submit_seed"
      action: "display"
      content: |
        🔎 自動抽出
        {{auto_drive_submit_seed}}
    
    - name: "collect_submit_inputs"
      action: "ask_questions_with_template"
      template: |
        === GoogleDrive提出メタ ===
        1. Program（既定: AIPM_Hackathon）
        →
        2. Project（アプリ名）
        →
        3. 提出フォルダ名（例: submit_{{today}}）
        →
        4. 参加者名（ZIPファイル名に使用。例: yamada_taro）
        →
        5. 提出者メールアドレス（GoogleDriveアクセス用）
        →
        6. 追加メモ（任意。README_submission.mdに追記）
        →
        =====================================
    
    - name: "wait_submit"
      action: "wait_for_all_answers"
    
    # 成果物の事前確認
    - name: "check_submission_folder"
      action: "execute_shell"
      command: |
        if [ -d "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}" ]; then
          echo "✅ 提出フォルダが見つかりました"
          ls -la "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}"
        else
          echo "❌ 提出フォルダが見つかりません。先に @成果物パッケージング を実行してください"
          exit 1
        fi
    
    - name: "confirm_zip_creation"
      action: "confirm"
      message: "提出フォルダをZIP圧縮して、GoogleDriveアップロード用に準備します。よろしいですか？"
    
    # 追加メモがある場合はREADME_submission.mdに追記
    - name: "update_submission_readme"
      action: "execute_shell"
      command: |
        if [ -n "{{additional_memo}}" ]; then
          echo "" >> "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}/README_submission.md"
          echo "## 追加メモ（GoogleDrive提出時）" >> "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}/README_submission.md"
          echo "{{additional_memo}}" >> "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}/README_submission.md"
          echo "- 提出者: {{participant_name}}" >> "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}/README_submission.md"
          echo "- 提出者メール: {{submitter_email}}" >> "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}/README_submission.md"
          echo "- 提出日時: {{today}} $(date '+%H:%M:%S')" >> "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{submit_folder}}/README_submission.md"
        fi
    
    # ZIP圧縮（参加者名_プロジェクト名_日付.zip形式）
    - name: "create_zip"
      action: "execute_shell"
      command: |
        cd "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions" && \
        ZIP_NAME="{{participant_name}}_{{project_id}}_{{today}}.zip" && \
        zip -r "$ZIP_NAME" "{{submit_folder}}" && \
        echo "✅ ZIP作成完了: $ZIP_NAME" && \
        ls -lh "$ZIP_NAME"
    
    # ZIP情報の表示
    - name: "show_zip_info"
      action: "execute_shell"
      command: |
        cd "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions" && \
        ZIP_NAME="{{participant_name}}_{{project_id}}_{{today}}.zip" && \
        echo "📦 ZIP詳細情報:" && \
        unzip -l "$ZIP_NAME" | head -20
    
    - name: "show_upload_instructions"
      action: "display"
      content: |
        📋 **GoogleDriveアップロード手順**
        
        **1. ZIPファイルの場所**:
        ```
        Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{participant_name}}_{{project_id}}_{{today}}.zip
        ```
        
        **2. アップロード先**:
        🔗 **GoogleDriveフォルダ**: https://drive.google.com/
        
        **3. アップロード手順**:
        
        **📱 ブラウザ経由（推奨）**:
        1. 上記GoogleDriveリンクをクリックして開く
        2. Googleアカウント（{{submitter_email}}）でログイン
        3. 「新規」→「ファイルのアップロード」をクリック
        4. 上記ZIPファイルを選択してアップロード
        5. アップロード完了後、ファイル名を確認
        
        **🖥️ Google Drive デスクトップ同期（代替手段）**:
        1. Google Drive デスクトップアプリをインストール
        2. 同期フォルダにZIPファイルをコピー
        3. 自動同期を待つ
        
        **4. アップロード完了の確認**:
        - GoogleDriveフォルダで自分のZIPファイルが表示されることを確認
        - ファイル名: `{{participant_name}}_{{project_id}}_{{today}}.zip`
        
        **5. 提出完了報告（推奨）**:
        - 運営者にSlack/メールで提出完了を報告
        - ZIPファイル名を明記: `{{participant_name}}_{{project_id}}_{{today}}.zip`
    
    - name: "create_upload_checklist"
      action: "create_markdown_file"
      path: "Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/GoogleDrive_upload_checklist.md"
      content: |
        # GoogleDriveアップロード チェックリスト
        
        ## 基本情報
        - **参加者名**: {{participant_name}}
        - **プロジェクト**: {{project_id}}
        - **ZIPファイル名**: {{participant_name}}_{{project_id}}_{{today}}.zip
        - **提出者メール**: {{submitter_email}}
        - **作成日時**: {{today}} $(date '+%H:%M:%S')
        
        ## アップロード手順チェック
        
        ### 事前準備
        - [ ] Googleアカウント（{{submitter_email}}）でログイン済み
        - [ ] GoogleDriveフォルダへのアクセス権限確認済み
        - [ ] ZIPファイルの存在確認済み
        
        ### アップロード実行
        - [ ] GoogleDriveフォルダを開く: https://drive.google.com/drive/folders/
        - [ ] 「新規」→「ファイルのアップロード」をクリック
        - [ ] ZIPファイル（{{participant_name}}_{{project_id}}_{{today}}.zip）を選択
        - [ ] アップロード進行状況を確認
        - [ ] アップロード完了メッセージを確認
        
        ### 提出完了確認
        - [ ] GoogleDriveフォルダで自分のZIPファイルを確認
        - [ ] ファイル名が正しいことを確認
        - [ ] ファイルサイズが妥当であることを確認
        - [ ] 運営者への提出完了報告（Slack/メール）
        
        ## トラブルシューティング
        
        **アクセス権限エラーの場合**:
        - 運営者に{{submitter_email}}でのアクセス権限付与を依頼
        
        **アップロードエラーの場合**:
        - ファイルサイズを確認（大きすぎる場合は圧縮レベルを上げる）
        - ネットワーク接続を確認
        - ブラウザを再起動して再試行
        
        **ファイルが見つからない場合**:
        - ZIPファイルのパスを再確認
        - 先に @成果物パッケージング を実行したか確認
        
        ## 連絡先
        - 技術的な問題: 運営者まで
        - アクセス権限の問題: 運営者まで
    
    - name: "open_drive_folder"
      action: "execute_shell"
      command: |
        echo "🌐 GoogleDriveフォルダを開いています..."
        if command -v open >/dev/null 2>&1; then
          open "https://drive.google.com/drive/folders/"
        elif command -v xdg-open >/dev/null 2>&1; then
          xdg-open "https://drive.google.com/drive/folders/"
        elif command -v start >/dev/null 2>&1; then
          start "https://drive.google.com/drive/folders/"
        else
          echo "手動でブラウザを開き、以下URLにアクセスしてください:"
          echo "https://drive.google.com/drive/folders/
        fi
    
    - name: "notify_completion"
      action: "display"
      content: |
        ✅ **GoogleDrive提出準備が完了しました**
        
        📦 **作成されたファイル**:
        - **ZIPファイル**: `Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/{{participant_name}}_{{project_id}}_{{today}}.zip`
        - **チェックリスト**: `Stock/programs/{{program_id}}/projects/{{project_id}}/submissions/GoogleDrive_upload_checklist.md`
        
        🔗 **次のステップ**:
        1. 自動で開いたGoogleDriveフォルダ（または手動でアクセス）
        2. ZIPファイルをアップロード
        3. チェックリストに従って提出完了を確認
        4. 運営者に提出完了を報告
        
        📋 **提出情報**:
        - **参加者**: {{participant_name}}
        - **プロジェクト**: {{project_id}}
        - **ZIPファイル名**: {{participant_name}}_{{project_id}}_{{today}}.zip
        - **提出者メール**: {{submitter_email}}
        
        🎯 **重要**: アップロード後は必ずGoogleDriveフォルダで自分のファイルが正しく表示されることを確認してください。
```

## 備考
- GitHubアカウントを持たない参加者向けの代替提出方法
- GoogleDriveフォルダへのアクセス権限が必要（運営者に事前確認）
- ZIPファイル名は `参加者名_プロジェクト名_日付.zip` 形式で統一
- アップロード完了後は運営者への報告を推奨

## 関連コマンド
- 前段階: `@成果物パッケージング` で提出フォルダを準備
- 代替手段: `@GitHub提出` でGitHub経由提出



