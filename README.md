# AI Reference Vault

외부 AI 코딩 도구들의 시스템 프롬프트, 아키텍처, 설계 패턴을 수집·분석하는 레퍼런스 저장소.

우리 다중 에이전트 체계(PM/파트장/수행)의 지속적 개선을 위한 참조 자료로 활용한다.

---

## 구조

```
ai-reference-vault/
├── claude-code/              # Anthropic Claude Code 시스템 프롬프트
│   ├── system-prompts/       # 110+ 프롬프트 파일 (Piebald-AI 추출)
│   ├── tools/                # 빌트인 도구 정의
│   ├── CHANGELOG.md          # 버전별 변경 이력
│   └── README.md             # 원본 설명
│
├── cursor/                   # Cursor IDE 시스템 프롬프트
│   ├── Agent Prompt 2.0.txt  # 최신 Agent 프롬프트 (38KB)
│   ├── Agent Prompt v1.2.txt # v1.2 프롬프트
│   ├── Agent Prompt v1.0.txt # v1.0 프롬프트
│   ├── Agent Prompt 2025-09-03.txt
│   ├── Agent CLI Prompt 2025-08-07.txt
│   ├── Agent Tools v1.0.json # 도구 정의 (JSON)
│   └── Chat Prompt.txt       # 채팅 모드 프롬프트
│
├── analysis/                 # 분석 및 적용 문서
│   └── actionable-patterns.md # 우리 체계에 적용 가능한 패턴
│
└── README.md                 # 이 파일
```

---

## 출처

| 폴더 | 원본 저장소 | Stars | 설명 |
|------|------------|-------|------|
| `claude-code/` | [Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) | 7.8K | Claude Code npm 디컴파일 → 시스템 프롬프트 추출, 버전별 추적 |
| `cursor/` | [x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) | 134K | 28개+ AI 도구 시스템 프롬프트 모음 (Cursor 폴더만 추출) |

---

## 배경: Claude Code 소스코드 유출 사건

2026년 3월 31일, Anthropic의 `@anthropic-ai/claude-code` npm 패키지 v2.1.88에 59.8MB 크기의 source map 파일(`cli.js.map`)이 포함된 채 배포되었다. Bun 번들러의 기본 설정으로 source map이 자동 생성되는데, `.npmignore`에서 `*.map` 파일 제외 처리를 누락한 것이 원인이다.

이를 통해 1,906개 TypeScript 파일(51만 2천 줄+)의 원본 소스코드가 공개됐으며, GitHub에서 2시간 만에 57,000 star를 기록했다. 2025년 2월에도 동일한 방식의 유출 전례가 있었으나 재발 방지가 이루어지지 않았다.

### 유출로 드러난 미공개 기능

- **KAIROS**: 백그라운드 자율 실행 에이전트 + autoDream 메모리 통합
- **Undercover Mode**: 공개 저장소에서 AI 흔적을 git에서 자동 삭제
- **BUDDY**: 다마고치 스타일 ASCII 펫 (18종, 희귀도)
- **Ultraplan**: 원격 Opus 세션으로 30분짜리 계획 수립
- **Coordinator Mode**: 멀티 에이전트 오케스트레이션

---

## 업데이트 방법

### Claude Code 프롬프트 갱신
```bash
# Piebald-AI는 Claude Code 새 버전마다 수 분 내 업데이트
cd claude-code
# 원본 저장소에서 최신 파일 확인 후 수동 갱신
```

### Cursor 프롬프트 갱신
```bash
# x1xhlol 저장소에서 Cursor Prompts 폴더 확인
# 새 버전이 올라오면 해당 파일만 다운로드
```

---

## 관련 프로젝트

- **universal-devkit**: 우리 에이전트 규칙/표준 (`projects-hub/universal-devkit`)
- **multi-agent-workflow.mdc**: PM/파트장/수행 워크플로우 규칙
- **who-we-are.md**: 관계 맥락 + 작업 원칙

---

> *이 저장소의 자료는 학습/참고 목적으로 수집되었으며, 상업적 이용을 목적으로 하지 않는다.*
