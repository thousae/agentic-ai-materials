---
marp: true
theme: csi-agent
paginate: true
size: 1920x1080
footer: "CSI Agent Lab @ SKKU"
---

<!-- _class: cover -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# Agentic AI

## 2차시: Multi-Agent (AutoGen)

Microsoft AutoGen 기반 멀티 에이전트 시스템 구축

---

# 목차

<div class="columns">
<div>

### Part 1: AutoGen 소개

1. AutoGen 프레임워크 개요
2. 설치 및 기본 설정
3. Agent 구성요소

</div>
<div>

### Part 2: Multi-Agent 패턴

4. RoundRobinGroupChat (순환 대화)
5. SelectorGroupChat (동적 선택)
6. Swarm (자율 핸드오프)

</div>
</div>

### Part 3: 실전 심화

7. Tool Use (도구 사용)
8. Human-in-the-Loop (사람 개입)
9. Termination 조건
10. 통합 실습

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 01

## AutoGen 프레임워크 개요

---

# AutoGen이란?

Microsoft가 개발한 _멀티 에이전트 AI 애플리케이션_ 구축 프레임워크

- 여러 Agent가 *대화를 통해 협업*하여 복잡한 작업 해결
- Agent 간 역할 분담, 도구 사용, 코드 실행 지원
- 다양한 LLM 벤더 지원 (OpenAI, Azure, Anthropic, Ollama 등)

---

# AutoGen 아키텍처

<div class="columns">
<div>

### 두 가지 레이어

|    레이어     | 설명                           |
| :-----------: | ------------------------------ |
| **AgentChat** | 고수준 API, 빠른 프로토타이핑  |
|   **Core**    | 저수준 Actor 모델, 분산 시스템 |

</div>
<div>

### AgentChat (권장 시작점)

- 사전 정의된 Agent 타입 제공
- 다양한 Team 패턴 내장
- 직관적인 API 설계

### Core

- 이벤트 기반 메시지 처리
- 비동기 Agent 런타임
- 프로덕션 분산 배포

</div>
</div>

---

# 1차시 vs 2차시 비교

|      구분       |   1차시 (LangChain)   |      2차시 (AutoGen)       |
| :-------------: | :-------------------: | :------------------------: |
| **에이전트 수** |     Single Agent      |        Multi-Agent         |
|  **협업 방식**  |  혼자 모든 작업 수행  |   역할 분담 + 대화 협업    |
| **프레임워크**  | LangChain / LangGraph |     AutoGen AgentChat      |
|  **핵심 개념**  |  Chain, Tool, Memory  | Team, Handoff, Termination |
| **적합한 작업** |   단일 작업 자동화    |  복잡한 다단계 협업 작업   |

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 02

## 설치 및 기본 설정

---

# AutoGen 설치

```python
# AutoGen AgentChat + OpenAI 확장 설치
pip install -U "autogen-agentchat" "autogen-ext[openai]"

# Azure OpenAI를 사용하는 경우
pip install -U "autogen-agentchat" "autogen-ext[openai,azure]"
```

### 지원 모델 클라이언트

|            클라이언트             | 패키지                      | 비고               |
| :-------------------------------: | --------------------------- | ------------------ |
|   `OpenAIChatCompletionClient`    | `autogen-ext[openai]`       | GPT-4o, GPT-4.1 등 |
| `AzureOpenAIChatCompletionClient` | `autogen-ext[openai,azure]` | Azure 배포 모델    |
|  `AnthropicChatCompletionClient`  | `autogen-ext[anthropic]`    | Claude (실험적)    |
|   `OllamaChatCompletionClient`    | `autogen-ext[ollama]`       | 로컬 모델 (실험적) |

---

# 모델 클라이언트 설정

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

# OpenAI 모델 클라이언트 생성
model_client = OpenAIChatCompletionClient(
    model="gpt-4o",
    # api_key="YOUR_API_KEY",  # 또는 OPENAI_API_KEY 환경변수 설정
)

# Azure OpenAI 사용 시
from autogen_ext.models.openai import AzureOpenAIChatCompletionClient

azure_client = AzureOpenAIChatCompletionClient(
    azure_deployment="gpt-4o",
    azure_endpoint="https://your-endpoint.openai.azure.com/",
    model="gpt-4o",
    api_version="2024-06-01",
)
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 03

## Agent 구성요소

