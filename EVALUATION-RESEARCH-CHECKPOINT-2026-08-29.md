# Evaluation 연구 체크포인트 — 2026-08-29

> **이 문서의 성격.** 연구 checkpoint / evidence note 다. **설계 결정도 구현 계획도 아니다.**
> 기존 Evaluation 설계를 대체하지 않는다. 오늘 확인한 것은 **Evaluation이 무엇을 주장할 수 있는가의 경계**이며, 실행 골격은 뒤집히지 않았다.
>
> 기존 골격(유효):
> `실제 User Query → Agent Response + 실제 Returned Evidence/Trace → Role×Task Criteria → Pointwise Judge → Verdict + Reason + Locator → Policy Aggregation`

---

## 1. 오늘 확인된 결론

**① Grounding ≠ Truth**
런타임 입력이 `User Query + Agent Response + returned Evidence` 뿐이면, Judge 가 판정할 수 있는 것은 *"이 답이 이 evidence 로 정당화되는가"* 다. Evidence 자체가 현실이라는 보장이 없으므로 업무적 진실성은 자동으로 따라오지 않는다.

**② 세 면 분리 — 하나의 FAIL enum 으로 평탄화하지 않는다**

| 면 | 질문 |
|---|---|
| ① Response Quality | 받은 evidence 로 답을 제대로 했는가 |
| ② Evidence Fitness | 그 답을 하기 위한 evidence 에 명백한 결손·문제가 있는가 |
| ③ Judge Reliability | 그것을 판정하는 Judge 를 믿을 만한가 |

**③ Task Adequacy 는 두 개다**
- **evidence-relative completeness** — 받은 evidence 안에서 빠뜨렸는가 → 운영 trace 만으로 평가 가능
- **world-relative completeness** — 현실에 있는 것을 다 찾았는가 → 독립 진리 신호 없이는 불가

**④ 결정적/판정 절단선은 "면 단위"가 아니라 "면 안에서" 그어지며, 태스크마다 위치가 다르다**
③ Judge Health 는 면을 무효화하지 않고 **그 면 안에서 LLM 이 산출한 판정의 신뢰성만 제한**한다. 요구 evidence 종류의 존재 여부 같은 결정적 검사는 Judge 상태와 무관하게 유효하다.
바닥 두께는 태스크마다 크게 다르다 — 영향도 분석은 두껍고, **원인 분석은 거의 없다**(실패 신호 하나뿐, 나머지는 탐색 중 발견).

**⑤ ③의 검증 비용은 ①의 criteria 설계에 직접 걸려 있다**
xSAVIKx 하네스가 약 9 케이스로 되는 이유는 차원이 4개 고정이고 candidate 가 한 문장이기 때문이다. 우리 쪽은 `Task × 변이 × evidence 층` 으로 수십 케이스 + 각각 얼린 trace 가 필요하다. **Role×Task 를 넓게 열수록 Judge 를 검증할 수 없게 되고, 검증 못 하는 Judge 는 ③이 없는 것과 같다.**

---

## 2. Evaluation 이 보증하지 않는 것 (오늘의 핵심 산출물)

1. **Corpus 가 현실과 완전히 일치한다는 것**
2. **Retrieval 이 가능한 evidence 를 전부 찾았다는 것**
3. **세계 전체 기준으로 답변이 완전하다는 것**

> PASS 의 의미: *"이 답변은 **당시 제공된 evidence 와 해당 Task 기준에서** 통과했다."*
> **"현실에서 참이다" 가 아니다.**

이 세 줄이 있으면 *"Evaluation PASS 인데 왜 실제로 틀렸나"* 라는 질문에 대해 **시스템 실패인지 보증 범위 밖 문제인지** 즉시 갈린다.

---

## 3. 살아남은 가설 (구현 대상 아님)

