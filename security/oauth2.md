OAuth2란 무엇인가? 웹 개발 분야에 있으면서 많이 들어봤지만 실상 제대로 알고 사용한 적은 없다.

OAuth2는 사용자가 자신의 비밀번호를 직접 공유하지 않고도 제3자 애플리케이션에 자원 접근 권한을 부여할 수 있게 해주는 인증 및 권한 부여 프레임워크이다.

## OAuth2의 탄생 배경

기존 방식의 한계점
- 사용자가 제3자 앱에 자신의 아이디/비밀번호를 직접 제공해야 했음
- 비밀번호가 노출되면 모든 계정 권한이 위험해짐
- 특정 기능만 허용하고 싶어도 전체 계정 접근권을 줘야 했음
- 권한 회수가 어려워 비밀번호를 변경해야 했음

이러한 보안 취약점과 사용자 경험 문제를 해결하기 위해 OAuth 1.0이 2007년에 등장했고, 이후 더 간단하고 유연한 OAuth 2.0이 2012년에 RFC 6749로 표준화

## OAuth2의 동작 방식

핵심 구성요소
- Resource Owner: 자원의 소유자 (사용자)
- Client: 자원에 접근하려는 애플리케이션
- Authorization Server: 인증 및 토큰 발급 서버
- Resource Server: 실제 자원(API)을 제공하는 서버

기본 흐름
- 클라이언트가 사용자에게 권한 요청
- 사용자가 인증 서버에서 로그인하고 권한 승인
- 인증 서버가 인증 코드(Authorization Code) 발급
- 클라이언트가 인증 코드로 액세스 토큰 요청
- 클라이언트가 액세스 토큰으로 리소스 서버에 API 호출

## 실제 사용 예시

일상적인 예시
- 모바일 앱에서 "Google로 로그인" 기능
- 웹사이트에서 "Facebook 계정으로 가입" 기능
- Spotify가 Instagram에 현재 듣는 음악 공유
- 배달 앱이 Google Maps API를 사용해 위치 정보 활용

기업 환경 예시
- 직원이 Slack에서 Google Drive 파일 공유
- CRM 시스템이 Gmail API로 이메일 발송
- 프로젝트 관리 도구가 GitHub 저장소 연동

## 보안 고려 사항

- 토큰 만료 시간 설정
- Refresh Token 관리
- CSRF 공격 방지
- 적절한 Scope 설정
- 토큰 Blacklist
- HTTPS 필수

## 데이터 전송 매개체

- JWT
- Session 기반 인증: 서버 메모리/DB에 세션 저장
- Opaque Token: 의미 없는 랜덤 토큰 + 서버 검증
- Stateless Session: 쿠키 기반 서명된 세션

## 확장 고려사항

- Role-Based Access Control (RBAC): 사용자 역할별 권한 관리
- API Rate Limiting: 사용자별 API 호출 제한
- Audit Logging: 인증/인가 로그 기록


## OAuth2 Grant Type??

