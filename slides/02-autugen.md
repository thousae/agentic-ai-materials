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

<div class="columns">
<div>

### Part 3: 실전 심화 (1)

7. Tool Use (도구 사용)
8. FunctionTool과 고급 도구 패턴
9. Human-in-the-Loop (사람 개입)
10. Termination 조건

</div>
<div>

### Part 3: 실전 심화 (2)

11. 에이전트 상태 저장과 복원
12. 디버깅과 모니터링
13. 통합 실습: 코드 리뷰 에이전트 팀
14. 실전 팁과 베스트 프랙티스

</div>
</div>

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
from autogen_agentchat.messages import BaseAgentEvent, BaseChatMessage

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

# Agent에 Tool 등록

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

### `reflect_on_tool_use` 옵션

| 값        | 동작                                    |
| --------- | --------------------------------------- |
| `False`   | 도구 결과를 **그대로** 메시지로 반환     |
| `True`    | LLM이 도구 결과를 **자연어로 정리**하여 반환 |


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

# 실전 Tool 예시: 웹 검색 + DB 조회

```python
import httpx

async def search_web(query: str) -> str:
    async with httpx.AsyncClient() as client:
        resp = await client.get(
            "https://api.search.example.com/search",
            params={"q": query, "limit": 3},
        )
        results = resp.json()["results"]
        return "\n".join(f"- {r['title']}: {r['snippet']}" for r in results)

async def query_database(sql: str) -> str:
    """읽기 전용 SQL 쿼리를 실행합니다. SELECT만 허용됩니다."""
    if not sql.strip().upper().startswith("SELECT"):
        return "오류: SELECT 쿼리만 허용됩니다."
    return f"쿼리 결과: [{sql}] 실행 완료 - 3건 조회"
```

> 도구의 **독스트링**이 LLM에게 도구 설명으로 전달됨 - 명확하게 작성할수록 정확한 호출

---

# Team에서 Tool 공유 vs 전담

<div class="columns">
<div>

### 전략 A: Tool 전담 Agent

```python
searcher = AssistantAgent(
    "Searcher", tools=[search_web],
    model_client=model_client,
    system_message="검색 전문가입니다.",
)
analyst = AssistantAgent(
    "Analyst", tools=[query_database, calculate],
    model_client=model_client,
    system_message="데이터 분석가입니다.",
)
```

각 Agent가 특정 도구만 담당

</div>
<div>

### 전략 B: 범용 Agent

```python
generalist = AssistantAgent(
    "Generalist",
    tools=[
        search_web,
        query_database,
        calculate,
    ],
    model_client=model_client,
    system_message="모든 도구를 활용하세요.",
)
```

하나의 Agent가 모든 도구 사용

</div>
</div>

> 일반적으로 **전략 A**(전담)가 역할이 명확하고 디버깅이 쉬움

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 08

## FunctionTool과 고급 도구 패턴

---

# FunctionTool 클래스

함수를 도구로 래핑하여 _스키마를 세밀하게 제어_

```python
from autogen_core.tools import FunctionTool

async def translate(text: str, target_lang: str) -> str:
    """텍스트를 대상 언어로 번역합니다."""
    return f"[{target_lang}] {text} 번역 완료"

translate_tool = FunctionTool(
    func=translate,
    name="translate_text",                       # 커스텀 이름
    description="텍스트를 지정된 언어로 번역합니다. "
                "target_lang은 ISO 639-1 코드(ko, en, ja 등)를 사용합니다.",
)

agent = AssistantAgent(
    name="translator",
    model_client=model_client,
    tools=[translate_tool],                      # FunctionTool 객체 전달
)
```

---

# FunctionTool vs 함수 직접 전달

| 항목           | 함수 직접 전달         | FunctionTool                     |
| -------------- | ---------------------- | -------------------------------- |
| **이름**       | 함수명 자동 사용       | `name` 파라미터로 커스터마이즈     |
| **설명**       | 독스트링 자동 사용     | `description`으로 상세 설명 가능  |
| **스키마**     | 타입 힌트에서 자동 추론 | 동일 (자동 추론)                  |
| **사용 시점**  | 빠른 프로토타이핑      | 프로덕션, 설명이 중요한 도구      |

