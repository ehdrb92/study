ORM은 Object-Relational Mapping의 약자로 관계형 데이터베이스의 테이블을 객체로 다루기 위한 매핑 규칙이다.

ORM을 쓰지 않고 순수 SQL로 데이터베이스 쿼리를 한다면?

```python
cursor.execute(
    "SELECT id, email, created_at FROM users WHERE id = %s",
    (user_id,),
)
row = cursor.fetchone()

user = {
    "id": row[0],
    "email": row[1],
    "created_at": row[2],
}
```

위는 psycopg2와 같은 DB-API 2.0 규격을 사용하는 예시이다. 이렇게 되면 다음 문제점이 생길 수 있다.

- SQL 문자열이 코드 여기저기 직접 작성됨
- 특정 컬럼 이름이 변경되면 모든 SQL 문자열 수정될 가능성 생김
- 타입 안정성이 낮음(예를 들어 row[0]이 정확히 무엇인지 SQL을 확인해야함 실수 가능성 존재)

ORM은 다음과 같이 데이터베이스의 객체와 애플리케이션의 객체를 매핑시켜서 애플리케이션 코드로 데이터베이스 객체를 다룰 수 있게 해준다.

- 테이블 <-> 클래스
- 컬럼 <-> 속성
- 행 <-> 객체
