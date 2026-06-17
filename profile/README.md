# TrustGate

> **릴리스 계보 검증을 통한 공급망 공격 차단 도구**


---

## 멤버 구성
임성빈(PL) 정현진(PM) 김민재(PA) 정일재(PA)

## 한 줄 소개

패키지를 설치하기 전에, 그 릴리스가 정당한 경로(리뷰된 PR · 정상 워크플로)로 만들어졌는지 검증해 공급망 공격을 차단합니다.

## 프로젝트 배경

2026년의 대형 공급망 공격(Miasma, TanStack 등)은 attestation이 없어서가 아니라, 유효한 서명을 달고도 발생했습니다. 공격자는 훔친 자격증명으로 정상 배포 경로를 타고 악성 버전을 올렸고, 서명·OIDC·provenance가 전부 진짜였습니다.

기존 도구(`slsa-verifier` 등)는 "어디서 빌드됐나" 는 검증하지만, 그 빌드가 정당한 거버넌스를 거쳤는지는 검증하지 못합니다.

> TrustGate는 서명 너머의 거버넌스 경로 를 봅니다.

## 무엇을 검사하나

릴리스의 계보를 복원합니다:

```
아티팩트 → 워크플로 → 커밋 → PR → 리뷰어 → OIDC 신원
```

그리고 다음 신호를 판정합니다:

| 규칙 | 잡아내는 것 |
|---|---|
| **Orphan Release** | PR 없이 직접 푸시된 커밋발 릴리스 |
| **Unreviewed** | 리뷰 없음 / self-approval / 봇만 승인 |
| **Workflow Drift** | publish 워크플로가 릴리스 직전 변경됨 |
| **OIDC Mismatch** | 서명 신원·저장소가 예상과 불일치 |
| **Unexpected Builder** | 평소와 다른 CI·트리거·브랜치 |
| **Tag/Identity Drift** | maintainer·태그 급변 |

## 동작 방식

```
입력(package@version)
  → 수집 (npm · GitHub · Sigstore/Rekor)
  → 계보 복원 (그래프)
  → 규칙 판정 + 신뢰 점수
  → PASS / RISK / UNVERIFIABLE
  → (게이트) 위험하면 설치 차단
```


## 아키텍처

- **Collectors** — npm·GitHub·Sigstore/Rekor에서 증거 수집
- **Lineage Reconstructor** — 증거를 계보 그래프로 조립
- **Rule Engine** — 6개 규칙 + 신뢰 점수
- **Gate** — 판정 결과로 설치 차단/허용

## 지원 범위 & 한계

- **지원:** npm (PyPI 예정)
- **대상:** 공개 GitHub 소스 + provenance(attestation) 보유 패키지
- **한계:** provenance가 없는 패키지는 계보 복원 불가 → `UNVERIFIABLE`로 표시 (안전/위험으로 단정하지 않음)
- 비공개 저장소는 검증 불가


## 팀

| 역할 | 담당 |
|---|---|
| npm 수집 | 미정 |
| GitHub 수집 | 미정 |
| Sigstore/Rekor 수집 | 미정 |
| 통합·평가 | 미정 |


WBS
<img width="1206" height="998" alt="image" src="https://github.com/user-attachments/assets/91759100-39bf-4f13-92a3-ac5325e89178" />