```python
# 간단한 도구 → 함수 직접 전달
agent = AssistantAgent("a", tools=[get_weather], model_client=model_client)

# 정교한 도구 → FunctionTool 사용
weather_tool = FunctionTool(
    func=get_weather,
    name="check_current_weather",
    description="실시간 날씨를 조회합니다. city는 한글 도시명(예: 서울, 부산)을 사용합니다.",
)
agent = AssistantAgent("a", tools=[weather_tool], model_client=model_client)
```

---

<style scoped>
  pre { font-size: 0.62em; }
</style>

# 도구 에러 처리 패턴

도구 실행 실패 시 _에러 메시지를 반환_하면 LLM이 대처 가능

```python
import httpx

async def fetch_stock_price(ticker: str) -> str:
    """주식 종목의 현재 가격을 조회합니다. ticker는 영문 심볼(예: AAPL, TSLA)입니다."""
    try:
        async with httpx.AsyncClient(timeout=10.0) as client:
            resp = await client.get(f"https://api.example.com/stock/{ticker}")
            resp.raise_for_status()
            data = resp.json()
            return f"{ticker}: ${data['price']:.2f} ({data['change']:+.2f}%)"
    except httpx.TimeoutException:
        return f"오류: {ticker} 가격 조회 시 타임아웃이 발생했습니다. 잠시 후 다시 시도해주세요."
    except httpx.HTTPStatusError as e:
        return f"오류: {ticker} 조회 실패 (HTTP {e.response.status_code}). 심볼을 확인해주세요."
    except Exception as e:
        return f"오류: 예상치 못한 문제가 발생했습니다 - {str(e)}"
```

> 예외를 **raise 하지 말고** 문자열로 반환 → LLM이 오류를 이해하고 다음 행동 결정

---

# 도구 조합 패턴: 파이프라인

여러 도구를 _순차적으로 호출_하는 워크플로우 구성

```
사용자: "AAPL 주가를 조회하고 한국어로 요약해줘"
    │
    ↓
┌───────────────┐  tool call 1      ┌────────────────┐
│ AssistantAgent│ ─────────────────→ │fetch_stock_price│
│               │                   │("AAPL")         │
│               │ ←──────────────── │→ "AAPL: $198.50"│
│               │                   └────────────────┘
│               │  tool call 2      ┌────────────────┐
│               │ ─────────────────→ │  translate     │
│               │                   │("AAPL: $198",  │
│               │ ←──────────────── │ "ko")           │
│               │                   └────────────────┘
│               │
│  reflect: "애플 주가는 198.50달러입니다."
└───────────────┘
```

> Agent가 도구 결과를 보고 **자율적으로** 다음 도구 호출 여부를 판단

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 09

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

# Human-in-the-Loop: 대화 중 개입 실습

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

# Human-in-the-Loop: Swarm에서의 사용자 핸드오프

Swarm 패턴에서 _사용자 확인이 필요한 시점에 자동 핸드오프_ -> 중요한 결정 전에 사용자 승인을 받아야함

```python
from autogen_agentchat.agents import UserProxyAgent

user = UserProxyAgent("user", input_func=input)
order_agent = AssistantAgent(
    "order_agent", model_client=model_client,
    handoffs=["payment_agent", "user"],    # 사용자에게도 핸드오프 가능
    system_message="""주문을 처리합니다. 결제 전 반드시 user에게 핸드오프하여 주문 내용을 확인받으세요.""",
)

payment_agent = AssistantAgent(
    "payment_agent", model_client=model_client,
    handoffs=["order_agent", "user"],
    tools=[process_payment],
    system_message="결제를 처리합니다.",
)
```
---

# Human-in-the-Loop 실행 흐름 (Swarm)

