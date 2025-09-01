# Git自動コミット・プッシュシステム セットアップガイド

このドキュメントは、Claude Codeと連携したGit自動コミット・プッシュシステムのセットアップ手順を説明します。

## システム概要

Claude Codeでの作業停止時に自動的に：
1. 変更内容を検出
2. Gemini AIがコミットメッセージを生成
3. 自動コミット・プッシュを実行

## 前提条件

- Windows 11
- Python 3.6以上
- Git（設定済み）
- Claude Code
- Google AI Studio APIキー

## 環境変数設定

このガイドでは以下の環境変数を使用します：

- `%TOOLS_DIR%` - ツール配置ディレクトリ（推奨: `C:\Tools`）
- `%USERPROFILE%` - ユーザーホームディレクトリ（システム標準）

**セットアップ前に実行**：
```bash
# TOOLS_DIR環境変数を設定
powershell -command "[Environment]::SetEnvironmentVariable('TOOLS_DIR', 'C:\Tools', 'User')"
```

## セットアップ手順

### 1. 必要ツールのインストール

#### Go言語のインストール
```bash
# Go 1.23.4をダウンロード・インストール
curl -L -o go1.23.4.windows-amd64.zip https://go.dev/dl/go1.23.4.windows-amd64.zip
# %TOOLS_DIR% に展開（推奨: C:\Tools\Go）
powershell -command "Expand-Archive -Path go1.23.4.windows-amd64.zip -DestinationPath %TOOLS_DIR%\Go"
```

#### Gemini CLIのインストール
```bash
# GOPATH設定
powershell -command "[Environment]::SetEnvironmentVariable('GOPATH', '%TOOLS_DIR%\Go\workspace', 'User')"

# Gemini CLIインストール
powershell -command "& '%TOOLS_DIR%\Go\go\bin\go.exe' install github.com/reugn/gemini-cli/cmd/gemini@latest"
```

### 2. claude-code-auto-commitツールの設置

```bash
# ツール用ディレクトリ作成
mkdir %TOOLS_DIR%

# リポジトリクローン
git clone https://github.com/owayo/claude-code-auto-commit.git %TOOLS_DIR%\claude-code-auto-commit
```

### 3. 実行バッチファイルの作成

`%TOOLS_DIR%\claude-code-auto-commit\run-auto-commit.bat`を作成：

```batch
@echo off
set "GEMINI_API_KEY=your-api-key-here"
set "CLAUDE_CODE_AUTO_PUSH=1"
set "CLAUDE_CODE_COMMIT_LANGUAGE=日本語"
set "CLAUDE_CODE_DEFAULT_COMMIT_MESSAGE=chore: Claude Codeによる自動修正"
set "PATH=%USERPROFILE%\go\bin;%PATH%"
python "%~dp0auto-git-commit.py"
```

### 4. Claude Code設定

`~/.claude/settings.json`を作成・編集：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "%TOOLS_DIR%\\claude-code-auto-commit\\run-auto-commit.bat",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**注意**: 実際の設定時は`%TOOLS_DIR%`を具体的なパス（例: `C:\\Tools`）に置き換えてください。

### 5. 環境変数の設定

#### Gemini APIキーの設定
1. [Google AI Studio](https://makersuite.google.com/app/apikey)でAPIキー取得
2. `run-auto-commit.bat`内の`your-api-key-here`を実際のAPIキーに置き換え

#### パス設定
```bash
# Goバイナリパスをユーザー環境変数に追加
powershell -command "[Environment]::SetEnvironmentVariable('PATH', [Environment]::GetEnvironmentVariable('PATH', 'User') + ';%USERPROFILE%\go\bin', 'User')"

# TOOLS_DIR環境変数の設定（推奨）
powershell -command "[Environment]::SetEnvironmentVariable('TOOLS_DIR', 'C:\Tools', 'User')"
```

### 6. プロジェクトのGit初期化

新しいプロジェクトに自動コミット機能を追加する場合：

```bash
cd "プロジェクトディレクトリ"
git init
git add .
git config user.name "Your Name"
git config user.email "your-email@example.com"
git commit -m "feat: 初回プロジェクト作成"

# リモートリポジトリ設定（プッシュ機能使用時）
git remote add origin https://github.com/username/repository.git
git push -u origin master
```

## 設定オプション

### 環境変数

| 変数名 | 説明 | デフォルト値 |
|--------|------|-------------|
| `CLAUDE_CODE_COMMIT_LANGUAGE` | コミットメッセージの言語 | 日本語 |
| `CLAUDE_CODE_DEFAULT_COMMIT_MESSAGE` | AI生成失敗時のメッセージ | `chore: Claude Codeによる自動修正` |
| `CLAUDE_CODE_AUTO_PUSH` | 自動プッシュ有効/無効 | `1`（有効） |

### .gitignore設定例

```gitignore
# 開発・テスト用ファイル
test.txt
*.tmp
*.log

# 環境固有の設定ファイル
.claude/
gemini_cli_config.json

# インストーラー・アーカイブファイル
*.msi
*.zip
*.tar.gz

# Python関連
__pycache__/
*.py[cod]
*$py.class

# 仮想環境（必要に応じて）
venv/
env/

# OS固有
.DS_Store
Thumbs.db
desktop.ini
```

## 動作確認

1. Claude Codeでプロジェクト内のファイルを編集
2. 作業を停止
3. 自動的にコミット・プッシュが実行されることを確認

## トラブルシューティング

### よくある問題

#### 1. `gemini` コマンドが見つからない
- Goのインストール確認
- PATH環境変数の設定確認
- Gemini CLIの再インストール

#### 2. APIキーエラー
- APIキーの正確性を確認
- 環境変数の設定確認
- Google AI Studioでのクォータ確認

#### 3. Git権限エラー
- Gitの認証設定確認
- リモートリポジトリのアクセス権確認
- SSH/HTTPS設定の確認

#### 4. PowerShell実行ポリシーエラー
```bash
# 実行ポリシーの変更（管理者権限で実行）
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### ログ確認

バッチファイルにログ出力を追加：
```batch
@echo off
echo [%date% %time%] Starting auto-commit process >> %TOOLS_DIR%\auto-commit.log
set "GEMINI_API_KEY=your-api-key-here"
...
python "%~dp0auto-git-commit.py" 2>&1 >> %TOOLS_DIR%\auto-commit.log
```

## セキュリティ注意事項

1. **APIキーの管理**
   - バッチファイル内のAPIキーは適切に管理
   - 必要に応じて環境変数での設定を検討

2. **Git認証情報**
   - 適切なGit認証設定を使用
   - パーソナルアクセストークンの使用を推奨

3. **アクセス権限**
   - ツールディレクトリへの適切なアクセス権設定
   - 不要な権限昇格は避ける

## ライセンス・クレジット

- claude-code-auto-commit: MIT License
- Gemini CLI: 各プロジェクトのライセンスに従う
- 参考記事: [Zenn記事](https://zenn.dev/owayo/articles/9461d3ebb3b77a)