**H1. Required Evidence Floor**
Role×Task 는 최소 evidence 바닥을 **클래스 단위**로 선언할 수 있을지 모른다.
- **비대칭**: floor 미달 → INSUFFICIENT 를 잡을 수 있음 / floor 충족 → SUFFICIENT 를 증명한 것은 **아님**
- 성립 강도는 `Task 종류 × Claim 종류(존재/부재) × Task identity provenance` 에 따라 다름
- **부재 주장**("X 가 없다")은 positive evidence floor 로 평가 불가 — coverage evidence 라는 다른 유형이 필요
- **한계**: floor 는 evidence type 의 존재를 검사하지, **retrieval completeness 를 보증하지 않는다**

**H2. Task identity provenance**
Task 가 어디서 오는지에 따라 floor 검사의 결정성이 달라진다 — `explicitly declared` / `runtime-observed` / `post-hoc inferred`. Evidence 뿐 아니라 **task classification 에도 provenance 가 필요할 수 있다.**

**H3. External Reality Check** *(이름 주의: "Truth Channel" 아님)*
①②③ 만으로는 world truth 를 보증할 수 없다는 **한계 선언**에서 나온 것이지, 네 번째 자동화 시스템 요구가 아니다.
- **표본**이면 충분하다. 전수 아님
- **판단 근거가 같은 corpus 에서 오면 안 된다** — 그러면 독립이 아니고 common-mode failure 를 통과시킨다
- downstream outcome 은 강력한 신호 하나지 **범용 채널이 아니다**(설명형 질의는 성패 자체가 없음)

**H4. 비보증 범위의 verdict surface 노출** *(전이 가설 — 다른 문제에서 온 패턴)*
비보증 범위가 결과 표면에서 사라지면 시간이 지나며 PASS 가 확대 해석된다. **지금 verdict taxonomy 를 만들자는 뜻이 아니다.**

**H5. Judge Validation 보강 (xSAVIKx 참고점)**
반복 테스트는 **일관성**을 보지 **정확도**를 못 본다 — 일관되게 틀릴 수 있다. 소량 labelled fixture 로 보완 가능:
- 명명된 실패 유형별 **변이 가족** (strong / restated / hallucinated / over-long 류)
- **차원별 비균일 기대 밴드** — 나쁜 후보가 위반하지 않은 차원에서는 잘 받아도 된다 → **criterion 독립성** 시험 (ordering 보다 강함)
- **evidence 빈곤도 층화**
- **변동 허용치 선언** (예: ±0.5, 인접 밴드 허용)
- Judge 는 **triage** 를 하지 verdict 를 하지 않는다 (하위 점수 = 사람 검토 큐)

---

## 4. UNKNOWN — 딱 하나

**기존 Aggregation semantics.**
자연어 업무정책 기반 Aggregation 을 선택했다는 것까지는 확실하다. 그러나 **매 실행마다 LLM 이 정책을 해석했는지**, **이미 고정된 실행 형태가 있었는지**는 현재 자료로 알 수 없다.

관측 없이 A/B 실험을 설계하지 않는다. **Eval Spec 원문 또는 실제 aggregator 코드/프롬프트 확보 시 해제.**

---

## 5. 월요일 확인표 — **반증형**

> 오늘 만든 어휘를 들고 기존 Spec 을 읽으면 거기서 우리 결론을 "발견"하게 된다. 그래서 **무엇이 나오면 아니라고 판정할지**를 미리 적는다.

| 확인 항목 | 이게 나오면 "안 들어있다" |
|---|---|
| 실제 Query/Response/Returned Evidence 기반인가 | 평가 입력이 **사람이 만든 고정 질문셋**이거나, evidence 없이 응답만 들어간다 |
| ①②③ 이 안 섞였나 | verdict enum **하나**에 응답 실패와 evidence 실패가 함께 들어있다 |
| PASS 가 과장돼 있나 | 산출물·문서 어디에도 **비보증 범위가 없다.** PASS 가 수식어 없이 단독으로 쓰인다 |
| Aggregation 실제 형태 | 정책 텍스트가 **런타임 프롬프트 안에** 있다 → 런타임 해석 / 별도 규칙·설정으로 고정 → 이미 결정적 |