```
사용자: "노트북 1대 주문해줘"
    │
    ↓
┌──────────────┐  "주문 내용: 노트북 1대, 1,200,000원"
│ order_agent  │ ─→ HandoffMessage(target="user")   ← 확인 요청
└──────────────┘
    │
    ↓  사용자: "네, 진행해주세요"
    │
┌──────────────┐  "결제를 진행합니다"
│ order_agent  │ ─→ HandoffMessage(target="payment_agent")
└──────────────┘
    │
    ↓
┌───────────────┐  process_payment() 호출
│ payment_agent │ ─→ HandoffMessage(target="order_agent")
└───────────────┘
    │
    ↓
┌──────────────┐  "주문이 완료되었습니다. TERMINATE"
│ order_agent  │
└──────────────┘
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 10

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

# Termination 설계 가이드

### 왜 종료 조건이 중요한가?

종료 조건 없이 실행하면 _무한 루프, 토큰 과다 소모, 비용 폭증_ 위험

### 실전 설계 원칙

```
┌─────────────────────────────────────────────────┐
│          Termination 조건 설계 체크리스트          │
│                                                   │
│  ✓ 정상 종료: TextMentionTermination("APPROVE")  │
│    → Agent가 작업 완료를 명시적으로 선언            │
│                                                   │
│  ✓ 안전 장치: MaxMessageTermination(30)           │
│    → 무한 루프 방지용 상한선                       │
│                                                   │
│  ✓ 시간 제한: TimeoutTermination(300)             │
│    → 프로덕션 환경의 SLA 준수                      │
│                                                   │
│  항상 OR 조합으로 안전 장치를 포함하세요!           │
└─────────────────────────────────────────────────┘
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

# 11

## 에이전트 상태 저장과 복원

---

# 왜 상태 저장이 필요한가?

프로덕션 환경에서 _장기 실행 에이전트의 상태를 보존_

```
┌─────────────┐     저장      ┌─────────────┐
│  실행 중인   │ ──────────→  │   JSON 파일  │
│  Team 상태   │              │  또는 DB     │
└─────────────┘              └──────┬──────┘
                                    │
                              복원  │
                                    ↓
                             ┌─────────────┐
                             │  새 세션에서  │
                             │  대화 재개    │
                             └─────────────┘
```

### 활용 시나리오

- 서버 재시작 후 대화 재개
- 긴 작업을 여러 세션에 걸쳐 수행
- 체크포인트 기반 복구

---

# Team 상태 저장/복원

```python
import json

# 1. Team 실행
team = RoundRobinGroupChat([writer, critic], termination_condition=termination)
result = await Console(team.run_stream(task="AI 윤리에 대한 에세이를 써줘"))

# 2. 상태 저장
state = await team.save_state()
with open("team_state.json", "w") as f:
    json.dump(state, f, ensure_ascii=False)

# 3. 상태 복원 (새 세션에서)
team2 = RoundRobinGroupChat([writer, critic], termination_condition=termination)
with open("team_state.json", "r") as f:
    state = json.load(f)
await team2.load_state(state)

# 4. 이전 대화 이어서 진행
result = await Console(team2.run_stream(task="결론 부분을 보강해줘"))
```

---

# Agent 단위 상태 저장/복원

###
###

<div class="columns">
<div>

```python
# 개별 Agent의 상태도 독립적으로 저장 가능
agent_state = await writer.save_state()

# 새 Agent 인스턴스에 상태 복원
new_writer = AssistantAgent(
    "writer",
    model_client=model_client,
    system_message="당신은 창의적인 작가입니다.",
)
await new_writer.load_state(agent_state)
```
</div>
<div>

### 상태에 포함되는 것

| 포함됨                | 포함되지 않음        |
| --------------------- | -------------------- |
| 대화 히스토리 (메시지) | `model_client` 설정   |
| 모델 컨텍스트         | 도구 함수 정의       |
| 에이전트 내부 상태     | `system_message`     |

</div>
</div>

> 복원 시 Agent 설정(model_client, tools, system_message)은 동일하게 생성해야 함

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 12

