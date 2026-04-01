# 적용 가능 패턴 분석

> 유출/공개된 AI 코딩 도구 프롬프트에서 추출한, 우리 다중 에이전트 체계에 직접 적용할 수 있는 패턴들.

---

## 1. Verification Specialist — 자기 불신 패턴

**출처**: `claude-code/system-prompts/agent-prompt-verification-specialist.md`

**핵심 원문**:
> "You are Claude, and you are bad at verification. This is documented and persistent."

**패턴 요약**:
- 코드를 읽고 "맞는 것 같다" → 검증이 아님. 실행해야 검증.
- "아마 괜찮을 거다" → "아마"는 검증이 아님.
- 부모 에이전트도 LLM이므로 테스트가 순환적(circular)일 수 있음.
- PARTIAL은 도피처가 아님 — 실행했으면 PASS/FAIL 결정 필수.

**적용 대상**: `multi-agent-workflow.mdc` 파트장 리뷰 체크리스트

**구체적 변경안**:
```markdown
## 파트장 리뷰 체크리스트 (추가)

- [ ] 코드를 "읽고 판단"한 게 아니라 실제로 grep/실행으로 확인했는가?
- [ ] "아마 맞을 것이다"라고 생각한 부분이 있다면, 그 부분을 검증했는가?
- [ ] 수행 에이전트의 테스트가 순환적이지 않은가? (assert가 코드 동작을 그대로 반복하고 있지 않은가?)
```

**적용 상태**: ✅ 적용 완료 (2026-03-29) — `multi-agent-workflow.mdc` "파트장 검증 체크리스트" 섹션

---

## 2. Dream Memory Consolidation — 세션 간 기억 통합

**출처**: `claude-code/system-prompts/agent-prompt-dream-memory-consolidation.md`

**패턴 요약** (4단계):
1. **Orient** — 기존 메모리 구조 파악, 인덱스 읽기
2. **Gather** — 최근 로그, 모순된 사실, 트랜스크립트에서 좁은 범위 검색
3. **Consolidate** — 기존 토픽에 병합 (중복 생성 방지), 상대 날짜 → 절대 날짜 변환
4. **Prune** — 인덱스를 25KB 이내로 유지, 오래된/모순된 포인터 제거

**적용 대상**: `SESSION_HANDOFF.md` 갱신 규칙

**구체적 변경안**:
```markdown
## SESSION_HANDOFF 갱신 규칙 (추가)

- 상대 날짜("어제", "지난주") 사용 금지 → 절대 날짜(2026-03-04) 사용
- 새 정보가 기존 항목과 모순되면, 기존 항목을 수정/삭제 (누적하지 않음)
- "최근 작업" 섹션은 최근 5개 세션까지만 유지 (오래된 건 요약 후 삭제)
- 인덱스 역할을 하는 "다음에 할 것" 섹션은 완료 항목 즉시 제거
```

**적용 상태**: ✅ 적용 완료 (2026-03-29) — `.cursorrules` "SESSION_HANDOFF 갱신 규칙" 섹션

---

## 3. Security Monitor — 범위 확대 감지

**출처**: `claude-code/system-prompts/agent-prompt-security-monitor-for-autonomous-agent-actions-first-part.md`

**패턴 요약**:
- 기본값은 ALLOW — 차단은 명시적 조건 매칭 시에만
- **범위 확대 = 자율 행동**: 유저가 "디버그해줘" → 에이전트가 인프라 삭제 → BLOCK
- **질문은 동의가 아님**: "이거 고칠 수 있어?" ≠ "고쳐"
- **침묵은 동의가 아님**: 유저가 개입하지 않은 것 ≠ 승인
- 서브에이전트 위임 시 프롬프트 내용도 BLOCK 규칙 적용

**적용 대상**: 수행 에이전트 가이드라인

**구체적 변경안**:
```markdown
## 수행 에이전트 행동 규칙 (추가)

- Work Order에 명시되지 않은 파일은 수정하지 않는다
- "이것도 같이 고치면 좋겠다"는 자체 판단으로 범위를 확장하지 않는다
- 삭제/배포/환경변수 변경은 Work Order에 명시된 경우에만 수행한다
- 확신이 없으면 PM에게 보고한다 (추측으로 진행하지 않는다)
```

**적용 상태**: ✅ 적용 완료 (2026-03-29) — `multi-agent-workflow.mdc` "수행 에이전트 행동 규칙" 섹션

---

## 4. Auto Mode — 자율 실행의 경계

**출처**: `claude-code/system-prompts/system-prompt-auto-mode.md`