---

# AssistantAgent

AutoGen의 _핵심 에이전트 타입_ - LLM 기반 자율 에이전트

```python
from autogen_agentchat.agents import AssistantAgent

agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message="당신은 유용한 AI 비서입니다.",
)
```

### 주요 파라미터

| 파라미터         | 역할                                        |
| ---------------- | ------------------------------------------- |
| `name`           | 에이전트 고유 식별자                        |
| `description`    | Team 오케스트레이터가 에이전트 선택 시 참고 |
| `model_client`   | LLM 백엔드                                  |
| `tools`          | 사용 가능한 도구 (Python 함수) 목록         |
| `system_message` | 행동 지침                                   |
| `handoffs`       | 핸드오프 가능한 에이전트 목록 (Swarm)       |

---

# Agent 실행 방법

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message="당신은 유용한 AI 비서입니다.",
)

async def main():
    # 방법 1: 결과 직접 받기
    result = await agent.run(task="Python의 장점 3가지를 알려줘")
    print(result.messages[-1].content)

    # 방법 2: 스트리밍 출력
    await Console(agent.run_stream(task="Python의 장점 3가지를 알려줘"))

asyncio.run(main())
```

---

# UserProxyAgent

*사람의 입력*을 에이전트 대화에 전달하는 프록시

```python
from autogen_agentchat.agents import UserProxyAgent

# 기본: 콘솔 입력
user_proxy = UserProxyAgent("user_proxy", input_func=input)

# 커스텀: 웹소켓 등 외부 입력
async def web_input(prompt, cancellation_token=None):
    data = await websocket.receive_json()
    return data["content"]

user_proxy = UserProxyAgent("user_proxy", input_func=web_input)
```

> UserProxyAgent는 자신의 차례가 오면 실행을 *중단*하고 사람의 입력을 기다립니다.

---

# 메시지 타입

Agent 간 통신에 사용되는 _구조화된 메시지_

|     메시지 타입     | 내용                    | 용도                |
| :-----------------: | ----------------------- | ------------------- |
|    `TextMessage`    | 문자열                  | 일반 텍스트 통신    |
| `MultiModalMessage` | 텍스트 + 이미지         | 멀티모달 입력       |
|  `HandoffMessage`   | 출발/도착 에이전트      | Swarm 핸드오프 신호 |
|    `StopMessage`    | 종료 사유               | 대화 종료 신호      |
|    `TaskResult`     | 메시지 목록 + 종료 사유 | 실행 최종 결과      |

```python
from autogen_agentchat.messages import TextMessage, HandoffMessage

msg = TextMessage(content="안녕하세요!", source="User")
handoff = HandoffMessage(source="travel_agent", target="refund_agent",
                         content="환불 처리를 요청합니다.")
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 04

## RoundRobinGroupChat (순환 대화)

---

# RoundRobinGroupChat이란?

Agent들이 _고정된 순서로 번갈아_ 발언하는 가장 단순한 Team 패턴

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Agent A  │ ──→ │ Agent B  │ ──→ │ Agent C  │ ──→ (다시 A로)
└──────────┘     └──────────┘     └──────────┘
```

### 특징

- 모든 Agent가 *공유 메시지 히스토리*를 볼 수 있음
- 순서가 고정되어 예측 가능한 흐름
- _Writer + Critic_ (Reflection) 패턴에 적합

---

# RoundRobin 예시: Writer + Critic

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.ui import Console

writer = AssistantAgent(
    "writer",
    model_client=model_client,
    system_message="당신은 창의적인 작가입니다. 주어진 주제로 글을 작성하세요.",
)
critic = AssistantAgent(
    "critic",
    model_client=model_client,
    system_message="""당신은 글 비평가입니다. 글을 검토하고 구체적인 피드백을 제공하세요.
    만족스러우면 'APPROVE'라고 응답하세요.""",
)
termination = TextMentionTermination("APPROVE")

team = RoundRobinGroupChat([writer, critic], termination_condition=termination)
await Console(team.run_stream(task="가을을 주제로 짧은 시를 써줘"))
```

---

# RoundRobin 실행 흐름