## 디버깅과 모니터링

---

# 로깅 설정

AutoGen의 내부 동작을 _로그로 추적_하여 디버깅

```python
import logging

# AutoGen 전체 로깅 활성화
logging.basicConfig(level=logging.WARNING)
logging.getLogger("autogen_agentchat").setLevel(logging.DEBUG)
logging.getLogger("autogen_core").setLevel(logging.DEBUG)
```

### 주요 로그 포인트

| 로그 레벨 | 내용                             |
| --------- | -------------------------------- |
| `DEBUG`   | 메시지 송수신, 도구 호출 상세    |
| `INFO`    | Agent 선택, 핸드오프 이벤트     |
| `WARNING` | Termination 트리거, 재시도       |
| `ERROR`   | LLM 호출 실패, 도구 실행 오류   |

---

# 실행 결과 분석

`TaskResult` 객체에서 _전체 대화 흐름을 분석_

```python
result = await team.run(task="보고서를 작성해줘")

# 전체 메시지 히스토리 확인
for msg in result.messages:
    print(f"[{msg.source}] {type(msg).__name__}: {msg.content[:80]}...")

print(f"종료 사유: {result.stop_reason}")
```

### 출력 예시

```
[user]       TextMessage:    보고서를 작성해줘
[Planner]    TextMessage:    작업을 분해하겠습니다. 1) 자료 조사 2) 초안 작성...
[Researcher] TextMessage:    조사 결과: AI 시장 규모는 2025년 기준...
[Writer]     TextMessage:    ## AI 기술 동향 보고서...
[Critic]     TextMessage:    APPROVE - 잘 작성되었습니다.
종료 사유: Text 'APPROVE' mentioned
```

---

# 토큰 사용량 추적

비용 관리를 위한 _토큰 사용량 모니터링_

```python
result = await team.run(task="보고서를 작성해줘")

# 메시지별 토큰 사용량 확인
for msg in result.messages:
    if hasattr(msg, "models_usage") and msg.models_usage:
        usage = msg.models_usage
        print(f"[{msg.source}] "
              f"입력: {usage.prompt_tokens}, "
              f"출력: {usage.completion_tokens}")
```

### 비용 최적화 팁

- `MaxMessageTermination`으로 **상한선** 설정
- 불필요한 Agent 참여 방지 → `selector_func` 활용
- `system_message`를 간결하게 → 매 턴마다 전송되므로 토큰 누적

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 13

## 통합 실습: 코드 리뷰 에이전트 팀

---

# 실습 시나리오

> "코드를 분석하고 리뷰 의견을 제공하는 멀티 에이전트 팀"

```
┌───────────────────────────────────────────────────────┐
│              SelectorGroupChat                         │
│                                                        │
│  ┌────────────┐  ┌──────────────┐  ┌───────────────┐ │
│  │  Architect  │  │SecurityReview│  │CodeOptimizer │ │
│  │  구조 분석   │  │  보안 검토    │  │  성능 최적화   │ │
│  └────────────┘  └──────────────┘  └───────────────┘ │
│                                                        │
│  ┌────────────┐                                       │
│  │ Summarizer │  → "REVIEW_COMPLETE" 시 종료           │
│  │ 종합 리뷰   │                                       │
│  └────────────┘                                       │
└───────────────────────────────────────────────────────┘
```

---

<style scoped>
  pre { font-size: 0.55em; }
</style>

# 코드 리뷰 팀 구현 (1/3): Tool 정의

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

async def count_lines(code: str) -> str:
    """코드의 줄 수와 함수 수를 분석합니다."""
    lines = code.strip().split("\n")
    func_count = sum(1 for line in lines if line.strip().startswith("def "))
    class_count = sum(1 for line in lines if line.strip().startswith("class "))
    return f"총 {len(lines)}줄, 함수 {func_count}개, 클래스 {class_count}개"