**패턴 요약**:
- 즉시 실행, 질문 최소화, 합리적 가정으로 진행
- 그러나 **파괴적 행동은 여전히 확인 필요** (삭제, 프로덕션 수정)
- 데이터 유출 금지 — 시크릿은 명시적 승인 없이 공유 불가

**적용 대상**: PM 커밋/배포 규칙

**구체적 변경안**:
```markdown
## PM 자율 실행 예외 (추가)

- 파일 삭제 (5개 이상): 목록 확인 후 진행
- 환경변수 변경: 변경 내용 명시
- 프로덕션 배포: Railway/Vercel 환경 확인
- API 키/시크릿 언급: 절대 금지
```

**적용 상태**: ✅ 적용 완료 (2026-03-29) — `.cursorrules` "PM 자율 실행 예외" 섹션

---

## Cursor vs Claude Code 편집 패턴 비교 (참고)

**출처**: `cursor/Agent Prompt 2.0.txt` vs `claude-code/system-prompts/tool-description-edit.md`

| 항목 | Cursor | Claude Code |
|------|--------|-------------|
| 편집 방식 | `// ... existing code ...` + 작은 모델이 적용 | 정확한 문자열 교체 (exact match) |
| 장점 | 변경 의도 전달에 유리, 대규모 편집 가능 | 정확도 높음, 부작용 없음 |
| 단점 | 작은 모델이 잘못 적용할 수 있음 | 긴 코드블록 교체 시 old_string 찾기 어려움 |

**인사이트**: Cursor의 위임 패턴은 "빠른 대량 편집"에 유리하고, Claude Code의 정확 교체는 "안전한 소량 편집"에 유리. 우리 체계에서는 수행 에이전트의 코드 편집 시 두 패턴의 장단점을 상황에 맞게 활용할 수 있다.

---

## 서브에이전트 아키텍처 비교 (참고)

| 구조 | Claude Code | 우리 체계 |
|------|-------------|----------|
| 탐색 | Explore 에이전트 (readonly) | 파트장 사전 리뷰 (readonly) |
| 계획 | Plan 에이전트 + Ultraplan | PM Work Order |
| 구현 | Task 에이전트 (generalPurpose) | 수행 에이전트 |
| 검증 | Verification Specialist | 파트장 코드 리뷰 |
| 배포 | Quick PR/Commit 에이전트 | PM 커밋 |
| 감시 | Security Monitor | 수행 에이전트 행동 규칙에 내재화 |
| 메모리 | Dream Consolidation + Session Memory | SESSION_HANDOFF.md |
| 조율 | Coordinator Mode (미출시) | PM이 수동 조율 |

**인사이트**: Claude Code는 **검증**과 **보안 감시**를 별도 에이전트로 분리한 게 특징. 우리는 파트장이 검증까지 겸하고 있어서, 대규모 작업에서는 파트장 부하가 클 수 있다. Verification Specialist 패턴을 파트장 리뷰에 내재화하는 것이 현실적.

---

## 5. Simplify — 구현 직후 자동 정화

**출처**: `claude-code/system-prompts/skill-simplify.md`

**패턴 요약**:
- 구현 완료 후 3개 관점에서 병렬 리뷰: 코드 재사용 / 코드 품질 / 효율성
- 재사용: 새 코드가 기존 유틸/헬퍼와 중복되지 않는가?
- 품질: 불필요한 상태, 복붙, 파라미터 난립, 불필요한 주석
- 효율: N+1 패턴, 놓친 병렬화, 핫패스 부하, TOCTOU 안티패턴

**적용 상태**: ✅ 적용 완료 (2026-04-01) — `multi-agent-workflow.mdc` 흐름 4단계 "수행: 자기 정화" (재사용/품질/효율 3축 적용. 원문의 N+1, TOCTOU 등은 런타임 검증 단계에서 커버)

---

## 6. Verification ≠ Testing — 검증은 테스트가 아니다

**출처**: `claude-code/system-prompts/skill-verify-skill.md`

**패턴 요약**:
- 테스트(`pytest`, `npm test`)는 CI가 한다. 다시 돌리는 건 CI를 재실행한 것일 뿐.
- 검증은 앱을 실제로 실행해서 Surface(CLI, API, UI)에서 동작을 확인하는 것.
- "import해서 함수 호출 + console.log = 단위테스트를 새로 쓴 것, 앱을 실행한 게 아님"
- Adversarial probe 필수: 경계값, 동시성, 멱등성 중 최소 1개