```
사용자: "가을을 주제로 짧은 시를 써줘"
    │
    ↓
┌─────────┐  "단풍잎이 물드는 계절..."
│  writer  │ ─────────────────────────→
└─────────┘
    │
    ↓
┌─────────┐  "운율을 개선하면 좋겠습니다..."
│  critic  │ ─────────────────────────→
└─────────┘
    │
    ↓
┌─────────┐  "수정된 시: 붉게 물든..."
│  writer  │ ─────────────────────────→
└─────────┘
    │
    ↓
┌─────────┐  "APPROVE"  ← 종료 조건 충족!
│  critic  │ ─────────────────────────→
└─────────┘
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 05

## SelectorGroupChat (동적 선택)

---

# SelectorGroupChat이란?

LLM이 대화 맥락을 분석하여 *다음 발언자를 동적으로 선택*하는 패턴

```
                  ┌──────────────────┐
                  │  Selector (LLM)  │
                  │  "누가 적합한가?" │
                  └────────┬─────────┘
              ┌────────────┼────────────┐
              ↓            ↓            ↓
         ┌────────┐  ┌─────────┐  ┌──────────┐
         │Planner │  │Searcher │  │ Analyst  │
         └────────┘  └─────────┘  └──────────┘
```

### 핵심 특징

- Agent의 `name`과 `description`을 기반으로 선택
- `selector_func`으로 커스텀 선택 로직 가능
- 복잡한 *멀티 전문가 협업*에 적합

---

<style scoped>
  pre { font-size: 0.62em; }
</style>

# SelectorGroupChat 구현

```python
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination

planner = AssistantAgent(
    "PlanningAgent",
    description="작업을 계획하는 에이전트. 새 작업 시 먼저 참여해야 함.",
    model_client=model_client,
    system_message="복잡한 작업을 하위 작업으로 분해하세요.",
)
searcher = AssistantAgent(
    "WebSearchAgent",
    description="웹에서 정보를 검색하는 에이전트.",
    tools=[search_web],
    model_client=model_client,
    system_message="검색하여 관련 정보를 반환하세요.",
)
analyst = AssistantAgent(
    "DataAnalystAgent",
    description="데이터 분석 및 계산을 수행하는 에이전트.",
    tools=[calculate],
    model_client=model_client,
    system_message="데이터를 분석하고 수치를 계산하세요.",
)
```

---

<style scoped>
  pre { font-size: 0.62em; }
</style>

# SelectorGroupChat 실행

```python
termination = TextMentionTermination("TERMINATE") | MaxMessageTermination(25)

team = SelectorGroupChat(
    [planner, searcher, analyst],
    model_client=model_client,                # Selector용 LLM
    termination_condition=termination,
    allow_repeated_speaker=True,              # 같은 에이전트 연속 발언 허용
)

await Console(team.run_stream(
    task="테슬라의 최근 주가 동향을 분석해줘"
))
```

### Selector의 선택 과정

1. 현재까지의 대화 히스토리 분석
2. 각 Agent의 `name` + `description` 확인
3. *가장 적합한 Agent*를 다음 발언자로 선택

---

<style scoped>
  pre { font-size: 0.62em; }
</style>

# 커스텀 Selector 함수

```python
from typing import Sequence
from autogen_agentchat.base import BaseAgentEvent, BaseChatMessage

def custom_selector(messages: Sequence[BaseAgentEvent | BaseChatMessage]) -> str | None:
    """항상 Planner를 거치도록 커스텀 라우팅"""
    if messages[-1].source != "PlanningAgent":
        return "PlanningAgent"      # Planner로 돌려보내기
    return None                      # None이면 LLM 기반 선택으로 폴백

team = SelectorGroupChat(
    [planner, searcher, analyst],
    model_client=model_client,
    termination_condition=termination,
    selector_func=custom_selector,   # 커스텀 선택 함수 등록
)
```

> `selector_func`이 Agent 이름을 반환하면 해당 Agent가 발언하고, `None`을 반환하면 LLM 기반 선택으로 폴백합니다.

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 06

## Swarm (자율 핸드오프)

---

# Swarm이란?

Agent가 스스로 *다음 담당자에게 핸드오프*하는 분산형 패턴

```
┌─────────────┐  HandoffMessage   ┌──────────────┐
│ travel_agent │ ───────────────→  │flights_refund │
│              │                   │              │
│  handoffs:   │  ←───────────────  │  handoffs:   │
│  [refund,    │    HandoffMessage │  [travel,    │
│   user]      │                   │   user]      │
└─────────────┘                   └──────────────┘
```

### 핵심 특징

- 중앙 오케스트레이터 _없음_ - 각 Agent가 자율적으로 판단
- `handoffs` 파라미터로 위임 가능한 대상 지정
- 고객 서비스, 워크플로우 라우팅에 적합

---

<style scoped>
  pre { font-size: 0.62em; }
</style>

# Swarm 구현: 고객 서비스

```python
from autogen_agentchat.teams import Swarm
from autogen_agentchat.conditions import HandoffTermination, TextMentionTermination