async def check_security_patterns(code: str) -> str:
    """코드에서 보안 취약점 패턴을 검사합니다."""
    issues = []
    if "eval(" in code: issues.append("eval() 사용 감지 - 코드 인젝션 위험")
    if "exec(" in code: issues.append("exec() 사용 감지 - 코드 인젝션 위험")
    if "password" in code.lower() and "=" in code: issues.append("하드코딩된 비밀번호 의심")
    if "SELECT" in code and "+" in code: issues.append("SQL 인젝션 가능성")
    return "\n".join(issues) if issues else "주요 보안 패턴 이슈 없음"
```

---

<style scoped>
  pre { font-size: 0.55em; }
</style>

# 코드 리뷰 팀 구현 (2/3): Agent 정의

```python
architect = AssistantAgent(
    "Architect",
    description="코드의 구조, 설계 패턴, 아키텍처를 분석하는 에이전트.",
    model_client=model_client,
    tools=[count_lines],
    system_message="""코드의 구조적 품질을 분석하세요:
    - SOLID 원칙 준수 여부, 함수/클래스 설계, 의존성 구조, 네이밍 컨벤션""",
)
security_reviewer = AssistantAgent(
    "SecurityReviewer",
    description="코드의 보안 취약점을 검토하는 에이전트.",
    model_client=model_client,
    tools=[check_security_patterns],
    system_message="""코드의 보안 취약점을 검토하세요:
    - 입력 검증, 인젝션 위험, 인증/인가, 민감 데이터 노출""",
)
optimizer = AssistantAgent(
    "CodeOptimizer",
    description="코드의 성능과 효율성을 개선하는 에이전트.",
    model_client=model_client,
    system_message="""코드의 성능 최적화 방안을 제시하세요:
    - 시간/공간 복잡도, 불필요한 연산, 캐싱 기회, 알고리즘 개선""",
)
```

---

<style scoped>
  pre { font-size: 0.55em; }
</style>

# 코드 리뷰 팀 구현 (3/3): Team 구성 및 실행

```python
summarizer = AssistantAgent(
    "Summarizer",
    description="모든 리뷰 의견을 종합하여 최종 리뷰를 작성하는 에이전트.",
    model_client=model_client,
    system_message="""다른 에이전트들의 리뷰를 종합하여 최종 코드 리뷰를 작성하세요.
    형식: ## 종합 코드 리뷰 / 심각도(상/중/하) 포함 / 개선 제안 포함
    최종 리뷰 작성 완료 시 반드시 'REVIEW_COMPLETE'라고 마지막에 적으세요.""",
)

termination = TextMentionTermination("REVIEW_COMPLETE") | MaxMessageTermination(20)

team = SelectorGroupChat(
    [architect, security_reviewer, optimizer, summarizer],
    model_client=model_client,
    termination_condition=termination,
)

code_to_review = """
def get_user(user_id):
    query = "SELECT * FROM users WHERE id = " + str(user_id)
    result = eval(db.execute(query))
    password = "admin123"
    return result
"""

asyncio.run(Console(team.run_stream(task=f"다음 코드를 리뷰해주세요:\n```python\n{code_to_review}\n```")))
```

---

# 코드 리뷰 실행 흐름 예시

```
[Architect]       count_lines() 호출 → "총 5줄, 함수 1개"
                  "단일 함수에 DB 조회 + 인증 로직이 혼재.
                   관심사 분리(SoC) 원칙 위반."

[SecurityReviewer] check_security_patterns() 호출
                  "🔴 eval() 사용 - 코드 인젝션 위험
                   🔴 SQL 인젝션 가능성
                   🔴 하드코딩된 비밀번호"

[CodeOptimizer]   "SELECT * 대신 필요한 컬럼만 조회.
                   파라미터 바인딩으로 쿼리 최적화 가능."

[Summarizer]      "## 종합 코드 리뷰
                   심각도 상: eval() 제거, SQL 파라미터 바인딩
                   심각도 상: 비밀번호 환경변수로 이동
                   심각도 중: 관심사 분리 리팩토링
                   REVIEW_COMPLETE"
