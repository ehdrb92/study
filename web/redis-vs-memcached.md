## Redis vs Memcached

**Memcached는 캐시에만 집중한 단순한 도구**이고, **Redis는 캐시 이상의 역할을 하는 다목적 인메모리 데이터 스토어**입니다.

---

### 핵심 차이 요약

| 항목 | Redis | Memcached |
|------|-------|-----------|
| 자료구조 | String, Hash, List, Set, Sorted Set, Bitmap, HyperLogLog, Geospatial 등 | String (키-값 문자열)만 |
| 스레드 모델 | **싱글 스레드** (단일 코어 활용) | **멀티 스레드** (다중 코어 활용) |
| 데이터 영속성 | RDB 스냅샷, AOF 로그 지원 | 없음 (휘발성, 재시작 시 소멸) |
| 복제 / HA | Master-Replica 복제 + Sentinel/Cluster 내장 | 외부 도구 필요 |
| Pub/Sub | 지원 | 미지원 |
| 트랜잭션 | 지원 (MULTI/EXEC, Lua 스크립트) | 단일 atomic 명령만 |
| 메모리 효율 | 자료구조 오버헤드 있음 | Slab 할당으로 메모리 효율적 |
| 값 최대 크기 | 512MB | 기본 1MB (변경 가능) |
| 라이선스 (2025 기준) | AGPLv3 (Redis 8.0+) | BSD (완전 오픈소스) |

> ⚠️ **라이선스 이슈**: Redis 8.0부터 AGPLv3 라이선스가 적용되어, 코드 변경 시 오픈소스로 기여할 의무가 생기는 copyleft 조항이 포함되어 있습니다. 이 때문에 많은 기업에서 법적 위험을 이유로 사용을 제한하는 경우가 생겼고, 완전한 BSD 라이선스를 유지하는 Memcached가 다시 주목받는 상황입니다.

---

### Memcached의 특징

**장점**
- 멀티스레드 아키텍처 덕분에 멀티코어 CPU를 최대한 활용할 수 있어, 단순 대용량 캐싱에서 처리량이 높습니다.
- Slab 기반 메모리 할당 방식으로 메모리 단편화를 방지하고 메모리 효율을 높입니다.
- 단순한 구조 덕분에 학습 곡선이 낮고 운영 부담이 적습니다.

**단점**
- String 타입만 지원하므로, 예를 들어 세션 객체에서 특정 필드 하나만 수정하려면 역직렬화 → 수정 → 재직렬화 전체를 수행해야 합니다.
- 데이터 영속성이 없어 서버 재시작 시 캐시가 전부 날아갑니다.
- LRU 단일 eviction 정책만 지원합니다.

---

### Redis의 특징

**장점**
- String 외에도 List, Set, Sorted Set, Hash, Bitmap, HyperLogLog 등 다양한 자료구조를 지원해 더 복잡한 패턴을 네이티브하게 처리할 수 있습니다. 예를 들어 Sorted Set 하나로 실시간 리더보드를 구현할 수 있습니다.
- RDB/AOF를 통한 데이터 영속성으로 재시작 후에도 복구 가능합니다.
- Pub/Sub, Lua 스크립팅, 트랜잭션, 지리공간 쿼리 등 부가 기능이 풍부합니다.
- 6가지 eviction 정책을 지원합니다: noeviction, allkeys-lru, volatile-lru, allkeys-random, volatile-random, volatile-ttl.

**단점**
- 싱글 스레드 구조라 멀티코어 활용이 제한적입니다.
- 자료구조 오버헤드로 메모리 사용량이 더 많을 수 있습니다.
- AGPLv3 라이선스 이슈 (Redis 8.0+).

---

### 모범 사용 사례

**Memcached를 선택할 때**
- HTML 페이지 캐싱, DB 쿼리 결과 캐싱 등 단순 문자열 데이터 캐싱
- 멀티코어 서버에서 순수 캐시 처리량을 극대화하고 싶을 때
- 데이터 손실이 허용되고 기능보다 단순함/가벼움이 우선일 때
- 이미 Memcached 기반 레거시 인프라가 있을 때

**Redis를 선택할 때**
- 세션 관리, 사용자 인증 토큰 저장 (TTL 자동 만료)
- 실시간 랭킹/리더보드 (Sorted Set 활용)
- Rate Limiting, 분산 락 (Distributed Lock)
- Pub/Sub 기반 실시간 알림, 채팅, 피드
- 메시지 큐 (Lists 또는 Redis Streams 활용)
- 지리공간 검색 (가까운 매장 찾기 등)

---

### 간단한 예시 코드

```python
import redis
import pymemcache.client.base as memcache

# Redis - Sorted Set으로 실시간 랭킹 구현
r = redis.Redis(host='localhost', port=6379)
r.zadd('leaderboard', {'alice': 1500, 'bob': 2300, 'carol': 1800})
top3 = r.zrevrange('leaderboard', 0, 2, withscores=True)
print(top3)  # [('bob', 2300.0), ('carol', 1800.0), ('alice', 1500.0)]

# Redis - Hash로 세션 일부 필드만 수정
r.hset('session:user123', 'last_page', '/checkout')  # 다른 필드는 그대로 유지

# Memcached - 단순 문자열 캐싱만 가능
mc = memcache.Client(('localhost', 11211))
mc.set('page_cache:/home', '<html>...</html>', expire=300)
html = mc.get('page_cache:/home')
```

---

### 결론

Redis는 복잡한 데이터 구조와 추가 기능이 필요한 다양한 사용 사례에 적합하고, Memcached는 단순하고 멀티스레드 특성을 활용한 기본 캐싱 요구에 뛰어납니다.

오늘날 대부분의 신규 프로젝트에서 **Redis가 기본 선택**이 되는 이유는, 캐시만 쓰더라도 나중에 Pub/Sub이나 세션 스토어 같은 기능을 추가하기 쉽기 때문입니다. Memcached가 유리한 케이스는 순수 고처리량 단순 캐시 + 멀티코어 최대 활용이 필요한 매우 특정한 상황으로 점점 좁아지고 있습니다.

**참고 링크**
- [AWS: Redis OSS vs Memcached](https://aws.amazon.com/elasticache/redis-vs-memcached/)
- [Kinsta: Memcached vs Redis](https://kinsta.com/blog/memcached-vs-redis/)
- [ImaginaryCloud: Redis vs Memcached 심층 분석](https://www.imaginarycloud.com/blog/redis-vs-memcached)
- [ScaleGrid: Redis vs Memcached 2025](https://scalegrid.io/blog/redis-vs-memcached/)