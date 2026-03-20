---
name: python-patterns
description: Pythonicなイディオム、PEP 8標準、型ヒント、堅牢で効率的かつ保守性の高いPythonアプリケーション構築のためのベストプラクティス。
---

# Python 開発パターン

堅牢で効率的かつ保守性の高いアプリケーション構築のための、慣用的なPythonパターンとベストプラクティス。

## 活性化するタイミング

- 新しいPythonコードを書くとき
- Pythonコードをレビューするとき
- 既存のPythonコードをリファクタリングするとき
- Pythonパッケージ/モジュールを設計するとき

## コアプリンシプル

### 1. 可読性が重要

Pythonは可読性を最優先にします。コードは明確で理解しやすくあるべきです。

```python
# 良い例: 明確で読みやすい
def get_active_users(users: list[User]) -> list[User]:
    """提供されたリストからアクティブなユーザーのみを返す。"""
    return [user for user in users if user.is_active]


# 悪い例: 賢いが分かりにくい
def get_active_users(u):
    return [x for x in u if x.a]
```

### 2. 暗黙より明示

マジックを避け、コードが何をするかを明確にします。

```python
# 良い例: 明示的な設定
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# 悪い例: 隠れた副作用
import some_module
some_module.setup()  # これは何をする？
```

### 3. EAFP - 許可を求めるより許しを求める方が簡単

Pythonは条件チェックより例外処理を好みます。

```python
# 良い例: EAFPスタイル
def get_value(dictionary: dict, key: str) -> Any:
    try:
        return dictionary[key]
    except KeyError:
        return default_value

# 悪い例: LBYLスタイル（先に見てから飛び込む）
def get_value(dictionary: dict, key: str) -> Any:
    if key in dictionary:
        return dictionary[key]
    else:
        return default_value
```

## 型ヒント

### 基本的な型アノテーション

```python
from typing import Optional, List, Dict, Any

def process_user(
    user_id: str,
    data: Dict[str, Any],
    active: bool = True
) -> Optional[User]:
    """ユーザーを処理し、更新されたUserまたはNoneを返す。"""
    if not active:
        return None
    return User(user_id, data)
```

### モダンな型ヒント（Python 3.9+）

```python
# Python 3.9+ - 組み込み型を使用
def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}

# Python 3.8以前 - typingモジュールを使用
from typing import List, Dict

def process_items(items: List[str]) -> Dict[str, int]:
    return {item: len(item) for item in items}
```

### 型エイリアスとTypeVar

```python
from typing import TypeVar, Union

# 複雑な型の型エイリアス
JSON = Union[dict[str, Any], list[Any], str, int, float, bool, None]

def parse_json(data: str) -> JSON:
    return json.loads(data)

# ジェネリック型
T = TypeVar('T')

def first(items: list[T]) -> T | None:
    """最初の要素を返す。リストが空の場合はNone。"""
    return items[0] if items else None
```

### プロトコルベースのダックタイピング

```python
from typing import Protocol

class Renderable(Protocol):
    def render(self) -> str:
        """オブジェクトを文字列にレンダリングする。"""

def render_all(items: list[Renderable]) -> str:
    """Renderableプロトコルを実装するすべてのアイテムをレンダリング。"""
    return "\n".join(item.render() for item in items)
```

## エラーハンドリングパターン

### 特定の例外処理

```python
# 良い例: 特定の例外を捕捉
def load_config(path: str) -> Config:
    try:
        with open(path) as f:
            return Config.from_json(f.read())
    except FileNotFoundError as e:
        raise ConfigError(f"設定ファイルが見つかりません: {path}") from e
    except json.JSONDecodeError as e:
        raise ConfigError(f"設定のJSONが無効です: {path}") from e

# 悪い例: 裸のexcept
def load_config(path: str) -> Config:
    try:
        with open(path) as f:
            return Config.from_json(f.read())
    except:
        return None  # サイレント失敗！
```

### 例外チェーン

```python
def process_data(data: str) -> Result:
    try:
        parsed = json.loads(data)
    except json.JSONDecodeError as e:
        # トレースバックを保持するために例外をチェーン
        raise ValueError(f"データの解析に失敗: {data}") from e
```