def refund_flight(flight_id: str) -> str:
    """항공편을 환불 처리합니다."""
    return f"항공편 {flight_id} 환불이 완료되었습니다."

travel_agent = AssistantAgent(
    "travel_agent",
    model_client=model_client,
    handoffs=["flights_refunder", "user"],
    system_message="""당신은 여행 상담 에이전트입니다. 환불 요청은 flights_refunder에게 위임하세요.
    사용자 정보가 필요하면 user에게 핸드오프하세요. 완료 시 TERMINATE라고 응답하세요.""",
)

flights_refunder = AssistantAgent(
    "flights_refunder",
    model_client=model_client,
    handoffs=["travel_agent", "user"],
    tools=[refund_flight],
    system_message="""당신은 환불 전문 에이전트입니다. refund_flight 도구로 환불을 처리하세요.
    완료 후 travel_agent에게 핸드오프하세요.""",
)
```

---

# Swarm 실행

```python
termination = (
    HandoffTermination(target="user")
    | TextMentionTermination("TERMINATE")
)

team = Swarm(
    [travel_agent, flights_refunder],
    termination_condition=termination,
)
```

---

# Swarm 실행 (계속)

```python
# 실행 및 사용자 핸드오프 처리 루프
async def main():
    result = await Console(
        team.run_stream(task="항공편 환불을 요청합니다.")
    )
    last = result.messages[-1]

    # 사용자에게 핸드오프되면 입력 받고 재개
    while isinstance(last, HandoffMessage) and last.target == "user":
        user_input = input("사용자: ")
        result = await Console(team.run_stream(
            task=HandoffMessage(
                source="user", target=last.source, content=user_input
            )
        ))
        last = result.messages[-1]
```

---

# Swarm 실행 흐름

```
사용자: "항공편 환불을 요청합니다"
    │
    ↓
┌──────────────┐  "항공편 ID가 필요합니다"
│ travel_agent │ ─→ HandoffMessage(target="user")   ← 사용자에게 핸드오프
└──────────────┘
    │
    ↓  사용자 입력: "FL-12345"
    │
┌──────────────┐  "환불 처리를 진행하겠습니다"
│ travel_agent │ ─→ HandoffMessage(target="flights_refunder")
└──────────────┘
    │
    ↓
┌────────────────┐  refund_flight("FL-12345") 호출
│flights_refunder│ ─→ HandoffMessage(target="travel_agent")
└────────────────┘
    │
    ↓
┌──────────────┐  "환불이 완료되었습니다. TERMINATE"
│ travel_agent │
└──────────────┘
```

---

# 3가지 Team 패턴 비교

|        구분        |  RoundRobin   | SelectorGroupChat |       Swarm       |
| :----------------: | :-----------: | :---------------: | :---------------: |
|   **발언 순서**    |   고정 순서   |  LLM이 동적 선택  | Agent가 자율 결정 |
| **오케스트레이터** |  없음 (순환)  |   LLM Selector    |    없음 (분산)    |
|  **적합한 작업**   | Writer-Critic |    전문가 협업    | 워크플로우 라우팅 |
|     **복잡도**     |     낮음      |       중간        |       높음        |
|     **유연성**     |     낮음      |       높음        |     매우 높음     |

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 07

## Tool Use (도구 사용)

---

# AutoGen에서의 Tool 정의

일반 Python 함수를 _도구로 직접 등록_ - 타입 힌트와 독스트링이 스키마로 자동 변환

```python
from autogen_agentchat.agents import AssistantAgent

# 도구: 일반 Python 함수 정의
async def get_weather(city: str) -> str:
    """주어진 도시의 현재 날씨를 조회합니다."""
    return f"{city}의 날씨: 맑음, 22°C"

async def calculate(expression: str) -> str:
    """수학 표현식을 계산합니다."""
    try:
        return f"결과: {eval(expression)}"
    except Exception:
        return "계산 오류"
