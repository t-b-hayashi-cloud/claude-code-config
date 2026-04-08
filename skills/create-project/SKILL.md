---
name: create-project
description: データサイエンスプロジェクトのディレクトリ構造・テンプレートファイルを一括生成します。ds_instructions_guide リポジトリ準拠の構成。
---

# プロジェクト作成スキル

## 概要

このスキルはデータサイエンスプロジェクトの標準ディレクトリ構造とテンプレートファイルを生成します。

## 手順

### ステップ1：プロジェクト名とプロジェクト目的の確認

引数が指定されていればそれをプロジェクト名として使用する。未指定の場合はユーザーに確認する。

プロジェクト名はスネークケース（例: `my_project`）を推奨。
`src/<project_name>/` のモジュール名にも使用される。

次に、**プロジェクトの目的**をユーザーに尋ねる。以下を質問する:

- このプロジェクトで何を達成したいか（ビジネス課題・分析目標）
- 主なユースケースや対象ユーザー（あれば）

回答を `CLAUDE.md` の「プロジェクト目的」セクションに記載する。

### ステップ2：ディレクトリ構造の作成

以下のディレクトリとファイルを作成する。カレントディレクトリ直下に `<project_name>/` を作成する。

```
<project_name>/
├── .devcontainer/
│   └── devcontainer.json       # VSCode Dev Container設定
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   └── .gitkeep
│   └── PULL_REQUEST_TEMPLATE.md
├── .streamlit/
│   └── config.toml             # Streamlit設定（使用統計の無効化等）
├── app/
│   └── .gitkeep                # Streamlitなどのアプリケーションコード
├── data/
│   └── .gitkeep                # データセット（gitignoreされる）
├── docs/
│   └── .gitkeep                # プロジェクトドキュメント・分析計画・分析結果
├── model/
│   └── .gitkeep                # 学習済みモデルの保存先（gitignoreされる）
├── notebook/
│   └── .gitkeep                # EDA・分析用Jupyter Notebook
├── outputs/
│   └── .gitkeep                # 分析結果・可視化の出力先（gitignoreされる）
├── scripts/
│   └── .gitkeep                # データ取得・前処理などのユーティリティスクリプト
├── src/
│   └── <project_name>/
│       └── __init__.py         # メインのソースコード・モジュール
├── tests/
│   └── __init__.py             # pytestによるテストコード
├── .env.example
├── .gitignore
├── CLAUDE.md                   # プロジェクト目的・Claude Codeへの指示
├── Dockerfile
├── README.md
├── compose.yml
└── pyproject.toml
```

### ステップ3：各ファイルの内容

#### `CLAUDE.md`

```markdown
# <project_name>

## プロジェクト目的

<!-- ステップ1でユーザーから聞いた目的をここに記載 -->
<ユーザーが回答したプロジェクトの目的・ビジネス課題・分析目標>

## 主なユースケース・対象ユーザー

<!-- ステップ1でユーザーから聞いた内容があれば記載、なければ省略 -->

## 分析の方向性

このプロジェクトでは `/ppdac-workflow` に沿ってデータ分析を進める。
分析計画は `docs/analysis_plan/`、分析結果は `docs/analysis_result/` に保存される。
```

#### `.gitignore`

```gitignore
# Python
__pycache__/
.pytest_cache/
.mypy_cache/
.ipynb_checkpoints
.venv/
*.egg-info/

# 環境変数
.env

# エディタ
.vscode/
.DS_Store

# 実験管理
mlruns/
multirun/

# データ・モデル・出力（.gitkeepは除外しない）
data/*
!data/.gitkeep
model/*
!model/.gitkeep
outputs/*
!outputs/.gitkeep
```

#### `.env.example`

```dotenv
# コンテナ名（複数のdocker-composeを使用する場合はプロジェクトごとに変更してください）
CONTAINER_NAME=<project_name>

# Streamlitのポート番号（複数立ち上げる場合は競合しないよう変更してください）
PORT=8501
```

#### `compose.yml`

```yaml
services:
  app:
    build: .
    container_name: ${CONTAINER_NAME}
    volumes:
      - .:/app:cached
    working_dir: /app
    env_file:
      - .env
    ports:
      - "127.0.0.1:${PORT}:8501"
    tty: true
```

#### `Dockerfile`

```dockerfile
FROM python:3.12-slim

# システムパッケージ
RUN apt-get update && apt-get install -y \
    git curl vim tmux \
    locales \
    && locale-gen ja_JP.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=ja_JP.UTF-8 \
    TZ=Asia/Tokyo \
    UV_SYSTEM_PYTHON=1 \
    UV_LINK_MODE=copy

# uv のインストール
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.cargo/bin:$PATH"

WORKDIR /app

# 依存関係のキャッシュ（ソースコードより変更頻度が低い）
COPY pyproject.toml uv.lock* ./
RUN uv sync --frozen --no-install-project

# ソースコードのコピー
COPY src/ src/
COPY app/ app/

EXPOSE 8501
CMD ["uv", "run", "streamlit", "run", "app/main.py"]
```

