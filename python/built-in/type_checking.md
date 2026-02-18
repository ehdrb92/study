파이썬의 `TYPE_CHECKING`을 이용하면 어떻게 순환 임포트 문제가 해결되는가?

`TYPE_CHECKING`은 `typing` 모듈의 특별한 상수이다. 타입 체커가 실행될 때는 값이 True이지만, 런타임에는 항상 False이다.

두 모듈이 다음과 같이 작성되어있다고 생각해보자.

```python
# user.py
from post import Post  # 런타임에 임포트 에러!

class User:
    def __init__(self):
        self.posts: list[Post] = []
```

```python
# post.py  
from user import User  # 런타임에 임포트 에러!

class Post:
    def __init__(self, author: User):
        self.author = author
```

타입 힌트에 사용하기위해 서로의 모듈에서 클래스 임포트가 필요한데 런타임에서 위와 같이 작성되면 파이썬은 순환 임포트 에러를 발생시킨다.

이를 해결하기 위해서 `TYPE_CHECKING`을 사용하면 된다. 예시를 보자.

```python
# user.py
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from post import Post  # 타입 체킹 시에만 임포트

class User:
    def __init__(self):
        self.posts: list['Post'] = []  # 문자열 어노테이션 사용
```

```python
# post.py
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from user import User  # 타입 체킹 시에만 임포트

class Post:
    def __init__(self, author: 'User'):
        self.author = author
```