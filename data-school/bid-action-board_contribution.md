# Bid Action Assistant | 공공조달 입찰 기회 매칭 플랫폼

> 시설관리 기업이 관련 공고를 찾고 첨부문서의 참가 조건을 검토한 뒤, 부서별 제출 준비로 이어가도록 지원하는 플랫폼

- 기간: 2026.05 (2주)
- 팀 구성: 6명 (수집, 분석, 대시보드, PM, 문서 담당 각 1명, 본인)
- 담당: 팀 리딩, 아키텍처 설계, 데이터 및 LLM 엔지니어링
- 개발: Python, Azure Functions, Azure OpenAI, Azure AI Search / 저장: PostgreSQL, Blob Storage

## 프로젝트와 본인 역할

- **문제:** 입찰 담당자가 공고문과 업무 범위 설명서에 흩어진 참가 자격과 제출 요건을 수작업으로 확인
- **서비스:** 관련 공고 탐색과 문서 구조화, 회사 자격 비교, 원문 질의응답과 부서별 준비 업무 연결
- **본인 담당:** 수집과 문서 처리 파이프라인, LLM 요건 추출, RAG 질의응답, 분석팀 추천 코드의 실행 환경 통합, 인프라 구성
- **공동 작업:** 추천과 준비 업무 생성의 초기 알고리즘, 대시보드

## 주요 기여

### 1. 공고 수집과 문서 처리 파이프라인

- 기간별 전체 목록 수집은 관련 없는 공고까지 건별 면허·첨부 조회를 일으켜, 제목 검색 API를 검증하고 18개 키워드로 수집 단계에서 선별
- 공고번호와 차수 기준 중복 제거, HWP/HWPX/PDF 파싱과 검색용 문서 조각 생성, 수집 이후 5단계 처리를 Timer Trigger로 자동 실행
- **결과:** 후속 처리 대상 15,894건 → 779건 (95.1% 축소), 공고별 보조 API 잠재 호출 최대 31,788회 → 1,558회
- 관련 PR: [#131](https://github.com/dataschool4-team2/bid-action-board/pull/131) · [#181](https://github.com/dataschool4-team2/bid-action-board/pull/181) · [#72](https://github.com/dataschool4-team2/bid-action-board/pull/72) · [#191](https://github.com/dataschool4-team2/bid-action-board/pull/191)

### 2. LLM 요건 추출

- 비용 기준으로 고른 초기 모델이 실제 공고문에서 조건을 추출하지 못해, 같은 첨부문서로 모델을 비교하고 gpt-4.1-mini로 재선정
- 업체 소재지 제한 여부의 판별 기준과 정답·오답 예시를 프롬프트에 추가하고, 원문 인용 검증과 면허코드 기준 중복 제거 적용
- **결과:** 같은 공고 2건의 조건 추출 0 → 7건, 0 → 19건 / 지역 조건 오분류 3건 교정, 원문 대신 ‘확인 필요’로 채운 근거 2건 → 0건
- 관련 PR: [#80](https://github.com/dataschool4-team2/bid-action-board/pull/80) · [#123](https://github.com/dataschool4-team2/bid-action-board/pull/123) · [#169](https://github.com/dataschool4-team2/bid-action-board/pull/169) · [#177](https://github.com/dataschool4-team2/bid-action-board/pull/177)

### 3. 근거 기반 RAG와 서비스 연결

- Azure AI Search로 선택한 공고의 문서만 검색하고, 답변에 사용한 원문과 출처 파일을 함께 표시
- 의미 검색에서 자격 코드와 등록기한이 누락돼, 키워드 검색과 하이브리드 검색 결과를 병합하고 답변에 코드와 날짜를 포함하도록 보강
- 분석팀 추천 코드를 Azure Functions에 통합해 면허·소재지 판정을 보완하고, 대시보드 질의응답과 Teams 알림 연결
- **결과:** 대표 공고 질문에서 누락되던 업종 식별 코드와 등록기한 포함
- 관련 PR: [#171](https://github.com/dataschool4-team2/bid-action-board/pull/171) · [#175](https://github.com/dataschool4-team2/bid-action-board/pull/175) · [#119](https://github.com/dataschool4-team2/bid-action-board/pull/119) · [#121](https://github.com/dataschool4-team2/bid-action-board/pull/121) · [#219](https://github.com/dataschool4-team2/bid-action-board/pull/219)

## 그 밖의 기여

- Bicep으로 8개 서비스 영역(리소스 17개)을 코드화하고 GitHub Actions로 컨테이너 빌드와 배포 자동화 ([#53](https://github.com/dataschool4-team2/bid-action-board/pull/53) · [#90](https://github.com/dataschool4-team2/bid-action-board/pull/90))
- 이슈 템플릿과 완료 기준, ADR 운영 체계 수립으로 6인 팀의 작업 분배와 검증 기준 정리

## 검증 범위

- 95.1%는 검색 결과 건수 기준이며, 회사 적합 판정 건수나 시간 절감률이 아님. 검색에서 놓친 관련 공고 비율은 미측정
- 추출 개선은 해당 문서 기준이며 전체 추출 정확도, 검색 방식과 프롬프트 변경의 개별 효과는 미측정
