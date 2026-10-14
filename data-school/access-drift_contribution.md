# Access Drift | 오프보딩 이후 잔여 접근권한 탐지 및 조치 지원 플랫폼

> 서로 다른 시스템의 계정과 권한을 연결해 퇴사 이후 남은 접근 경로를 찾고, 보안 담당자의 조치를 지원하는 플랫폼

- 기간: 2026.06 ~ 2026.07
- 팀 구성: 5명 (DE 2명(본인 포함), DS 2명, PO 겸 대시보드 1명)
- 담당: 팀 리딩, 아키텍처 설계, 데이터 엔지니어링
- 개발: Python, PySpark, Azure Databricks / 저장: Delta Lake, ADLS Gen2

## 프로젝트와 본인 역할

- **문제:** 회사 로그인 계정을 차단해도 직원이 관리하던 자동화 계정의 인증키와 접근권한이 남을 수 있음
- **서비스:** 인사 정보와 각 서비스의 계정 정보를 연결해 남은 접근 경로를 탐지하고, 정책 문서를 참고한 조치 방향 제공
- **본인 담당:** 공통 데이터 모델과 파이프라인 설계·구현, 품질 검증 체계 설계와 관계 검증, DLT 적재 전환, RAG 근거 데이터 구성
- **팀원 보조:** Bronze·Silver DLT expectation과 스키마 계약 CI 구현
- **팀 담당:** 탐지 룰과 프롬프트 (DS), 대시보드 (PO)

## 주요 기여

### 1. 공통 접근 모델과 파이프라인

- 시스템마다 다른 계정·권한 표현을 접근 주체, 인증 정보, 자산, 관계의 4개 공통 엔티티로 정규화해 직접·간접 접근을 같은 모델로 표현
- 원본을 보존하는 메달리온 파이프라인을 PySpark로 구현하고, 실제 조직 환경 대신 API 응답 형식을 모사한 합성 데이터 생성기 구성
- GitHub 앱의 소유자가 사람 계정에 연결되지 않아 퇴사 여부를 판단하지 못하는 문제를 찾아 소유자 이메일로 소유 관계 연결
- **결과:** 7개 시스템 16개 데이터 객체 → 4개 공통 엔티티, 누락됐던 소유 관계 5건 복원
- 관련 PR: [#39](https://github.com/dataschool-proj2-team5/access-drift/pull/39) · [#45](https://github.com/dataschool-proj2-team5/access-drift/pull/45) · [#46](https://github.com/dataschool-proj2-team5/access-drift/pull/46) · [#89](https://github.com/dataschool-proj2-team5/access-drift/pull/89)

### 2. 데이터 품질 검증 체계

- 변환 중, 적재 후, 코드 변경 시점의 오류가 서로 달라 DLT 품질 게이트, 적재 후 SQL 검증, 스키마 계약 CI로 나눈 검증 체계 설계
- 형식은 맞지만 의미상 성립하지 않는 관계(비인간 계정이 사람 계정을 소유하는 경우 등)를 품질 게이트 규칙으로 추가하고, 오류를 의도적으로 주입해 실제 탐지 여부 검증
- 기존 조회 경로를 유지하며 DLT 기반 적재로 전환하고, 12개 태스크 DAG로 검증 실패 시 후속 작업 차단
- **결과:** 주입한 스키마·데이터·관계 오류 37건 전부 탐지, 검사 대상의 중복 또는 빈 식별자 0건
- 관련 PR: [#60](https://github.com/dataschool-proj2-team5/access-drift/pull/60) · [#64](https://github.com/dataschool-proj2-team5/access-drift/pull/64) · [#108](https://github.com/dataschool-proj2-team5/access-drift/pull/108) · [#111](https://github.com/dataschool-proj2-team5/access-drift/pull/111)

### 3. RAG 근거 코퍼스

- 문서 유사도에만 의존하면 역할이 다른 문서가 섞여, 정책·Playbook·사례·API 가이드에서 근거를 직접 선별하고 리스크·시스템·조치 유형 태깅
- 근거 식별자와 출처, 변경 이력을 저장해 답변을 원문까지 추적할 수 있게 하고, 인증정보 원문과 중복 데이터는 적재 전에 차단
- **결과:** 근거 12개 청크 적재(중복 0개), 대표 리스크 케이스의 근거 후보 9개 확인, 평가 시나리오 7건과 정답 근거 설계
- 관련 PR: [#79](https://github.com/dataschool-proj2-team5/access-drift/pull/79) · [#48](https://github.com/dataschool-proj2-team5/access-drift/pull/48)

## 그 밖의 기여

- 탐지 결과를 다시 만들어도 담당자의 검토 기록이 남도록 최신 검토 상태와 변경 이력을 별도 저장하고 화면에 연동 ([#52](https://github.com/dataschool-proj2-team5/access-drift/pull/52) · [#70](https://github.com/dataschool-proj2-team5/access-drift/pull/70) · [#88](https://github.com/dataschool-proj2-team5/access-drift/pull/88))
- Bicep과 Databricks Asset Bundle로 실행 환경을 구성하고, GitHub Actions로 Databricks Apps 배포 자동화 ([#2](https://github.com/dataschool-proj2-team5/access-drift/pull/2) · [#28](https://github.com/dataschool-proj2-team5/access-drift/pull/28) · [#94](https://github.com/dataschool-proj2-team5/access-drift/pull/94))

## 검증 범위

- 결과는 소규모 합성 데이터 기준이며, 실제 고객사 적용이나 정확도 향상률, 시간 및 비용 절감률은 미측정
- 소유 관계 복원은 해당 합성 입력의 결과이며, 모든 서비스의 동일인 식별을 해결한 것은 아님
