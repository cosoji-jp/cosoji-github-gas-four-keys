# github-gas-four-keys
## 概要
GoogleSpreadsheetでGitHubレポジトリのFourKeysの計測/可視化を行うツールです.
以下の特徴があります.
- PullRequestのラベルを利用してFourKeys計測を行うため、導入が容易
- GASを用いて自動化されるため、インフラコストが不要

![出力例](img/example.png)

## インストレーション
### GitHubアクセストークンの準備
https://github.com/settings/tokens/new からrepoにチェックを入れアクセストークンを発行します(トークンは後ほど使います).

### GAS APIの有効化
https://script.google.com/home/usersettings からGoogle Apps Script APIをオンにします.

### ClaspによるSpreadsheetとGASの作成

1. 以下のシェルを実行. GoogleDrive上に `Github-gas-four-keys` というスプレッドシートが作成されます.
```sh
npm install @google/clasp -g

git clone https://github.com/cosoji-jp/github-gas-four-keys.git
cd github-gas-four-keys
npm init -y

# ブラウザでGoogleアカウントのログインが求められます.
clasp login
# ログインしたアカウントのGoogleDriveのマイドライブのルートにSpreadsheetが作成されます.
clasp create --type sheets
clasp push
```

2. GoogleDrive上に `Github-gas-four-keys` という名前のスプレッドシートが作られます.
3. 作成されたスプレッドシートの[Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)を開きます.
4. 以下の[ScriptProperty](https://developers.google.com/apps-script/guides/properties?hl=ja#add_script_properties)を設定します.

|プロパティ|値|
|----|----|
|GITHUB_API_TOKEN|GitHubアクセストークンの準備で作成したトークン. 例) `ghp_` から始まる文字列 |
|GITHUB_REPO_NAMES|レポジトリ名のJSON配列. 例) `["github-gas-four-keys"]`|
|GITHUB_REPO_OWNER|レポジトリのOwnerあるいはOrganization名. 例) `cosoji-jp`|

5. [Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)から、 `initialize` 関数を実行し、スプレッドシートを初期化します.
6. [Apps Script プロジェクト](https://developers.google.com/apps-script/guides/projects?hl=ja#create-from-docs-sheets-slides)から、 `getAllRepos` 関数を実行し、GithubからPR情報を取得します.
7. スプレッドシートの"FourKeys計測結果"シートにFourKeys分析結果が出力されます.

### 自動計測設定
上記手順で `initialize` 関数を実行すると毎週日曜日00:00~01:00で自動的にPRが取得される[トリガー](https://developers.google.com/apps-script/guides/triggers/installable?hl=ja#time-driven_triggers)が設定されます。カスタマイズし任意の時間帯や間隔で実施することもできます.


## 各FourKeys項目の計測手法
### デプロイ頻度
GitHub上のPullRequestのマージ頻度を計測しています.
1日あたりの平均マージ回数によって、Elite/High/Medium/Lowの分類が行われます.

デフォルトでは1週間に3回以上(1日あたり0.4285714286回以上)の場合にEliteと判定されます.
この値はスプレッドシート上の分析設定シートE5セルで設定することができます.

### 変更リードタイム
PullRequestのソースブランチ(作業ブランチ)上で行われた初回コミットから、そのPullRequestがマージされるまでの時間を計測しています.
マージされるまでの平均時間によって、Elite/High/Medium/Lowの分類が行われます.

デフォルトでは平均24時間(1日)以内の場合にEliteと判定されます.
この値はスプレッドシート上の分析設定シートE8セルで設定することができます.

### 変更障害率
各PullRequestのラベルのうち、障害対応を行ったPR(ラベルに`app-bug/4K`、`bug/4K`、`hotfix/4K`が付くPR)の割合を計測しています.

この割合によって、Elite/High/Medium/Lowの分類が行われます.
デフォルトでは割合が15%以下の場合Eliteと判定されます.
この値はスプレッドシート上の分析設定シートE11セルで設定することができます.

障害対応を行ったPRの判定は、PRに付けられたラベルに基づいて自動的に行われます.

### 平均修復時間
障害対応を行ったPullRequest(ラベルに`app-bug/4K`、`bug/4K`、`hotfix/4K`が付くPR)の初回コミットから、そのPullRequestがマージされるまでの時間を計測しています.
マージされるまでの平均時間によって、Elite/High/Medium/Lowの分類が行われます.

デフォルトでは平均24時間(1日)以内の場合にEliteと判定されます.
この値はスプレッドシート上の分析設定シートE14セルで設定することができます.

## その他のカスタマイズ
### 計測範囲
FourKeys計測結果シートの計測は移動平均に基づいて計測されます.
移動平均のウィンドウ幅(日単位)は分析設定シートE2セルで設定することができます.
デフォルトでは28日(4週間)に設定されています.

### 計測対象ユーザの設定
分析設定シートのA列にPullRequestを作成したユーザ名が表示されます.
各ユーザの隣のセル(B列)に`FALSE`を入力するとそのユーザのPullRequestを計測対象から除外することができます.
`FALSE`以外の値が入力されている、もしくはブランクの場合、計測対象になります.