### カスタム例外階層

```python
class AppError(Exception):
    """すべてのアプリケーションエラーの基底例外。"""
    pass

class ValidationError(AppError):
    """入力バリデーション失敗時に発生。"""
    pass

class NotFoundError(AppError):
    """リクエストされたリソースが見つからない時に発生。"""
    pass

# 使用例
def get_user(user_id: str) -> User:
    user = db.find_user(user_id)
    if not user:
        raise NotFoundError(f"ユーザーが見つかりません: {user_id}")
    return user
```

## コンテキストマネージャー

### リソース管理

```python
# 良い例: コンテキストマネージャーを使用
def process_file(path: str) -> str:
    with open(path, 'r') as f:
        return f.read()

# 悪い例: 手動リソース管理
def process_file(path: str) -> str:
    f = open(path, 'r')
    try:
        return f.read()
    finally:
        f.close()
```

### カスタムコンテキストマネージャー

```python
from contextlib import contextmanager

@contextmanager
def timer(name: str):
    """コードブロックを計時するコンテキストマネージャー。"""
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    print(f"{name} に {elapsed:.4f} 秒かかりました")

# 使用例
with timer("データ処理"):
    process_large_dataset()
```

## 内包表記とジェネレーター

### リスト内包表記

```python
# 良い例: シンプルな変換にリスト内包表記
names = [user.name for user in users if user.is_active]

# 悪い例: 手動ループ
names = []
for user in users:
    if user.is_active:
        names.append(user.name)

# 複雑な内包表記は展開すべき
# 悪い例: 複雑すぎ
result = [x * 2 for x in items if x > 0 if x % 2 == 0]

# 良い例: ジェネレーター関数を使用
def filter_and_transform(items: Iterable[int]) -> list[int]:
    result = []
    for x in items:
        if x > 0 and x % 2 == 0:
            result.append(x * 2)
    return result
```

### ジェネレーター式

```python
# 良い例: 遅延評価のためにジェネレーター
total = sum(x * x for x in range(1_000_000))

# 悪い例: 大きな中間リストを作成
total = sum([x * x for x in range(1_000_000)])
```

### ジェネレーター関数

```python
def read_large_file(path: str) -> Iterator[str]:
    """大きなファイルを1行ずつ読み込む。"""
    with open(path) as f:
        for line in f:
            yield line.strip()

# 使用例
for line in read_large_file("huge.txt"):
    process(line)
```

## データクラスと名前付きタプル

### データクラス

```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    """自動的な__init__、__repr__、__eq__を持つユーザーエンティティ。"""
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True

# 使用例
user = User(
    id="123",
    name="Alice",
    email="alice@example.com"
)
```

### バリデーション付きデータクラス

```python
@dataclass
class User:
    email: str
    age: int

    def __post_init__(self):
        # メールフォーマットをバリデート
        if "@" not in self.email:
            raise ValueError(f"無効なメール: {self.email}")
        # 年齢範囲をバリデート
        if self.age < 0 or self.age > 150:
            raise ValueError(f"無効な年齢: {self.age}")
```

### 名前付きタプル

```python
from typing import NamedTuple

class Point(NamedTuple):
    """イミュータブルな2D点。"""
    x: float
    y: float

    def distance(self, other: 'Point') -> float:
        return ((self.x - other.x) ** 2 + (self.y - other.y) ** 2) ** 0.5

# 使用例
p1 = Point(0, 0)
p2 = Point(3, 4)
print(p1.distance(p2))  # 5.0
```

## デコレーター

### 関数デコレーター

```python
import functools
import time

def timer(func: Callable) -> Callable:
    """関数実行を計時するデコレーター。"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} に {elapsed:.4f}秒かかりました")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

### パラメータ付きデコレーター

```python
def repeat(times: int):
    """関数を複数回繰り返すデコレーター。"""
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(times=3)
def greet(name: str) -> str:
    return f"こんにちは、{name}さん！"
