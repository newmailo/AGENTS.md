# AGENTS.md (Infra Subdirectory)

> 이 문서는 상위 AGENTS.md를 보완하며, `infra/` 하위에만 적용됩니다.

---

## 1) 적용 범위
- 적용 대상: `infra/` 및 모든 하위 디렉터리
- 우선순위: 상위 AGENTS와 충돌 시 이 파일 규칙 우선

---

## 2) 인프라 영역 책임
- IaC(Terraform/Pulumi/CloudFormation 등), 배포 파이프라인, 컨테이너/클러스터 설정, 네트워크/보안 정책, 관측성 구성
- 인프라 변경은 **가시성 + 복구 가능성 + 점진적 적용**을 기본으로 함

---

## 3) 핵심 원칙
1. **불변성과 선언형**
   - 수동 콘솔 변경보다 코드 기반(IaC) 우선
2. **최소 권한**
   - IAM/RBAC는 필요한 권한만 부여
3. **안전한 배포**
   - 계획(Plan) 확인 후 적용(Apply)
4. **복구 가능성**
   - 롤백 전략/재배포 경로 확보
5. **감사 가능성**
   - 누가, 무엇을, 왜 바꿨는지 PR/커밋에 명시

---

## 4) 디렉터리/파일 배치 규칙 (예시, 팀 구조에 맞게 조정)
- 환경 분리:
  - `infra/environments/dev`
  - `infra/environments/staging`
  - `infra/environments/prod`
- 공통 모듈:
  - `infra/modules/*`
- 배포 파이프라인:
  - `infra/ci/*` 또는 `.github/workflows/*`(루트에 존재할 수도 있음)
- 런북/운영 문서:
  - `infra/runbooks/*`

---

## 5) 변경 절차 (필수)
1. 변경 목적/영향 범위 정의
2. IaC 포맷/린트/검증 수행
3. Plan 생성 및 검토
4. 승인 기준 충족 시 Apply
5. 배포 후 상태 검증(헬스체크/메트릭/로그)
6. 결과 및 롤백 포인트 기록

---

## 6) 환경별 정책
- `prod`는 가장 엄격:
  - 직접 변경 금지(원칙)
  - 승인/리뷰 강화
  - 변경 시간대/점검창 고려
- `dev/staging`은 검증 목적:
  - 실험 가능하지만 파괴적 변경은 사전 공지
- 환경별 변수/시크릿은 절대 하드코딩 금지

---

## 7) 보안 규칙
- 비밀정보는 Secret Manager/Vault/환경변수 참조 사용
- 시크릿 값 평문 커밋 금지
- 외부 공개 설정(포트/버킷/정책)은 기본 deny 원칙
- 보안그룹/방화벽 규칙은 출처 CIDR와 목적을 명시

---

## 8) 상태(State) 관리 규칙 (Terraform 등)
- 원격 state 사용 권장 (락 활성화)
- state 파일 직접 수동 편집 금지
- state drift 감지 시 원인 파악 후 코드로 수렴
- import/move 작업은 기록(명령 + 배경) 필수

---

## 9) 테스트/검증 규칙 (Infra)
가능하면 아래 순서:
1. 포맷/린트
2. 정적 정책 검사
3. plan
4. apply(승인 후)
5. 사후 검증

### 권장 명령 (도구별 선택)
- Terraform:
  - `terraform fmt -check -recursive`
  - `terraform validate`
  - `terraform plan -out=tfplan`
  - `terraform apply tfplan`
- Kubernetes:
  - `kubectl diff -f ...`
  - `kubectl apply --server-side --dry-run=server -f ...`
- Helm:
  - `helm lint ...`
  - `helm template ...`
- Policy:
  - `conftest test ...`
  - `checkov -d .`

---

## 10) 배포/롤백 규칙
- 배포 전:
  - 변경 범위, 다운타임 여부, 롤백 방법 명시
- 배포 후:
  - 핵심 SLI/SLO 지표 확인
  - 에러율/지연/리소스 사용량 확인
- 문제 발생 시:
  - 즉시 롤백 또는 트래픽 차단/우회
  - 사고 기록(원인/영향/재발 방지책) 남김

---

## 11) 리뷰 중점
1. 최소 권한 원칙 준수 여부
2. 파괴적 변경 위험도
3. 멀티 환경 일관성
4. 재현 가능성(코드/명령 기반)
5. 운영 관측성(로그/메트릭/알람) 확보 여부

---

## 12) 금지 사항
- 콘솔 수동 변경 후 코드 미반영
- 시크릿 평문 커밋
- Plan 검토 없이 Apply
- 프로덕션 환경 무승인 직접 변경