```

---

<!-- _class: section-divider -->
<!-- _paginate: false -->
<!-- _footer: "" -->

# 14

## 실전 팁과 베스트 프랙티스

---

# Agent 설계 베스트 프랙티스

### 1. 역할을 명확하게

```python
agent = AssistantAgent("agent1", system_message="도움을 주세요.") # ❌ 모호한 설계

agent = AssistantAgent(
    "DataAnalyst",
    description="수치 데이터를 분석하고 통계적 인사이트를 제공하는 에이전트.",
    system_message="""당신은 데이터 분석 전문가입니다.
    - 항상 수치적 근거를 포함하세요
    - 분석 결과를 표 형식으로 정리하세요
    - 불확실한 경우 추가 데이터를 요청하세요""",
) # ✅ 명확한 역할 + 행동 지침

```

### 2. `description`은 Selector를 위한 것

- `system_message`: Agent 자신이 읽는 행동 지침
- `description`: SelectorGroupChat이 Agent를 **선택**할 때 참고하는 설명

---

# 흔한 실수와 해결법

| 실수 | 증상 | 해결법 |
| --- | --- | --- |
| 종료 조건 누락 | 무한 루프, 비용 폭증 | `MaxMessageTermination` 항상 포함 |
| Agent 수 과다 | 느린 응답, 비효율적 대화 | 2~4개 Agent로 시작 |
| 모호한 역할 | Selector가 잘못된 Agent 선택 | `description` 구체적으로 작성 |
| 도구 에러 미처리 | Agent가 오류에 대처 못함 | 문자열로 에러 반환 |
| `reset()` 미호출 | 이전 대화가 누적 | 새 작업 시 `team.reset()` |

---

# Team 패턴 선택 가이드

```
작업의 성격은?
    │
    ├─ 순서가 정해진 반복 피드백 ──→  RoundRobinGroupChat
    │   (Writer ↔ Critic)             - 예측 가능한 흐름
    │                                  - 간단한 설정
    │
    ├─ 전문가 협업, 동적 선택 ──────→  SelectorGroupChat
    │   (Planner + 전문가들)           - LLM 기반 라우팅
    │                                  - 유연한 참여
    │
    └─ 워크플로우 기반, 분기 있음 ──→  Swarm
        (고객 서비스, 주문 처리)        - 자율 핸드오프
                                       - 사용자 개입 용이
```

---

# 확장: Nested Chat (팀 안의 팀)

복잡한 작업은 _팀 자체를 하나의 Agent처럼_ 중첩 가능

```python
# 내부 팀: Writer + Critic (에세이 작성)
inner_team = RoundRobinGroupChat(
    [writer, critic],
    termination_condition=TextMentionTermination("APPROVE"),
)

# 외부 팀: Planner + 내부 팀 + Reviewer
outer_team = SelectorGroupChat(
    [planner, inner_team, final_reviewer],
    model_client=model_client,
    termination_condition=termination,
)
```

> 하위 Team이 하나의 Agent처럼 동작하여 **계층적 구조** 구성 가능

---

# 정리: AutoGen Multi-Agent 핵심 요소

|     구성요소      | 역할               | AutoGen 구현                        |
| :---------------: | ------------------ | ----------------------------------- |
|     **Agent**     | 자율적 작업 수행   | `AssistantAgent`, `UserProxyAgent`  |
|     **Team**      | 에이전트 협업 구조 | `RoundRobin`, `Selector`, `Swarm`   |
|     **Tool**      | 외부 기능 확장     | Python 함수, `FunctionTool`         |
|    **Handoff**    | 에이전트 간 위임   | `handoffs=[]` + `HandoffMessage`    |
|  **Termination**  | 종료 조건 관리     | `TextMention`, `MaxMessage` 등      |
| **Human-in-Loop** | 사람 개입          | `UserProxyAgent`, Swarm 핸드오프    |
|    **State**      | 상태 저장/복원     | `save_state()` / `load_state()`     |

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
