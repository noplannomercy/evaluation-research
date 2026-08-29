# Evaluation 연구 — 2026-08-29 체크포인트

**Private.** 내부 프로젝트 문맥 포함. 공개 전환 금지.

| 파일 | 내용 |
|---|---|
| [`EVALUATION-RESEARCH-CHECKPOINT-2026-08-29.md`](EVALUATION-RESEARCH-CHECKPOINT-2026-08-29.md) | 연구 checkpoint — 결론 / 살아남은 가설 / 비보증 3가지 / UNKNOWN / 반증형 확인표 / 폐기 목록 |
| [`CLAUDE.md`](CLAUDE.md) | 세션 부트스트랩 — 하지 말 것, 첫 행동, 유지할 규율 |

## 상태

**CONTINUE + VERIFY-ON-RESUME.** HOLD 아니다.
개발하다 만 Evaluation 구현을 이어서 완성하는 것이 현재 작업이고, 체크포인트는 재설계 지시가 아니라 **경계선**이다.

## 핵심 한 줄

> Evaluation 기본 설계는 유지한다. 다만 **"실제 진실을 판정하는 시스템"이 아니라 "실제 운영 trace 에서 Response·Evidence·Judge 의 품질과 실패 귀속을 평가하는 시스템"** 으로 경계를 명확히 한다. World Truth 는 보증 범위 밖이며 필요할 때 표본 Reality Check 로 보완한다.

## Evaluation 이 보증하지 않는 것

1. Corpus 가 현실과 완전히 일치한다는 것
2. Retrieval 이 가능한 evidence 를 전부 찾았다는 것
3. 세계 전체 기준으로 답변이 완전하다는 것

## UNKNOWN 하나

기존 **Aggregation semantics** — 런타임 LLM 해석이었는지, 이미 고정된 실행이었는지. Eval Spec 원문 또는 aggregator 코드/프롬프트 확보 시 해제.

## 별도 트랙

OKF producer / okf-go fork 작업은 다른 저장소다 (`okf-skills-fork`, `okf-series-validation`). 섞지 않는다.
