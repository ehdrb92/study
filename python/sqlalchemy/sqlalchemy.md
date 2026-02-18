SQLAlchemy는 Python용 SQL 도구이자 ORM이다. 엔터프라이즈 수준의 영속성 패턴의 완벽한 셋을 제공하며, 효율적이고 고성능의 데이터베이스 액세스를 위해 설계되었다.

SQLAlchemy는 두 가지 요소로 구성되어 있다.

- Core: SQL과 데이터베이스 통합의 기반 레이어
- ORM: Core 위에 구축된 객체 관계 매핑 레이어

## Core

### 특징
- SQL Expression Language: SQL 표현식을 구성 가능한 객체로 표현하는 시스템
- 스키마 중심 관점: 데이터베이스를 스키마 관점에서 다루며 불변성 지향
- 명령어 지향적: 명시적인 SQL 구문 작성과 실행

### 주요 기능
- SQL 표현식 구성: 복잡한 SQL 쿼리를 Python 객체로 표현
- 데이터베이스 연결 및 트랜잭션 관리
- DML 작업 (INSERT, UPDATE, DELETE)를 딕셔너리 매개변수와 함께 실행
- 결과셋 처리

### 사용해야할 때?
- 복잡한 SQL 쿼리가 필요한 경우
- 성능이 중요한 배치 처리
- 데이터베이스 스키마에 대한 세밀한 제어가 필요한 경우
- SQL에 익숙하고 직접적인 제어를 원하는 경우

```python
# Core 방식 - 스키마 중심의 접근
# SQL 표현식을 직접 구성하고 실행
engine = create_engine("sqlite:///example.db")
with engine.connect() as conn:
    result = conn.execute(
        text("SELECT * FROM users WHERE age > :age"),
        {"age": 25}
    )
```

## ORM

### 특징
- 도메인 중심 관점: 비즈니스 객체 모델을 데이터베이스 스키마에 매핑
- 상태 지향적: 객체의 상태 변화를 추적하고 자동으로 SQL로 변환
- 객체 지향 패러다임: 가변성에 의존하는 명시적인 객체 지향 접근

### 주요 기능
- Unit of Work 패턴: 비즈니스 객체의 상태 변화를 자동으로 INSERT, UPDATE, DELETE로 변환
- 자동화된 DML: 객체의 변경사항을 자동으로 데이터베이스에 반영
- 향상된 SELECT: ORM 특화된 자동화 및 객체 중심 쿼리 기능
- 관계 매핑: 테이블 간의 관계를 객체 간의 관계로 표현

### 사용해야할 때?
- 비즈니스 로직이 복잡한 애플리케이션
- 객체 지향 설계를 선호하는 경우
- 빠른 개발이 필요한 경우
- 테이블 간 관계가 복잡한 경우

```python
# ORM 방식 - 객체 중심의 접근
# 객체의 상태 변화가 자동으로 SQL로 변환됨
session = Session(engine)
user = User(name="홍길동", age=30)
session.add(user)
session.commit()  # 자동으로 INSERT 실행
```

## Engine

Engine은 SQLAlchemy 애플리케이션의 시작점이 되는 핵심 객체입니다.

### 주요 역할
- 중앙 연결 소스: 특정 데이터베이스에 대한 연결의 중심역할
- 연결 팩토리: 데이터베이스 연결을 생성하는 공장 역할
- 연결 풀 관리: Connection Pool을 통해 데이터베이스 연결을 효율적으로 관리

### 특징
- 글로벌 객체: 일반적으로 애플리케이션 전체에서 하나의 Engine 인스턴스 사용
- 특정 데이터베이스 서버당 하나: 각 데이터베이스 서버마다 별도의 Engine 생성

```python
# SQLite 인메모리 데이터베이스 사용
engine = create_engine("sqlite+pysqlite:///:memory:", echo=True)

# psycopg2 드라이버 사용
engine = create_engine("postgresql+psycopg2://user:password@localhost/dbname")

# asyncpg 드라이버 사용 (비동기)
engine = create_engine("postgresql+asyncpg://user:password@localhost/dbname")

# PyMySQL 드라이버
engine = create_engine("mysql+pymysql://user:password@localhost/dbname")

# mysqlclient 드라이버
engine = create_engine("mysql+mysqldb://user:password@localhost/dbname")
```

1. Dialect (데이터베이스 종류)
    - sqlite: 사용할 데이터베이스 종류
    - SQLAlchemy가 해당 데이터베이스와 통신하는 방법을 정의
2. DBAPI (데이터베이스 드라이버)
    - pysqlite: Python DBAPI 드라이버
    - 실제로는 Python 표준 라이브러리의 sqlite3 모듈
    - 생략 시 SQLAlchemy가 해당 데이터베이스의 기본 DBAPI 사용
3. 데이터베이스 위치
   - /:memory:: SQLite의 인메모리 데이터베이스 지시자
   - 실제 파일을 생성하지 않고 메모리에서만 동작

```python
# 연결 풀 설정 예시
engine = create_engine(
    "postgresql://user:password@localhost/dbname",
    pool_size=20,        # 연결 풀 크기
    max_overflow=30,     # 추가 연결 허용 수
    pool_timeout=30,     # 연결 대기 시간
    pool_recycle=3600    # 연결 재사용 시간
)

# 프로덕션 환경 고려사항
engine = create_engine(
    DATABASE_URL,
    echo=False,          # 프로덕션에서는 False
    pool_pre_ping=True,  # 연결 상태 확인
    connect_args={
        "check_same_thread": False  # SQLite 멀티스레드 지원
    }
)
```

## Transaction

