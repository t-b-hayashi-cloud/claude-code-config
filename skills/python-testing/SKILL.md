---
name: python-testing
description: pytest、TDD手法、フィクスチャ、モック、パラメトライズ、カバレッジ要件を使用したPythonテスト戦略。
---

# Python テストパターン

pytestとTDD手法を使用したPythonアプリケーションの包括的なテスト戦略とベストプラクティス。

## 活性化するタイミング

- 新しいPythonコードを書くとき（TDDに従う: レッド、グリーン、リファクタリング）
- Pythonプロジェクトのテストスイートを設計するとき
- Pythonテストカバレッジをレビューするとき
- テストインフラをセットアップするとき

## テストの基本哲学

### テスト駆動開発（TDD）

常にTDDサイクルに従います：

1. **RED**: 望む動作に対する失敗するテストを書く
2. **GREEN**: テストを通過させる最小限のコードを書く
3. **REFACTOR**: テストをグリーンに保ちながらコードを改善する

```python
# ステップ1: 失敗するテストを書く（RED）
def test_add_numbers():
    result = add(2, 3)
    assert result == 5

# ステップ2: 最小限の実装を書く（GREEN）
def add(a, b):
    return a + b

# ステップ3: 必要に応じてリファクタリング（REFACTOR）
```

### カバレッジ要件

- **目標**: 80%以上のコードカバレッジ
- **クリティカルパス**: 100%カバレッジが必要
- `pytest --cov` でカバレッジを測定

```bash
pytest --cov=mypackage --cov-report=term-missing --cov-report=html
```

## pytest 基礎

### 基本的なテスト構造

```python
import pytest

def test_addition():
    """基本的な加算をテスト。"""
    assert 2 + 2 == 4

def test_string_uppercase():
    """文字列の大文字化をテスト。"""
    text = "hello"
    assert text.upper() == "HELLO"

def test_list_append():
    """リストの追加をテスト。"""
    items = [1, 2, 3]
    items.append(4)
    assert 4 in items
    assert len(items) == 4
```

### アサーション

```python
# 等値
assert result == expected

# 非等値
assert result != unexpected

# 真偽値
assert result  # 真
assert not result  # 偽
assert result is True  # 厳密にTrue
assert result is None  # 厳密にNone

# メンバーシップ
assert item in collection

# 比較
assert result > 0
assert 0 <= result <= 100

# 型チェック
assert isinstance(result, str)

# 例外テスト（推奨アプローチ）
with pytest.raises(ValueError):
    raise ValueError("エラーメッセージ")

# 例外メッセージを確認
with pytest.raises(ValueError, match="無効な入力"):
    raise ValueError("無効な入力が提供されました")
```

## フィクスチャ

### 基本的なフィクスチャの使用

```python
import pytest

@pytest.fixture
def sample_data():
    """サンプルデータを提供するフィクスチャ。"""
    return {"name": "Alice", "age": 30}

def test_sample_data(sample_data):
    """フィクスチャを使用したテスト。"""
    assert sample_data["name"] == "Alice"
    assert sample_data["age"] == 30
```

### セットアップ/ティアダウン付きフィクスチャ

```python
@pytest.fixture
def database():
    """セットアップとティアダウン付きフィクスチャ。"""
    # セットアップ
    db = Database(":memory:")
    db.create_tables()
    db.insert_test_data()

    yield db  # テストに提供

    # ティアダウン
    db.close()

def test_database_query(database):
    """データベース操作のテスト。"""
    result = database.query("SELECT * FROM users")
    assert len(result) > 0
```

### フィクスチャのスコープ

```python
# 関数スコープ（デフォルト）- 各テストで実行
@pytest.fixture
def temp_file():
    with open("temp.txt", "w") as f:
        yield f
    os.remove("temp.txt")

# モジュールスコープ - モジュールごとに1回実行
@pytest.fixture(scope="module")
def module_db():
    db = Database(":memory:")
    db.create_tables()
    yield db
    db.close()

# セッションスコープ - テストセッションごとに1回実行
@pytest.fixture(scope="session")
def shared_resource():
    resource = ExpensiveResource()
    yield resource
    resource.cleanup()
```

