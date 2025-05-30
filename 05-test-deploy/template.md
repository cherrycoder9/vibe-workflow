# 05 – Test & Deploy Template: LLM 테스트 및 배포 파이프라인 설계 가이드 🚀

> 이 파일은 테스트 전략, CI/CD, 배포 방식, 운영 모니터링 등 서비스 운영 전 과정을 설계하기 위한 질문 목록입니다. 직접 수정하지 말고, 각 항목에 답하며 최종 계획을 `finalized.md`에 정리하세요.

---

## 🚀 시작하기

* **준비:** `00-planning`, `01-tech-stack`, `03-dev-guidelines` 결과를 검토하세요.
* **호출 예시:**

  > “테스트 및 배포 전략 설정을 시작해 주세요.”

---

## 1. 종합 테스트 전략 수립

### 질문

* 개발 중, CI, 배포 전 등 단계별로 어떤 테스트를 어떻게 수행할까요?

### 옵션

* 🤖 AI 추천 (프로젝트 특성 기반)
* 단위 테스트, 통합 테스트, E2E 테스트
* 성능 테스트: 로드/스트레스
* 보안 테스트: SAST, DAST, 취약점 스캔
* 사용성 테스트, 기타: \_\_\_\_\_\_\_\_\_\_

---

## 2. CI 트리거 조건

### 질문

* CI는 어떤 조건에서 실행할까요? (모든 Push, PR 시, 특정 브랜치, 스케줄, 수동 등)

### 옵션

* 🤖 AI 추천 (워크플로우 및 비용 효율화)
* 모든 Push 시, PR 생성/업데이트 시
* `main`/`develop` 브랜치에 Push/Merge 시
* 스케줄링(예: 매일 자정)
* 수동 트리거, 기타: \_\_\_\_\_\_\_\_\_\_

---

## 3. 주요 테스트 및 자동화 도구

### 질문

* 각 테스트 유형별로 어떤 도구를 사용할까요?

### 옵션

* 🤖 AI 추천 (기술 스택 기반)
* 단위/통합: Jest, Vitest, pytest 등
* E2E: Playwright, Cypress, Selenium
* 성능: k6, Locust
* 보안: OWASP ZAP, Snyk
* Mocking: Mockito, Nock
* 커버리지: Istanbul, Coverage.py
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 4. 배포 환경 구성

### 질문

* 개발·스테이징·프로덕션 환경은 어떤 인프라에서 운영할까요?

### 옵션

* 🤖 AI 추천 (확장성·운영 편의 고려)
* 컨테이너: Kubernetes(EKS/GKE), Docker Swarm
* 서버리스: Lambda, Cloud Functions
* PaaS: Heroku, Vercel
* IaaS: VMs, 베어메탈
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 5. 배포 자동화 & IaC 연계

### 질문

* CI 도구와 IaC(코드형 인프라)를 어떻게 연동해 자동 배포할까요?

### 옵션

* 🤖 AI 추천 (CI·배포 환경 기반)
* GitHub Actions, GitLab CI, Jenkins
* GitOps: Argo CD, Flux CD
* IaC: Terraform, Pulumi, CloudFormation
* 워크플로우 예시: PR→CI→이미지 빌드→레지스트리→스테이징→승인→프로덕션
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 6. 배포 전략 & 롤백 계획

### 질문

* 무중단 배포 및 롤백은 어떻게 설계할까요?

### 옵션

* 🤖 AI 추천 (안정성 기반)
* 블루/그린, 카나리, 롤링 업데이트
* 기능 플래그, Git 태그 롤백
* DB 마이그레이션 전략
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 7. 운영 모니터링 & 알림

### 질문

* 어떤 지표를 모니터링하고, 어떻게 알림을 받을까요?

### 옵션

* 🤖 AI 추천 (KPI·장애 감지 기반)
* 지연 시간, 트래픽, 오류율, 포화도
* 도구: Prometheus+Grafana, Datadog, Sentry
* 로그: ELK, Grafana Loki
* 알림: Slack, PagerDuty
* 기타: \_\_\_\_\_\_\_\_\_\_

---

## 8. IaC 도구 선택

### 질문

* 어떤 IaC 도구로 인프라를 코드로 관리할까요?

### 옵션

* 🤖 AI 추천 (클라우드 환경 기반)
* Terraform, Pulumi
* CloudFormation, ARM, Deployment Manager
* Ansible, Chef
* 기타: \_\_\_\_\_\_\_\_\_\_

---

> 위 항목을 따라 대화를 진행하고, 최종 전략은 `finalized.md`에서 확인하세요.