#### `pyproject.toml`

```toml
[project]
name = "<project_name>"
version = "0.1.0"
description = ""
requires-python = ">=3.12,<3.13"
dependencies = [
    "pandas>=2.0",
    "numpy>=1.26",
    "scikit-learn>=1.4",
    "streamlit>=1.30",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
    "mypy>=1.9",
    "ruff>=0.4",
    "jupyter>=1.0",
]

[tool.ruff]
select = ["E", "F", "B", "I"]
line-length = 88

[tool.mypy]
strict = true
ignore_missing_imports = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --cov=src --cov-report=term-missing"

[tool.coverage.report]
fail_under = 80
```

#### `README.md`

```markdown
# <project_name>

## ディレクトリ構成

```
<project_name>/
├── .devcontainer/   VSCode Dev Container設定（Docker環境の自動構築）
├── .github/         GitHubテンプレート（Issue・PRテンプレート）
├── .streamlit/      Streamlitアプリの設定
├── app/             Streamlit等のアプリケーションコード
├── data/            データセット置き場（gitignore済み）
├── docs/            プロジェクトドキュメント・分析計画・分析結果レポート
├── model/           学習済みモデルの保存先（gitignore済み）
├── notebook/        EDA・探索的分析用Jupyter Notebook
├── outputs/         分析結果・グラフ等の出力先（gitignore済み）
├── scripts/         データ取得・バッチ処理などのスクリプト
├── src/             メインのソースコード・パッケージ
└── tests/           pytestによるテストコード
```

## セットアップ

1. `.env.example` をコピーして `.env` を作成する

```bash
cp .env.example .env
```

2. （Docker使用時）Dev Containerを起動する、またはDocker Composeで起動する

```bash
docker compose up -d
```

3. （ローカル使用時）uvで依存関係をインストールする

```bash
uv sync
```
```

#### `.github/PULL_REQUEST_TEMPLATE.md`

```markdown
## レビュー期日

## 何を達成したいのか、なぜその変更をしたか

## 特にレビューして欲しい箇所

## 関連情報やissuesなど（あれば）
```

#### `.devcontainer/devcontainer.json`

```json
{
  "name": "<project_name>",
  "dockerComposeFile": "../compose.yml",
  "service": "app",
  "workspaceFolder": "/app",
  "shutdownAction": "stopCompose",
  "customizations": {
    "vscode": {
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "[python]": {
          "editor.defaultFormatter": "charliermarsh.ruff",
          "editor.formatOnSave": true,
          "editor.codeActionsOnSave": {
            "source.fixAll.ruff": "explicit",
            "source.organizeImports.ruff": "explicit"
          }
        },
        "[jupyter]": {
          "editor.defaultFormatter": "charliermarsh.ruff",
          "editor.formatOnSave": true
        },
        "python.testing.pytestEnabled": true,
        "python.testing.pytestArgs": ["-vv", "-s"],
        "mypy-type-checker.enabled": true,
        "autoDocstring.docstringFormat": "google"
      },
      "extensions": [
        "ms-python.python",
        "ms-python.mypy-type-checker",
        "ms-toolsai.jupyter",
        "charliermarsh.ruff",
        "ms-azuretools.vscode-docker",
        "mhutchie.git-graph",
        "ms-ceintl.vscode-language-pack-ja",
        "GrapeCity.gc-excelviewer"
      ]
    }
  }
}
```

#### `.streamlit/config.toml`

```toml
[browser]
gatherUsageStats = false
```

#### `src/<project_name>/__init__.py`

```python
```

（空ファイルでよい）

#### `tests/__init__.py`

```python
```

（空ファイルでよい）

### ステップ4：完了メッセージ

作成後、以下を案内する：

1. 生成したディレクトリ構造の一覧（tree形式）
2. 次のステップ：
   - `cd <project_name>` でプロジェクトに移動
   - `.env.example` を `.env` にコピー
   - `uv sync --dev` で依存関係をインストール（またはDocker Composeで起動）
   - `git init && git add . && git commit -m "chore: initial project setup"` で初期コミット

## 注意事項

- `data/`・`model/`・`outputs/` の内容は `.gitignore` で管理対象外。`.gitkeep` のみgit管理される
- `<project_name>` はスネークケースで統一する（`pyproject.toml` の `name`・`src/` 配下のモジュール名・`compose.yml` のコンテナ名）
- `uv.lock` はDocker build時の再現性のために `uv sync --frozen` を使用する。初回は `uv lock` で生成する