### Conftest.py で共有フィクスチャ

```python
# tests/conftest.py
import pytest

@pytest.fixture
def client():
    """すべてのテスト用の共有フィクスチャ。"""
    app = create_app(testing=True)
    with app.test_client() as client:
        yield client

@pytest.fixture
def auth_headers(client):
    """APIテスト用の認証ヘッダーを生成。"""
    response = client.post("/api/login", json={
        "username": "test",
        "password": "test"
    })
    token = response.json["token"]
    return {"Authorization": f"Bearer {token}"}
```

## パラメトライズ

### 基本的なパラメトライズ

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("PyThOn", "PYTHON"),
])
def test_uppercase(input, expected):
    """異なる入力で3回実行されるテスト。"""
    assert input.upper() == expected
```

### 複数パラメーター

```python
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
])
def test_add(a, b, expected):
    """複数入力での加算テスト。"""
    assert add(a, b) == expected
```

### ID付きパラメトライズ

```python
@pytest.mark.parametrize("input,expected", [
    ("valid@email.com", True),
    ("invalid", False),
    ("@no-domain.com", False),
], ids=["有効なメール", "アットマークなし", "ドメインなし"])
def test_email_validation(input, expected):
    """読みやすいテストIDでのメールバリデーションテスト。"""
    assert is_valid_email(input) is expected
```

## マーカーとテスト選択

### カスタムマーカー

```python
# 遅いテストをマーク
@pytest.mark.slow
def test_slow_operation():
    time.sleep(5)

# 統合テストをマーク
@pytest.mark.integration
def test_api_integration():
    response = requests.get("https://api.example.com")
    assert response.status_code == 200

# ユニットテストをマーク
@pytest.mark.unit
def test_unit_logic():
    assert calculate(2, 3) == 5
```

### 特定のテストを実行

```bash
# 速いテストのみ実行
pytest -m "not slow"

# 統合テストのみ実行
pytest -m integration

# 統合または遅いテストを実行
pytest -m "integration or slow"
```

## モックとパッチ

### 関数のモック

```python
from unittest.mock import patch, Mock

@patch("mypackage.external_api_call")
def test_with_mock(api_call_mock):
    """モックされた外部APIでテスト。"""
    api_call_mock.return_value = {"status": "success"}

    result = my_function()

    api_call_mock.assert_called_once()
    assert result["status"] == "success"
```

### 例外のモック

```python
@patch("mypackage.api_call")
def test_api_error_handling(api_call_mock):
    """モックされた例外でエラーハンドリングをテスト。"""
    api_call_mock.side_effect = ConnectionError("ネットワークエラー")

    with pytest.raises(ConnectionError):
        api_call()

    api_call_mock.assert_called_once()
```

### コンテキストマネージャーのモック

```python
@patch("builtins.open", new_callable=mock_open)
def test_file_reading(mock_file):
    """モックされたopenでファイル読み込みをテスト。"""
    mock_file.return_value.read.return_value = "ファイルコンテンツ"

    result = read_file("test.txt")

    mock_file.assert_called_once_with("test.txt", "r")
    assert result == "ファイルコンテンツ"
```

## 非同期コードのテスト

### pytest-asyncioを使った非同期テスト

```python
import pytest

@pytest.mark.asyncio
async def test_async_function():
    """非同期関数のテスト。"""
    result = await async_add(2, 3)
    assert result == 5

@pytest.mark.asyncio
async def test_async_with_fixture(async_client):
    """非同期フィクスチャを使った非同期テスト。"""
    response = await async_client.get("/api/users")
    assert response.status_code == 200
```

## テスト組織

### ディレクトリ構造

```
tests/
├── conftest.py                 # 共有フィクスチャ
├── __init__.py
├── unit/                       # ユニットテスト
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_utils.py
│   └── test_services.py
├── integration/                # 統合テスト
│   ├── __init__.py
│   ├── test_api.py
│   └── test_database.py
└── e2e/                        # エンドツーエンドテスト
    ├── __init__.py
    └── test_user_flow.py
