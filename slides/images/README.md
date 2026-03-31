# 이미지 가이드

이 폴더에 아래 이미지들을 추가해주세요.

## 필요한 이미지 목록

| 파일명 | 슬라이드 | 설명 | 권장 크기 |
|--------|---------|------|-----------|
| `cover.png` | 표지 | AI 로봇이 도구를 사용하는 일러스트 또는 뇌+톱니바퀴 컨셉 이미지 | 400x300px |
| `agentic-ai-components.png` | Agentic AI 핵심 능력 | 중앙 Agent + 4요소(Planning, Reflection, Memory, Tools) 원형 배치 | 600x400px |
| `langchain-overview.png` | LangChain 소개 | LangChain 로고 + Models→Prompts→Chains→Agents→Memory 흐름 | 500x300px |
| `single-agent-architecture.png` | Single Agent 아키텍처 | 사용자→Agent→[Planning\|Reflection\|Memory\|Tools\|RAG] 플로우차트 | 600x450px |
| `planning-example.png` | Planning 예시 | 날씨 확인→조건 분기→추천 의사결정 트리 | 550x350px |
| `self-reflection-loop.png` | Self-Reflection 패턴 | 생성→평가→분기(만족/불만족)→재생성 순환 루프 | 500x400px |
| `memory-architecture.png` | Memory 아키텍처 | Working Memory ↔ Long-term Memory 계층 구조 | 550x400px |
| `tool-use-concept.png` | Tool Use | Agent 중심 허브-스포크 다이어그램 (검색, 계산, API, 파일, 코드) | 500x350px |
| `cot-comparison.png` | Chain of Thought | 좌우 비교: Direct Answer vs CoT 단계별 체인 | 600x300px |
| `rag-architecture.png` | RAG 아키텍처 | Query→Embedding→Vector DB→Top-K→Prompt→LLM→Response 파이프라인 | 700x400px |
| `langgraph-workflow.png` | LangGraph 워크플로우 | START→Planner→Executor→Reflector→END 상태 기계 다이어그램 | 600x350px |
| `summary-infographic.png` | 정리 | 6개 핵심 요소 아이콘 + 한 줄 설명 인포그래픽 | 700x400px |

## 이미지 삽입 방법

슬라이드의 `![placeholder](images/파일명.png)` 부분을 그대로 두거나, HTML 주석의 삽입 방법 안내를 참고하세요.

### Marp 이미지 문법 예시

```markdown
# 기본 삽입
![](images/cover.png)

# 가운데 정렬
![center](images/diagram.png)

# 오른쪽 배치 (40% 너비)
![right:40%](images/sidebar.png)

# 크기 지정
![width:500px](images/large.png)
```

## 이미지 생성 도구 추천

- **Excalidraw**: 손그림 스타일 다이어그램 (https://excalidraw.com)
- **Figma**: 전문적인 디자인 (https://figma.com)
- **Mermaid**: 코드 기반 다이어그램 (Marp에서 직접 지원)
- **draw.io**: 무료 다이어그램 도구 (https://draw.io)
- **Canva**: 인포그래픽 제작 (https://canva.com)
