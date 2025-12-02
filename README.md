# Dropbox Upload Action
GitHub Actions workflow から Dropbox へファイルを簡単にアップロードするためのアクションです。  
`actions/upload-artifact` ライクな使い心地で、バックアップや成果物の共有に使えます。

## 特徴
リフレッシュトークン（Refresh Token）を用いた永続的な認証を標準サポートしています。  
これにより、アクセストークンの有効期限（4時間）を気にすることなく、長期にわたるワークフローの完全自動化を実現します。  
  
機能面では、150MBを超える大容量ファイルを自動検知して分割アップロード（Chunked Upload）を行います。  
また、ワイルドカード（Glob）を使用した複数ファイルの指定や、アップロード前の tar.gz 形式への  
自動圧縮オプションも備えており、ログファイルやビルド成果物を柔軟かつ確実に管理することができます。

## 使用方法
### 1. 推奨設定（リフレッシュトークンを使用）
一度設定すればトークンの有効期限を気にせず永続的に利用できます。

```yaml
- uses: d-yano/dropbox-upload-action@v1
  with:
    # Secretsに登録した認証情報を渡す
    app_key: ${{ secrets.DROPBOX_APP_KEY }}
    app_secret: ${{ secrets.DROPBOX_APP_SECRET }}
    refresh_token: ${{ secrets.DROPBOX_REFRESH_TOKEN }}
    
    files: |
      build/app-release.apk
      logs/*.log
```

### 2. オプション指定（アーカイブと保存先指定）
```yaml
- uses: d-yano/dropbox-upload-action@v1
  with:
    app_key: ${{ secrets.DROPBOX_APP_KEY }}
    app_secret: ${{ secrets.DROPBOX_APP_SECRET }}
    refresh_token: ${{ secrets.DROPBOX_REFRESH_TOKEN }}
    
    files: |
      dist/
      docs/**/*.pdf
    # アップロード前に tar.gz にまとめて圧縮する
    archive: true
    archive_name: release-package.tar.gz
    # Dropbox上の保存先を自由に指定 (GitHubの変数も使用可能)
    remote_path: '/MyProject/NightlyBuilds/${{ github.run_number }}'
```

### 3. 旧来の使い方（一時的なアクセストークン）
手動で発行したトークンを使用します（約4時間で無効になります）。一時的なテスト用途向けです。

```yaml
- uses: d-yano/dropbox-upload-action@v1
  with:
    dropbox_token: ${{ secrets.DROPBOX_ACCESS_TOKEN }}
    files: dist/*.zip
```

## Inputs (入力パラメータ)
| 入力名 | 必須 | 説明 |
| :--- | :---: | :--- |
| `files` | **Yes** | アップロードするファイルパス。ワイルドカードや複数行指定が可能。 |
| `app_key` | *Cond* | Dropbox App Key (リフレッシュトークン使用時に必須)。 |
| `app_secret` | *Cond* | Dropbox App Secret (リフレッシュトークン使用時に必須)。 |
| `refresh_token` | *Cond* | Dropbox Refresh Token (永続化のために推奨)。 |
| `dropbox_token` | *Cond* | 一時的な Access Token (上記3つを使わない場合に必須)。 |
| `archive` | No | `true` にすると、対象ファイルをtar.gzにまとめてからアップロードします (Default: `false`)。 |
| `archive_name` | No | `archive: true` の時の圧縮ファイル名 (Default: `artifact.tar.gz`)。 |
| `remote_path` | No | Dropbox上の保存先ディレクトリ。<br>未指定時は `/ci-artifacts/<Repo>/<Branch>/<RunID>` に保存されます。 |

## 設定手順 (永続化のための Refresh Token 取得)
Dropbox の仕様変更により、通常のアクセストークンは短時間(4時間程度)で失効します。  
以下の手順で **Refresh Token** を取得し、GitHub Secrets に設定してください。  

### Step 1: アプリの作成とキーの取得
1.  [Dropbox App Console](https://www.dropbox.com/developers/apps) にアクセスし、「Create app」をクリックします。
      - **Scoped access** を選択
      - **App folder**（専用フォルダのみ）または **Full Dropbox** を選択
      - アプリ名を入力して作成
2.  **Permissions** タブを開き、`files.content.write` と `files.content.read` にチェックを入れ、下部の **Submit** を押します。
3.  **Settings** タブに戻り、`App key` と `App secret` を控えます。

### Step 2: 認可コード (Access Code) の取得
ブラウザで以下のURLにアクセスします（`<APP_KEY>` を Step 1 の App key に置き換えてください）。  

```text
https://www.dropbox.com/oauth2/authorize?client_id=<APP_KEY>&token_access_type=offline&response_type=code
```

表示された画面で「許可」を押すと、短いコード（Access Code）が表示されるのでコピーします。

> **⚠️ 重要:** Access Code の有効期限は**非常に短く（約5分程度）**、一度しか使用できません。  
> コピーしたら**有効期限内に**次の Step 3 を実行してください。

### Step 3: Refresh Token の発行
PCのターミナル（コマンドプロンプト）で以下のコマンドを実行します。  
（`<...>` の部分をそれぞれの値に置き換えてください）

```bash
curl https://api.dropbox.com/oauth2/token \
    -d code=<Step2でコピーしたコード> \
    -d grant_type=authorization_code \
    -d client_id=<APP_KEY> \
    -d client_secret=<APP_SECRET>
```

返ってきた JSON の中に含まれる `"refresh_token": "**************"` という文字列がリフレッシュトークンです。

### Step 4: GitHub Secrets への登録
GitHub リポジトリの `Settings` \> `Secrets and variables` \> `Actions` に以下を登録します。
  - `DROPBOX_APP_KEY`: App key
  - `DROPBOX_APP_SECRET`: App secret
  - `DROPBOX_REFRESH_TOKEN`: 取得した Refresh Token

これで設定完了です。以降、アクションが自動的にトークンを更新してアップロードを行います。

## License
  - MIT License