**적용 상태**: ✅ 적용 완료 (2026-04-01) — `multi-agent-workflow.mdc` 파트장 검증 체크리스트 "런타임 검증" 섹션

---

## 7. Worker 보고 형식 표준화 + "이해 위임 금지"

**출처**: `claude-code/system-prompts/agent-prompt-worker-fork-execution.md`, `system-prompt-writing-subagent-prompts.md`

**패턴 요약**:
- PM → 서브에이전트: 목적+이유+파악한것+구체적범위. "조사해서 고쳐" 금지.
- 서브에이전트 → PM: Scope/Result/Key files/Files changed/Issues 형식, 500단어 이내
- "Never delegate understanding" — 프롬프트가 PM의 이해를 증명해야 한다

**적용 상태**: ✅ 적용 완료 (2026-04-01) — `multi-agent-workflow.mdc` "서브에이전트 보고 형식" 섹션 + "하지 말 것" 항목

---

## 8. Anthropic 코딩 철학 7계명

**출처**: `claude-code/system-prompts/system-prompt-doing-tasks-*.md` (7개 파일), `system-prompt-output-efficiency.md`

**패턴 요약**:
1. 불필요한 추가 금지 — 버그 수정이면 버그만
2. 조기 추상화 금지 — 비슷한 3줄이 추상화보다 낫다
3. 호환성 해킹 금지 — 안 쓰면 삭제
4. 불필요한 에러 처리 금지 — 경계에서만 검증
5. 시간 예측 금지
6. 야심찬 작업 허용
7. 출력 최소화 — 답부터, 세 문장이면 한 문장으로

**적용 상태**: ✅ 적용 완료 (2026-04-01) — `.cursorrules` "코딩 철학 (Anthropic 패턴)" 섹션

---

## 9. Skillify — 경험을 실행 가능한 자산으로

**출처**: `claude-code/system-prompts/system-prompt-skillify-current-session.md`

**패턴 요약**:
- 반복 가능한 작업 패턴을 발견하면, "배운 것 기록"이 아닌 "실행 가능한 체크리스트"로 전환
- learning-log: 읽기용 → SKILL.md: 실행용

**적용 상태**: ✅ 적용 완료 (2026-04-01) — `.cursorrules` "learning-log → 실행 가능한 체크리스트 전환" 섹션

---

## 10. Conversation Summary 구조 — 유저 피드백/에러 보존

**출처**: `claude-code/system-prompts/agent-prompt-conversation-summarization.md`

**패턴 요약**:
- SESSION_HANDOFF에 유저의 방향 수정 피드백 반드시 기록
- 에러와 해결 방법을 구체적으로 기록 (파일명, 에러 메시지, 수정 내용)
- "All user messages" — 유저의 피드백을 잃어버리지 않겠다는 원칙

**적용 상태**: ✅ 적용 완료 (2026-04-01) — `.cursorrules` SESSION_HANDOFF 갱신 규칙 추가 항목

---

## 적용 현황

| 순위 | 패턴 | 상태 | 적용일 | 대상 파일 |
|------|------|------|--------|----------|
| 1 | Verification Specialist | ✅ | 2026-03-29 | `multi-agent-workflow.mdc` |
| 2 | Dream Memory | ✅ | 2026-03-29 | `.cursorrules` |
| 3 | 범위 확대 감지 (Security Monitor) | ✅ | 2026-03-29 | `multi-agent-workflow.mdc` |
| 4 | 자율 실행 경계 (Auto Mode) | ✅ | 2026-03-29 | `.cursorrules` |
| — | 편집 패턴 비교 | — | — | 참고용 |
| — | 서브에이전트 아키텍처 비교 | — | — | 참고용 |
| 5 | Simplify 자동 정화 | ✅ | 2026-04-01 | `multi-agent-workflow.mdc` |
| 6 | Verification ≠ Testing | ✅ | 2026-04-01 | `multi-agent-workflow.mdc` |
| 7 | Worker 보고 형식 + 이해 위임 금지 | ✅ | 2026-04-01 | `multi-agent-workflow.mdc` |
| 8 | 코딩 철학 7계명 | ✅ | 2026-04-01 | `.cursorrules` |
| 9 | Skillify | ✅ | 2026-04-01 | `.cursorrules` |
| 10 | 유저 피드백/에러 보존 | ✅ | 2026-04-01 | `.cursorrules` |

---

> *이 문서는 새로운 패턴을 발견할 때마다 업데이트한다. 적용 완료 시 "적용 상태"를 변경하고, 적용 커밋 해시를 기록한다.*
