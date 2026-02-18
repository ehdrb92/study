## Typing Overloads

```python
from typing import Literal, overload


@overload
def transform(data: str, mode: Literal["split"]) -> list[str]: ...

@overload
def transform(data: str, mode: Literal["upper"]) -> str: ...

# 실제 사용함수
# 리턴 값이 둘 중 하나가 될 수 있지만 이전에 구현부 없이 정의한 함수가 타입 체크가 되도록 돕는다.
def transform(data: str, mode: Literal["split", "upper"]) -> list[str] | str:
    if mode == "split":
        return data.split()
    else:
        return data.upper()

split_words = transform("hello world", "split")  # Return type is list[str]
split_words[0]  # 타입 체크 문제 없음

upper_words = transform("hello world", "upper")  # Return type is str
upper_words.lower()  # 타입 체크 문제 없음
upper_words.append("!")  # Cannot access attribute "append" for "str"
```

파이썬의 @overload는 일반적으로 다른 프로그래밍 언어와 다르게 동작한다는 점을 인지하자. 일반적으로 overload는 함수의 다른 버전을 만드는 것이지만 파이썬에서는 타입 체크 힌트를 제공하기 위함이다.

```python
from typing import Literal


def set_color(color: Literal["red", "blue", "green"]) -> None: ...


set_color("red")
set_color("blue")
set_color("green")
set_color("fuchsia")  # Literal을 이용한 명시로 타입 체킹도 가능하다.
```

## Keyword-only and Positional-only Arguments

`*`는 함수에서 키워드 전용 매개변수를 표현하기 위함이고, `/`는 위치 전용 매개변수를 표현하기 위함이다.

```python
#   KW+POS | KW ONLY
#       vv | vv
def foo(a, *, b): ...


# == ALLOWED ==
foo(a=1, b=2)  # All keyword
foo(1, b=2)  # Half positional, half keyword


# == NOT ALLOWED ==
foo(1, 2)  # Cannot use positional for keyword-only parameter
#      ^

# POS ONLY | KW POS
#       vv | vv
def bar(a, /, b): ...


# == ALLOWED ==
bar(1, 2)  # All positional
bar(1, b=2)  # Half positional, half keyword

# == NOT ALLOWED ==
bar(a=1, b=2)  # Cannot use keyword for positional-only parameter
#   ^
```

## Protocal

```python
class Duck:
    def quack(self):
        print("Quack!")


class Person:
    def quack(self):
        print("I'm quacking!")


class Dog:
    def bark(self):
        print("Woof!")


def run_quack(obj):
    obj.quack()

run_quack(Duck())  # Works!
run_quack(Person())  # Works!
run_quack(Dog())  # Fails with AttributeError
```

```python
from typing import Protocol


class Quackable(Protocol):
    def quack(self) -> None: ...  # The ellipsis indicates this is just a method signature


class Duck:
    def quack(self):
        print("Quack!")

class Dog:
    def bark(self):
        print("Woof!")

def run_quack(obj: Quackable):
    obj.quack()

run_quack(Duck())  # Works!
run_quack(Dog())  # Fails during TYPE CHECKING (and runtime)
```

Protocol 기능은 마치 자바의 인터페이스와 유사한 것으로 보인다. @runtime_checkable 데코레이터를 Protocol 클래스에 추가하면 isinstance() 함수를 통해 타입 체크가 가능하다.

```python
from typing import runtime_checkable, Protocol


@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...
```

## Context Manager

```python
# OLD SYNTAX - Traditional OOP-style context manager
class retry:
    def __enter__(self):
        print("Entering Context")

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Exiting Context")

# NEW SYNTAX - New contextlib-based context manager
import contextlib


@contextlib.contextmanager
def retry():
    print("Entering Context")
    yield
    print("Exiting Context")
````