마지막 줄이 §4 의 UNKNOWN 을 **파일 하나 열어서** 가르는 방법이다.

---

## 6. 상태 — HOLD 아님

**CONTINUE + VERIFY-ON-RESUME.**

월요일 순서:
1. 개발하다 만 **기존 Evaluation 구현/Spec 을 연다**
2. §5 반증표로 대조한다
3. **멀쩡하면 안 뜯는다. 그대로 개발 재개**
4. 오늘 결론과 **실제로 충돌하는 부분만** 수정 후보

§3 의 가설들(H1~H5)은 **월요일 개발의 선행조건이 아니다.** 옆에 두고 기존 Evaluation 부터 완성해도 된다. §4 의 UNKNOWN 도 전체를 막지 않는다 — 그 부분 구현 차례가 왔을 때 확인하면 된다.

---

## 7. 오늘 폐기된 가설 — 재발견 방지

> 그럴듯해 보여서 다시 제안되기 쉬운 것들이다. **폐기 이유까지 같이 남긴다.**

| 폐기 | 이유 |
|---|---|
| **③→②→① 일렬 gate** | 과함. 결정적 evidence-missing 판정은 Judge 상태와 무관하게 유효하고, evidence 가 없어도 "근거 없음을 명시했는가"는 평가된다. → **면 안에서 결정적/판정 절단**으로 대체 |
| **Evaluation 의 okf-oracle(producer) 종속** | 근거 없음. *"영향도 분석에 최소 어떤 evidence 가 필요한가"* 는 **Eval Spec 문제**이며 producer 선택과 논리적으로 독립이다. 오전 OKF 작업 문맥이 저녁에 샌 것 |
| **상시 외부 Truth Channel 요구** | 논리 비약. F-1~F-5 가 존재한다는 관측에서 나오는 결론은 *"①②③ 만으로는 world truth 를 보증할 수 없다"* 는 **한계 선언**이지, 네 번째 자동화 시스템 요구가 아니다 → **H3(표본)** 으로 축소 |
| **FAIL 5종 단일 enum** | `RESPONSE_FAIL / EVIDENCE_INSUFFICIENT / EVIDENCE_CONFLICT / SOURCE_QUALITY_ISSUE / JUDGE_UNCERTAIN` 을 한 enum 에 평탄화하면 방금 만든 ①②③ 분리를 되접는다. 세 면에 각자 상태를 둔다 |
| **전수 Gold / 모든 답변 SME 검증 / Truth Engine / Retrieval Evaluation 편입** | 제품 방향(human 최소화)과 충돌하고 시스템을 다시 무겁게 만든다 |
| **aggregation A/B 실험을 지금 1순위로** | 기존 aggregation 이 런타임 LLM 해석이었다는 **증거가 없다.** 관측 전에 실험을 설계한 것 → §4 로 이동 |

---

## 8. 방법론 관찰

오늘 두 번 같은 방식으로 미끄러졌다 — **다른 문제에서 검증된 구조를 새 문제에 그대로 얹었다.**

> **Pattern Transfer ≠ Evidence.**
> 다른 문제에서 검증된 구조는 새 문제의 **가설**을 만드는 데 쓸 수 있지만, 새 문제의 **근거**가 되지는 않는다.
> 전이된 패턴은 **가설 등급으로 재진입**한다. 떠날 때의 확신 등급을 유지하지 않는다.

**탐지법**: 주장의 근거가 *"이 문제에서의 관측"* 이 아니라 *"저기서 통했다"* 면 그것은 전이다.

그리고 별개의 실패가 하나 더 있었다 — **관측 하나에 설명 하나를 즉시 붙이기**(대안 원인 미배제). 위와 기제가 다르므로 섞어 부르지 않는다.
