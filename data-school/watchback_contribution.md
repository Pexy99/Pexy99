# Watchback | 취약점 재평가 및 조치 자동화 플랫폼

> 외부 위험 분석을 조직의 보안 알림과 연결해, 라이브러리 업데이트 판단부터 저장소별 조치까지 지원하는 서비스

- 기간: 2026.08 ~ 2026.09
- 팀 구성: 6명 (데이터 엔지니어링 3명, 데이터 분석 2명, 본인)
- 담당: 팀 리딩, 전체 서비스 아키텍처 설계, 데이터 통합, 풀스택 개발
- 개발: Python, FastAPI, Next.js, Azure / 저장 및 분석: PostgreSQL, Microsoft Fabric

## 프로젝트와 본인 역할

- **문제:** 공개 이후 위험이 달라진 취약점을 다시 봐야 하지만, 저장소와 의존성 파일마다 쌓인 알림에서 같은 업데이트를 반복 검토
- **서비스:** 위험 변화 분석 결과를 조직의 미해결 알림과 연결하고, 검토부터 업데이트 PR과 해결 상태 확인까지 연결
- **본인 담당:** 초기 가설 검증, 전체 구조와 수집 전략 설계, 분석 결과를 검토와 조치로 연결하는 데이터 처리와 Web 서비스, AI 기능 구현
- **팀 담당:** 외부 위험 데이터의 수집 구현과 후보 분석 (데이터팀)

## 주요 기여

### 1. 데이터 수집과 동기화 설계

- 소스마다 제공 방식과 갱신 주기가 달라, EPSS는 날짜별 전체 스냅샷을 보존하고 GHSA·KEV는 변경분만 반영하도록 수집 전략 설계 (수집 구현은 데이터팀)
- Fabric 분석 결과와 GitHub 보안 알림을 PostgreSQL 운영 DB로 동기화하는 Worker 구현
- 고유 키 기반 upsert와 원천 갱신 시각 비교, PostgreSQL Advisory Lock과 트랜잭션으로 재실행·동시 실행·부분 실패에도 일관된 상태 유지
- **결과:** GHSA·KEV는 전체 재수집 없이 최신 상태 동기화, 동기화 재실행 시 추가 생성 0건
- 관련 PR: [#4](https://github.com/dt4-proj3-team3/watchback/pull/4) · [#7](https://github.com/dt4-proj3-team3/watchback/pull/7) · [#114](https://github.com/dt4-proj3-team3/watchback/pull/114) · [#127](https://github.com/dt4-proj3-team3/watchback/pull/127)

### 2. 분석 결과를 조치로 잇는 서비스

- 위험 분석은 취약점 단위, 알림은 저장소 단위, 조치는 의존성 파일 단위라 단계별 기준 단위를 나누고 서로 연결하는 데이터 모델 설계
- 원본 알림은 보존하면서 같은 라이브러리의 알림을 하나의 검토 항목으로 통합하고, 실제 업데이트는 저장소별로 관리
- FastAPI와 Next.js로 검토부터 위험 수용, 업데이트 PR 생성과 병합 후 해결 확인까지 구현
- **결과:** 전체 알림 406건 → 위험 변화 매칭 66건 → 검토·조치 단위 10건
- 관련 PR: [#110](https://github.com/dt4-proj3-team3/watchback/pull/110) · [#125](https://github.com/dt4-proj3-team3/watchback/pull/125) · [#128](https://github.com/dt4-proj3-team3/watchback/pull/128) · [#146](https://github.com/dt4-proj3-team3/watchback/pull/146)

### 3. 근거 기반 AI 기능

- 정형 데이터는 SQL로 조회하고 비정형 문서에만 벡터 검색을 쓰도록 나눠, 현재 화면과 조직에 맞는 근거만 LLM 컨텍스트로 구성
- LLM 호출 전 조직 범위를 다시 확인하고, 답변에 실제 사용한 근거 번호와 원문 링크 반환
- Azure OpenAI로 근거 요약을 일괄 생성해 프롬프트 버전과 함께 저장
- **결과:** 다른 조직의 근거가 섞이면 응답 생성 차단, 대시보드·재검토 건·패키지의 화면별 컨텍스트 구성
- 관련 PR: [#146](https://github.com/dt4-proj3-team3/watchback/pull/146)

## 그 밖의 기여

- 초기 PoC에서 고유 취약점 344개의 관측값 7,624건을 분석해 30개 취약점(8.7%)에서 급등 이벤트 42회 확인, 지속적 재평가의 근거로 활용
- Bicep으로 Azure 개발 환경을 구성하고 Key Vault와 Managed Identity로 비밀정보 전달, GitHub Actions와 OIDC로 컨테이너 배포 자동화 ([#124](https://github.com/dt4-proj3-team3/watchback/pull/124) · [#136](https://github.com/dt4-proj3-team3/watchback/pull/136) · [#138](https://github.com/dt4-proj3-team3/watchback/pull/138))

## 검증 범위

- PoC 결과는 단일 실험 저장소와 실험 기준이며, 실제 공격 증가를 측정한 것은 아님
- PR 조치 테스트에는 모의 GitHub 응답이 포함되며, 실제 검토 시간과 업데이트 성공률은 미측정
