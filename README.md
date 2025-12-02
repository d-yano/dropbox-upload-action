# Dropbox Upload Action
GitHub Actions workflow から Dropbox へファイルを簡単にアップロードするためのアクションです。
`actions/upload-artifact` ライクな使い心地で、バックアップや成果物の共有に使えます。

## 特徴
本アクションは、GitHub Actions ワークフローから Dropbox へのファイルアップロードをサポートします。
また、Dropbox APIの仕様に従って150MBを超える大容量ファイルを自動的に検知し、分割アップロード（Chunked Upload）転送を行います。
ファイルの指定はワイルドカードが利用でき、複数のログや成果物をまとめて指定が可能です。
また、オプション機能を使って自動的に tar.gz 形式に圧縮してからアップロードしたりすることも可能です。

## 使用方法
### 基本的な使い方
```yaml
- uses: d-yano/dropbox-upload-action@v1
  with:
    dropbox_token: ${{ secrets.DROPBOX_ACCESS_TOKEN }}
    files: |
      build/app-release.apk
      logs/*.log
      dist/*.zip
````

### オプション指定をした場合
```yaml
- uses: d-yano/dropbox-upload-action@v1
  with:
    dropbox_token: ${{ secrets.DROPBOX_ACCESS_TOKEN }}
    files: |
      dist/
      docs/**/*.pdf
      logs/error.log
    # アップロード前に tar.gz にまとめて圧縮する
    archive: true
    archive_name: release-package.tar.gz
    # Dropbox上の保存先を自由に指定 (GitHubの変数も使用可能)
    remote_path: '/MyProject/NightlyBuilds/${{ github.run_number }}'
```

## Inputs (入力パラメータ)
| 入力名 | 必須 | デフォルト値 | 説明 |
| :--- | :---: | :--- | :--- |
| `dropbox_token` | **Yes** | - | Dropbox API Access Token (Secretsに保存してください) |
| `files` | **Yes** | - | アップロードするファイルパス。ワイルドカードや複数行指定が可能。 |
| `archive` | No | `false` | `true` にすると、対象ファイルをtar.gzにまとめてからアップロードします。 |
| `archive_name` | No | `artifact.tar.gz` | `archive: true` の時の圧縮ファイル名。 |
| `remote_path` | No | (自動生成) | Dropbox上の保存先ディレクトリ。 |

## Dropbox Token の取得方法
1.  [Dropbox App Console](https://www.dropbox.com/developers/apps) にアクセスしてアプリを作成します ("App folder" 権限推奨)。
2.  "Permissions" タブで `files.content.write` にチェックを入れます。
3.  "Generated access token" の Generate ボタンを押してトークンを取得します。
4.  GitHubリポジトリの `Settings` \> `Secrets and variables` \> `Actions` に `DROPBOX_ACCESS_TOKEN` として登録します。

## License
- MIT License