```

---

# AutoGen에서서의 Tool 정의 (계속)

```python
# Agent에 도구 등록
agent = AssistantAgent(
    name="tool_agent",
    model_client=model_client,
    tools=[get_weather, calculate],
    system_message="도구를 사용하여 작업을 수행하세요.",
    reflect_on_tool_use=True,    # 도구 결과를 자연어로 정리
)
```

---

# Tool 실행 흐름

```
사용자: "서울 날씨 알려주고, 234 * 567 계산해줘"
    │
    ↓
┌──────────────────────────────────────┐
│           AssistantAgent (LLM)       │
│                                      │
│  1. 요청 분석                         │
│  2. get_weather("서울") 호출 결정      │
│  3. calculate("234 * 567") 호출 결정  │
└──────────────────┬───────────────────┘
                   │
         ┌─────────┴─────────┐
         ↓                   ↓
   get_weather("서울")  calculate("234*567")
   → "서울: 맑음, 22°C"  → "결과: 132678"
         │                   │
         └─────────┬─────────┘
                   ↓
┌──────────────────────────────────────┐
│  reflect_on_tool_use=True            │
│  → "서울은 맑고 22°C입니다.            │
│     234×567의 결과는 132,678입니다."   │
└──────────────────────────────────────┘
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 08

## Human-in-the-Loop (사람 개입)

---

# Human-in-the-Loop 패턴

<div class="columns">
<div>

### 전략 A: 대화 중 개입

`UserProxyAgent`를 Team에 포함

```python
team = RoundRobinGroupChat(
    [assistant, user_proxy],
    termination_condition=termination,
)
```

- Agent 차례마다 사람 입력 대기
- 실시간 피드백 제공 가능

</div>
<div>

### 전략 B: 실행 후 피드백

`max_turns=1`로 실행 후 루프

```python
team = RoundRobinGroupChat(
    [assistant], max_turns=1,
)
task = "시를 써줘"
while True:
    await Console(
        team.run_stream(task=task)
    )
    task = input("피드백: ")
    if task == "exit":
        break
```

</div>
</div>

---

# Human-in-the-Loop 실습

```python
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.ui import Console

assistant = AssistantAgent(
    "assistant",
    model_client=model_client,
    system_message="사용자와 대화하며 요청을 수행하세요.",
)
user_proxy = UserProxyAgent("user_proxy", input_func=input)

termination = TextMentionTermination("APPROVE")
team = RoundRobinGroupChat(
    [assistant, user_proxy],
    termination_condition=termination,
)

# 실행: assistant → user_proxy(입력 대기) → assistant → ...
await Console(team.run_stream(task="바다를 주제로 4줄 시를 써줘"))
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 09

## Termination 조건

---

# Termination 조건이란?

Team 대화의 *종료 시점을 결정*하는 조건

|             종료 조건              | 트리거                 |
| :--------------------------------: | ---------------------- |
|     `MaxMessageTermination(N)`     | 메시지 수 N개 도달     |
| `TextMentionTermination("키워드")` | 특정 키워드 언급       |
|     `TokenUsageTermination(N)`     | 토큰 사용량 초과       |
|      `TimeoutTermination(N)`       | N초 경과               |
|   `HandoffTermination("agent")`    | 특정 대상에게 핸드오프 |
|     `StopMessageTermination()`     | StopMessage 발생       |
|      `ExternalTermination()`       | 외부에서 `.set()` 호출 |

---

# Termination 조건 조합

```python
from autogen_agentchat.conditions import (
    MaxMessageTermination,
    TextMentionTermination,
    TimeoutTermination,
    ExternalTermination,
)

# OR 조합: 하나라도 충족되면 종료
termination = (
    TextMentionTermination("APPROVE")
    | MaxMessageTermination(20)
    | TimeoutTermination(timeout_seconds=120)
)
```

---

# Termination 조건 조합 (계속)

```python
# AND 조합: 모두 충족되어야 종료
termination = (
    MaxMessageTermination(10)
    & TextMentionTermination("DONE")
)

# 외부 제어
external = ExternalTermination()
team = RoundRobinGroupChat([agent_a, agent_b], termination_condition=external)