```

### テストクラス

```python
class TestUserService:
    """関連するテストをクラスにグループ化。"""

    @pytest.fixture(autouse=True)
    def setup(self):
        """このクラスの各テスト前に実行されるセットアップ。"""
        self.service = UserService()

    def test_create_user(self):
        """ユーザー作成のテスト。"""
        user = self.service.create_user("Alice")
        assert user.name == "Alice"

    def test_delete_user(self):
        """ユーザー削除のテスト。"""
        user = User(id=1, name="Bob")
        self.service.delete_user(user)
        assert not self.service.user_exists(1)
```

## ベストプラクティス

### すべきこと

- **TDDに従う**: コードの前にテストを書く（レッド・グリーン・リファクタリング）
- **1つのことをテスト**: 各テストは1つの動作を検証する
- **説明的な名前を使用**: `test_無効な認証情報でのユーザーログイン失敗`
- **フィクスチャを使用**: フィクスチャで重複を排除
- **外部依存関係をモック**: 外部サービスに依存しない
- **エッジケースをテスト**: 空の入力、None値、境界条件
- **80%以上のカバレッジを目指す**: クリティカルパスに集中
- **テストを速く保つ**: マーカーで遅いテストを分離

### すべきでないこと

- **実装をテストしない**: 内部でなく動作をテストする
- **テストに複雑な条件分岐を使わない**: テストはシンプルに保つ
- **テスト失敗を無視しない**: すべてのテストが通過しなければならない
- **サードパーティコードをテストしない**: ライブラリを信頼する
- **テスト間で状態を共有しない**: テストは独立しているべき
- **テストで例外を捕捉しない**: `pytest.raises` を使用する

## 一般的なパターン

### APIエンドポイントのテスト（FastAPI/Flask）

```python
@pytest.fixture
def client():
    app = create_app(testing=True)
    return app.test_client()

def test_get_user(client):
    response = client.get("/api/users/1")
    assert response.status_code == 200
    assert response.json["id"] == 1

def test_create_user(client):
    response = client.post("/api/users", json={
        "name": "Alice",
        "email": "alice@example.com"
    })
    assert response.status_code == 201
    assert response.json["name"] == "Alice"
```

### データベース操作のテスト

```python
@pytest.fixture
def db_session():
    """テストデータベースセッションを作成。"""
    session = Session(bind=engine)
    session.begin_nested()
    yield session
    session.rollback()
    session.close()

def test_create_user(db_session):
    user = User(name="Alice", email="alice@example.com")
    db_session.add(user)
    db_session.commit()

    retrieved = db_session.query(User).filter_by(name="Alice").first()
    assert retrieved.email == "alice@example.com"
```

## テストの実行

```bash
# すべてのテストを実行
pytest

# 特定のファイルを実行
pytest tests/test_utils.py

# 詳細出力で実行
pytest -v

# カバレッジ付きで実行
pytest --cov=mypackage --cov-report=html

# 速いテストのみ実行
pytest -m "not slow"

# 最初の失敗で停止
pytest -x

# 最後に失敗したテストを実行
pytest --lf

# パターンでテストを実行
pytest -k "test_user"

# 失敗時にデバッガーで実行
pytest --pdb
```

## クイックリファレンス

| パターン | 使用法 |
|---------|-------|
| `pytest.raises()` | 期待される例外をテスト |
| `@pytest.fixture()` | 再利用可能なテストフィクスチャを作成 |
| `@pytest.mark.parametrize()` | 複数の入力でテストを実行 |
| `@pytest.mark.slow` | 遅いテストをマーク |
| `pytest -m "not slow"` | 遅いテストをスキップ |
| `@patch()` | 関数とクラスをモック |
| `tmp_path` フィクスチャ | 自動一時ディレクトリ |
| `pytest --cov` | カバレッジレポートを生成 |
| `assert` | シンプルで読みやすいアサーション |

**注意**: テストもコードです。クリーンで読みやすく、保守しやすく保ちましょう。良いテストはバグを見つけ、優れたテストはバグを防ぎます。