```

## 並行性パターン

### I/Oバウンドタスクのスレッディング

```python
import concurrent.futures

def fetch_url(url: str) -> str:
    """URLをフェッチする（I/Oバウンド操作）。"""
    import urllib.request
    with urllib.request.urlopen(url) as response:
        return response.read().decode()

def fetch_all_urls(urls: list[str]) -> dict[str, str]:
    """スレッドを使用して複数URLを並行フェッチ。"""
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
        future_to_url = {executor.submit(fetch_url, url): url for url in urls}
        results = {}
        for future in concurrent.futures.as_completed(future_to_url):
            url = future_to_url[future]
            try:
                results[url] = future.result()
            except Exception as e:
                results[url] = f"エラー: {e}"
    return results
```

### CPUバウンドタスクのマルチプロセッシング

```python
def process_data(data: list[int]) -> int:
    """CPU集約的な計算。"""
    return sum(x ** 2 for x in data)

def process_all(datasets: list[list[int]]) -> list[int]:
    """複数プロセスを使用して複数データセットを処理。"""
    with concurrent.futures.ProcessPoolExecutor() as executor:
        results = list(executor.map(process_data, datasets))
    return results
```

### 並行I/OのAsync/Await

```python
import asyncio

async def fetch_async(url: str) -> str:
    """URLを非同期でフェッチ。"""
    import aiohttp
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def fetch_all(urls: list[str]) -> dict[str, str]:
    """複数URLを並行フェッチ。"""
    tasks = [fetch_async(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return dict(zip(urls, results))
```

## メモリとパフォーマンス

### メモリ効率のための__slots__

```python
# 悪い例: 通常のクラスは__dict__を使用（メモリ消費大）
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

# 良い例: __slots__でメモリ使用量を削減
class Point:
    __slots__ = ['x', 'y']

    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
```

### ループ内の文字列結合を避ける

```python
# 悪い例: 文字列のイミュータビリティによりO(n²)
result = ""
for item in items:
    result += str(item)

# 良い例: joinを使用してO(n)
result = "".join(str(item) for item in items)
```

## Pythonツール統合

### 必須コマンド

```bash
# コードフォーマット
ruff format .

# リント
ruff check .

# 型チェック
mypy .

# テスト
pytest --cov=src --cov-report=html

# セキュリティスキャン
bandit -r .
```

## クイックリファレンス: Pythonイディオム

| イディオム | 説明 |
|-------|-------------|
| EAFP | 許可を求めるより許しを求める方が簡単 |
| コンテキストマネージャー | リソース管理に `with` を使用 |
| リスト内包表記 | シンプルな変換に使用 |
| ジェネレーター | 遅延評価と大規模データセットに使用 |
| 型ヒント | 関数シグネチャにアノテーション |
| データクラス | 自動生成メソッド付きデータコンテナに使用 |
| `__slots__` | メモリ最適化に使用 |
| f-strings | 文字列フォーマットに使用（Python 3.6+）|
| `pathlib.Path` | パス操作に使用（Python 3.4+）|
| `enumerate` | ループ内のインデックスと要素のペアに使用 |

## 避けるべきアンチパターン

```python
# 悪い例: ミュータブルなデフォルト引数
def append_to(item, items=[]):
    items.append(item)
    return items

# 良い例: Noneを使用して新しいリストを作成
def append_to(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# 悪い例: type()で型チェック
if type(obj) == list:
    process(obj)

# 良い例: isinstanceを使用
if isinstance(obj, list):
    process(obj)

# 悪い例: ==でNoneと比較
if value == None:
    process()

# 良い例: isを使用
if value is None:
    process()

# 悪い例: from module import *
from os.path import *

# 良い例: 明示的なインポート
from os.path import join, exists

# 悪い例: 裸のexcept
try:
    risky_operation()
except:
    pass

# 良い例: 特定の例外
try:
    risky_operation()
except SpecificError as e:
    logger.error(f"操作に失敗しました: {e}")
```

**注意**: Pythonコードは読みやすく、明示的で、最小驚きの原則に従うべきです。迷ったときは、賢さより明快さを優先してください。