# 다른 코루틴에서: external.set()  → 현재 에이전트 턴 완료 후 종료
```

---

# Team 상태 관리

```python
# 대화 재개: reset 없이 run_stream()만 호출하면 이전 대화 이어감
result = await Console(team.run_stream(task="이전 분석 결과를 요약해줘"))

# 대화 초기화: 모든 Agent 히스토리 삭제
await team.reset()

# 새로운 대화 시작
result = await Console(team.run_stream(task="새로운 작업을 시작합니다"))
```

### 핵심 포인트

- `run_stream()`을 다시 호출하면 *이전 대화를 이어*감
- `team.reset()`을 호출해야 *새로운 대화*가 시작됨
- Termination 조건은 매 `run()` 호출마다 자동 초기화

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 10

## 통합 실습

---

# 실습: 리서치 에이전트 팀 구축

### 시나리오

> "AI 기술 동향을 조사하고 보고서를 작성하는 에이전트 팀"

```
┌──────────────────────────────────────────────┐
│            SelectorGroupChat                  │
│                                               │
│  ┌──────────┐  ┌──────────┐  ┌────────────┐ │
│  │ Planner  │  │Researcher│  │   Writer   │ │
│  │ 작업 계획 │  │ 정보 검색 │  │  보고서 작성 │ │
│  └──────────┘  └──────────┘  └────────────┘ │
│                                               │
│  ┌──────────┐                                │
│  │  Critic  │  → "APPROVE" 시 종료            │
│  │ 품질 검토 │                                │
│  └──────────┘                                │
└──────────────────────────────────────────────┘
```

---

# 통합 실습 코드 (1/3)

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

planner = AssistantAgent(
  "Planner",
  description="작업을 계획하고 조율하는 에이전트. 새 작업 시 먼저 참여.",
  model_client=model_client,
  system_message="""작업을 분석하고 단계별 계획을 수립하세요.
  각 단계를 적절한 전문가 에이전트에게 배정하세요.""",
)
```

---

# 통합 실습 코드 (2/3)

```python
researcher = AssistantAgent(
  "Researcher",
  description="정보를 조사하고 사실을 수집하는 에이전트.",
  model_client=model_client,
  system_message="주어진 주제에 대해 상세한 정보를 조사하세요.",
)

writer = AssistantAgent(
  "Writer",
  description="보고서를 작성하는 에이전트.",
  model_client=model_client,
  system_message="조사된 정보를 바탕으로 체계적인 보고서를 작성하세요.",
)
```

---

# 통합 실습 코드 (3/3)

```python
critic = AssistantAgent(
  "Critic",
  description="결과물의 품질을 검토하는 에이전트.",
  model_client=model_client,
  system_message="""보고서를 검토하고 피드백을 제공하세요.
  품질이 만족스러우면 'APPROVE'라고 응답하세요.""",
)

termination = TextMentionTermination("APPROVE") | MaxMessageTermination(30)

team = SelectorGroupChat(
  [planner, researcher, writer, critic],
  model_client=model_client,
  termination_condition=termination,
)

asyncio.run(Console(team.run_stream(
  task="2024-2025 생성형 AI 기술 동향 보고서를 작성해줘"
)))
```

---

# 정리: AutoGen Multi-Agent 핵심 요소

|     구성요소      | 역할               | AutoGen 구현                      |
| :---------------: | ------------------ | --------------------------------- |
|     **Agent**     | 자율적 작업 수행   | `AssistantAgent`                  |
|     **Team**      | 에이전트 협업 구조 | `RoundRobin`, `Selector`, `Swarm` |
|     **Tool**      | 외부 기능 확장     | Python 함수 + `tools=[]`          |
|    **Handoff**    | 에이전트 간 위임   | `handoffs=[]` + `HandoffMessage`  |
|  **Termination**  | 종료 조건 관리     | `TextMention`, `MaxMessage` 등    |
| **Human-in-Loop** | 사람 개입          | `UserProxyAgent`                  |

---

# 참고 자료

### 공식 문서

- **AutoGen**: https://microsoft.github.io/autogen/stable/
- **AgentChat Guide**: https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html
- **Core Guide**: https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/index.html

### Design Patterns

- Concurrent Agents / Sequential Workflow
- Group Chat / Handoffs / Mixture of Agents
- Multi-Agent Debate / Reflection / Code Execution

---

# Q&A

---

<!-- _class: closing -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 감사합니다

**2차시: Multi-Agent (AutoGen)** 완료
