## `__new__` 메서드를 이용한 전통적인 싱글톤 패턴

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        """
        객체 생성을 담당하는 메서드입니다.
        _instance가 비어있을 때만 super().__new__를 호출하여 실제 객체를 생성하고,
        그렇지 않으면 이미 생성된 _instance를 반환합니다.
        """
        if not cls._instance:
            print("--- 새로운 인스턴스를 생성합니다 ---")
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, name):
        """
        객체 초기화를 담당하는 메서드입니다.
        주의: 싱글톤이라도 __new__가 인스턴스를 반환하면 __init__은 매번 호출됩니다.
        따라서 초기화가 한 번만 실행되도록 방어 코드를 넣는 것이 좋습니다.
        """
        if not hasattr(self, '_initialized'):
            self.name = name
            self._initialized = True
            print(f"--- 초기화 실행: {name} ---")
        else:
            print(f"--- 이미 초기화된 객체입니다. (입력된 name '{name}' 무시됨) ---")

    def get_name(self):
        return self.name


# --- 테스트 코드 ---
if __name__ == "__main__":
    # 1. 첫 번째 객체 생성
    s1 = Singleton("First Instance")
    print(f"s1: {s1}, name: {s1.get_name()}")
    print()

    # 2. 두 번째 객체 생성 시도
    s2 = Singleton("Second Instance")
    print(f"s2: {s2}, name: {s2.get_name()}")
    print()

    # 3. 객체 비교
    print(f"s1과 s2는 동일한 객체인가? {s1 is s2}")
    
    # 4. s2에서 값을 바꿔도 s1도 영향을 받음 (같은 객체이므로)
    s2.name = "Changed Name"
    print(f"s1의 이름도 변경되었나? {s1.get_name()}")

"""
--- 새로운 인스턴스를 생성합니다 ---
--- 초기화 실행: First Instance ---
s1: <__main__.Singleton object at 0x7fbd5e8798d0>, name: First Instance

--- 이미 초기화된 객체입니다. (입력된 name 'Second Instance' 무시됨) ---
s2: <__main__.Singleton object at 0x7fbd5e8798d0>, name: First Instance

s1과 s2는 동일한 객체인가? True
s1의 이름도 변경되었나? Changed Name
=== 코드 실행 완료 ===
"""
```

## `@cache` 를 활용한 간단한 싱글톤 패턴

```python
from functools import cache

@cache
class SingletonWithCache:
    """
    @cache 데코레이터를 클래스에 직접 적용합니다.
    이 방식은 엄밀히 말하면 '인자가 같을 때' 동일한 객체를 반환하는 
    Multiton(Flyweight) 패턴에 가깝지만, 
    인자를 고정하거나 사용하지 않으면 싱글톤처럼 동작합니다.
    """
    def __init__(self, name):
        # __new__ 방식과 달리, 캐시된 객체가 반환되면 __init__은 다시 호출되지 않습니다.
        self.name = name
        print(f"--- 초기화 실행 (실제 객체 생성): {name} ---")

    def get_name(self):
        return self.name

if __name__ == "__main__":
    print("1. 동일한 인자('A')로 객체 생성")
    s1 = SingletonWithCache("A")
    s2 = SingletonWithCache("A")
    
    print(f"s1: {s1}")
    print(f"s2: {s2}")
    print(f"s1 is s2? {s1 is s2}")  # True (캐시된 객체 반환)
    print()

    print("2. 다른 인자('B')로 객체 생성 (주의!)")
    # functools.cache는 인자가 다르면 새로운 키로 인식하여 새 객체를 생성합니다.
    s3 = SingletonWithCache("B")
    
    print(f"s3: {s3}")
    print(f"s1 is s3? {s1 is s3}") # False (다른 객체 생성됨)
```

## 두 방식을 비교

- `__new__` 오버라이드
  - 상대적으로 복잡
  - 객체를 재사용해도 `__init__` 이 매번 호출되므로 방어 코드 필요
  - 인자가 달라도 무조건 하나의 인스턴스만 유지
  - 원본 클래스 타입 유지
  - 별도의 락 처리가 없으면 스레드 불안정의 가능성 존재
- `@cache` 데커레이터
  - 매우 간결함
  - `__init__` 이 중복 호출될 가능성 없음
  - 인자가 다르면 다른 객체가 생성(Multiton 패턴)
  - 원본 클래스를 감싼 래퍼 타입으로 변환됨
  - `functools` 내부는 C로 구현되어 있어 기본적으로 스레드 안전