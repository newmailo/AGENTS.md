# AGENTS.md (Backend Subdirectory)

> 이 문서는 상위 AGENTS.md를 보완하며, `backend/` 하위에만 적용됩니다.

---

## 1) 적용 범위
- 적용 대상: `backend/` 및 모든 하위 디렉터리
- 우선순위: 상위 AGENTS와 충돌 시 이 파일 규칙 우선

---

## 2) 백엔드 영역 책임
- API 서버, 도메인 로직, 데이터 접근, 인증/인가, 배치/워커, 서버 설정
- 비즈니스 규칙은 가능한 서비스/도메인 계층에 유지
- 프레젠테이션(컨트롤러/핸들러) 계층은 얇게 유지

---

## 3) 설계 원칙
1. **명확한 계층 분리**
   - Handler/Controller ↔ Service/UseCase ↔ Repository/DAO
2. **안전한 변경**
   - 공개 API/스키마 변경 시 하위 호환성 검토
3. **신뢰성 우선**
   - 예외 처리, 타임아웃, 재시도, 멱등성 고려
4. **관측 가능성**
   - 구조화 로그, 핵심 메트릭, 트레이싱 포인트 유지
5. **보안 기본값**
   - 입력 검증, 권한 체크, 민감정보 마스킹

---

## 4) 디렉터리/파일 배치 규칙 (팀 구조에 맞게 조정)
- 엔트리포인트: `backend/src/main.*` 또는 `backend/cmd/*`
- API 레이어: `backend/src/api` 또는 `backend/src/handlers`
- 서비스/유즈케이스: `backend/src/services` 또는 `backend/src/usecases`
- 저장소/DB 접근: `backend/src/repositories`
- 도메인 모델: `backend/src/domain`
- 공통 유틸: `backend/src/common` 또는 `backend/src/lib`
- 테스트:
  - 단위 테스트: 소스 인접(`*.test.*`, `*_test.*`)
  - 통합 테스트: `backend/tests/integration`

---

## 5) API 변경 규칙
- API 계약(요청/응답/상태코드) 변경 시:
  1. 변경 사유 및 영향 범위 명시
  2. 문서(OpenAPI/README/API docs) 업데이트
  3. 기존 클라이언트 영향/마이그레이션 경로 제시
- 에러 응답 포맷 일관성 유지 (코드/메시지/추적 ID 등)

---

## 6) 데이터/DB 규칙
- 스키마 변경은 마이그레이션 파일로 관리
- 파괴적 변경(drop/rename) 시 단계적 마이그레이션 원칙:
  1. 확장(새 컬럼/테이블 추가)
  2. 이중 쓰기/읽기 전환
  3. 정리(구 구조 제거)
- 쿼리 성능 영향(N+1, 풀스캔, 인덱스) 점검

---

## 7) 에러 처리/로깅 규칙
- 사용자 응답 메시지와 내부 원인 로그 분리
- 민감정보(토큰, 비밀번호, PII) 로그 금지
- 에러 로그에는 최소한 아래 포함 권장:
  - 요청 식별자(request_id / trace_id)
  - 에러 코드/유형
  - 실패 지점(모듈/함수)

---

## 8) 테스트 규칙 (Backend)
가능하면 아래 순서로 실행:
1. 린트/정적 분석
2. 단위 테스트
3. 통합 테스트
4. 빌드/패키징

### 권장 명령 (프로젝트 기술스택에 맞는 것만 사용)
- Node:
  - `npm run lint`
  - `npm run test`
  - `npm run test:integration`
  - `npm run build`
- Python:
  - `ruff check .`
  - `pytest -q`
  - `pytest -m integration`
- Go:
  - `go test ./...`
  - `go vet ./...`
- Java/Kotlin:
  - `./gradlew test`
  - `./gradlew build`
- Rust:
  - `cargo test`
  - `cargo clippy -- -D warnings`
  - `cargo build`

---

## 9) 성능/안정성 체크리스트
- [ ] 타임아웃/재시도 정책 적절
- [ ] 핫패스 쿼리 인덱스 점검
- [ ] 캐시 키/TTL 정책 검토
- [ ] 대량 요청 시 자원 사용량 점검
- [ ] 멱등성 필요한 엔드포인트 검토

---

## 10) 리뷰 중점
1. 비즈니스 규칙 정확성
2. API 계약/호환성
3. 예외/경계값 처리
4. 보안(검증/권한/노출)
5. 테스트 커버리지와 품질

---

## 11) 금지 사항
- 테스트 없는 핵심 로직 변경
- DB 파괴적 변경을 단일 단계로 바로 반영
- 민감정보 로깅/하드코딩
- 요청 범위를 벗어난 광범위 리팩터링