SQLAlchemy의 데이터베이스 상호작용은 주로 Connection, Session 객체를 통해 이루어집니다.

### Core에서 트랜잭션

```python
# "Commit as you go" 패턴
# 블록 내에서 필요할 때마다 커밋
with engine.connect() as conn:
    conn.execute(text("CREATE TABLE some_table (x int, y int)"))
    conn.execute(
        text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
        [{"x": 1, "y": 1}, {"x": 2, "y": 4}]
    )
    conn.commit()  # 명시적 커밋
    
    # 추가 작업 후 다시 커밋 가능
    conn.execute(text("INSERT INTO some_table (x, y) VALUES (3, 9)"))
    conn.commit()

# "Begin once" 패턴 (권장)
# 전체 블록을 하나의 트랜잭션으로 처리
with engine.begin() as conn:
    conn.execute(
        text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
        [{"x": 6, "y": 8}, {"x": 9, "y": 10}]
    )
    # 블록 종료 시 자동 COMMIT (예외 발생 시 ROLLBACK)
```

### ORM에서 트랜잭션

```python
from sqlalchemy.orm import Session

# Core Connection과 유사한 패턴
stmt = text("SELECT x, y FROM some_table WHERE y > :y ORDER BY x, y")
with Session(engine) as session:
    result = session.execute(stmt, {"y": 6})
    for row in result:
        print(f"x: {row.x} y: {row.y}")

# Session에서 커밋
with Session(engine) as session:
    session.execute(
        text("UPDATE some_table SET y=:y WHERE x=:x"),
        [{"x": 9, "y": 11}, {"x": 13, "y": 15}]
    )
    session.commit()
```

## Metadata

메타데이터(Metadata)는 데이터베이스의 구조를 설명하는 정보입니다. SQLAlchemy에서는 테이블, 컬럼, 제약조건 등을 Python 객체로 표현합니다.

### Core 방식

핵심 구성요소는 다음과 같다.

- MetaData: 테이블들의 컬렉션을 관리하는 컨테이너
- Table: 데이터베이스 테이블을 표현하는 객체
- Column: 테이블의 컬럼을 표현하는 객체

```python
from sqlalchemy import MetaData

# MetaData 컬렉션 생성
metadata_obj = MetaData()

from sqlalchemy import Table, Column, Integer, String, ForeignKey

# user_account 테이블 정의
user_table = Table(
    "user_account",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("name", String(30)),
    Column("fullname", String),
)

# address 테이블 정의 (외래키 포함)
address_table = Table(
    "address",
    metadata_obj,
    Column("id", Integer, primary_key=True),
    Column("user_id", ForeignKey("user_account.id"), nullable=False),
    Column("email_address", String, nullable=False),
)
```

### ORM 방식

```python
from sqlalchemy.orm import DeclarativeBase

# Declarative Base 클래스 생성
class Base(DeclarativeBase):
    pass

# 자동 생성된 MetaData 확인
print(Base.metadata)  # MetaData()
print(Base.registry)  # <sqlalchemy.orm.decl_api.registry object>


# 매핑 클래스 정의
from typing import List, Optional
from sqlalchemy.orm import Mapped, mapped_column, relationship

class User(Base):
    __tablename__ = "user_account"
    
    # 타입 어노테이션과 함께 컬럼 정의
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(30))
    fullname: Mapped[Optional[str]]  # nullable 컬럼
    
    # 관계 정의
    addresses: Mapped[List["Address"]] = relationship(back_populates="user")
    
    def __repr__(self) -> str:
        return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"

class Address(Base):
    __tablename__ = "address"
    
    id: Mapped[int] = mapped_column(primary_key=True)
    email_address: Mapped[str]
    user_id = mapped_column(ForeignKey("user_account.id"))
    
    # 역방향 관계
    user: Mapped[User] = relationship(back_populates="addresses")
    
    def __repr__(self) -> str:
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"
```

## ORM 데이터 조작

SQLAlchemy ORM은 Unit of Work 패턴을 사용하여 객체의 상태 변화를 추적하고 자동으로 SQL 문으로 변환합니다. Session 객체가 이 모든 과정을 관리합니다. Session의 역할은 다음과 같습니다.

- 트랜잭션 관리: 데이터베이스 트랜잭션의 시작과 종료
- 객체 추적: Identity Map을 통한 객체 상태 관리
- 지연 실행: 필요한 시점에 SQL 문 실행
- 캐시 관리: 같은 Primary Key의 객체는 하나의 인스턴스만 유지

```python
# Unit of Work 패턴
from sqlalchemy.orm import Session

# Session 생성 및 사용
with Session(engine) as session:
    # 객체 생성 및 조작
    user = User(name="squidward", fullname="Squidward Tentacles")
    session.add(user)
    
    # 트랜잭션 커밋 시 실제 SQL 실행
    session.commit()
```

### 단계별 Session 객체의 상태

```python
# 1. 임시 상태
# 새로 생성된 객체, 아직 Session과 연결되지 않음
squidward = User(name='squidward', fullname='Squidward Tentacles')
print(squidward.id)  # None - 아직 데이터베이스에 저장되지 않음

# 2. 대기 상태
# Session에 추가되었지만 아직 데이터베이스에 저장되지 않음
session.add(squidward)
# 이 시점에서 squidward는 pending 상태

# 3. 영속 상태
# 데이터베이스에 저장되고 Session에서 관리되는 상태
session.commit()  # 이후 squidward는 persistent 상태
print(squidward.id)  # 1 - 자동 생성된 Primary Key

# 4. 분리 상태
# Session이 닫힌 후의 상태
session.close()
# 이제 squidward는 detached 상태
```