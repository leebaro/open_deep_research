# Open Deep Research

Open Deep Research는 [OpenAI](https://openai.com/index/introducing-deep-research/) 및 [Gemini](https://blog.google.com/products/gemini/google-gemini-deep-research/) Deep Research와 유사한 워크플로우를 따라 모든 주제에 대한 포괄적인 보고서를 생성하는 웹 연구 보조 도구입니다. 그러나 모델, 프롬프트 (prompts), 보고서 구조 (report structure), 검색 API (search API) 및 연구 심층도 (research depth)를 사용자 정의할 수 있습니다. 구체적으로 다음 사항을 사용자 정의할 수 있습니다.

- 원하는 보고서 구조로 개요 제공
- 플래너 모델 (planner model) 설정 (예: DeepSeek, OpenAI 추론 모델 등)
- 보고서 섹션 계획에 대한 피드백을 제공하고 사용자 승인 (user approval)을 받을 때까지 반복
- 검색 API (search API) (예: Tavily, Perplexity) 및 각 연구 반복에 대해 실행할 검색 횟수 설정
- 각 섹션에 대한 검색 심층도 설정 (작성, 반영, 검색, 재작성 반복 횟수)
- 작성자 모델 (writer model) 사용자 정의 (예: Anthropic)

![report-generation](https://github.com/user-attachments/assets/6595d5cd-c981-43ec-8e8b-209e4fefc596)

## 🚀 Quickstart (빠른 시작)

원하는 도구에 대한 API 키 (API keys)가 설정되어 있는지 확인하십시오.

웹 검색 도구 (web search tool)를 선택하십시오 (기본적으로 Open Deep Research는 Tavily를 사용합니다).

* [Tavily API](https://tavily.com/) - 일반 웹 검색 (General web search)
* [Perplexity API](https://www.perplexity.ai/hub/blog/introducing-the-sonar-pro-api) - 일반 웹 검색 (General web search)
* [Exa API](https://exa.ai/) - 웹 콘텐츠를 위한 강력한 신경 검색 (Powerful neural search)
* [ArXiv](https://arxiv.org/) - 물리학, 수학, 컴퓨터 과학 등의 학술 논문
* [PubMed](https://pubmed.ncbi.nlm.nih.gov/) - MEDLINE, 생명 과학 저널 및 온라인 서적의 생물 의학 문헌

작성자 모델 (writer model)을 선택하십시오 (기본적으로 Open Deep Research는 Anthropic Claude 3.5 Sonnet을 사용합니다).

* [Anthropic](https://www.anthropic.com/)
* [OpenAI](https://openai.com/)
* [Groq](https://groq.com/)

플래너 모델 (planner model)을 선택하십시오 (기본적으로 Open Deep Research는 사고 기능이 활성화된 Claude 3.7 Sonnet을 사용합니다).

* [Anthropic](https://www.anthropic.com/)
* [OpenAI](https://openai.com/)
* [Groq](https://groq.com/)

### Using the package (패키지 사용)

(권장: 가상 환경 (virtual environment) 생성):
```
python -m venv open_deep_research
source open_deep_research/bin/activate
```

Install (설치):
```
pip install open-deep-research
```

웹 검색 (web search) 및 플래너/작성자 모델 (planner / writer models)에 대한 API 키 (API keys)가 설정되어 있는지 확인하십시오 (예시):
```bash
export TAVILY_API_KEY=<your_tavily_api_key>
export ANTHROPIC_API_KEY=<your_anthropic_api_key>
```

Jupyter 노트북에서 사용 예시는 [src/open_deep_research/graph.ipynb](src/open_deep_research/graph.ipynb)를 참조하십시오.

Import (임포트) 및 그래프 컴파일 (graph compile):
```python
from langgraph.checkpoint.memory import MemorySaver
from open_deep_research.graph import builder
memory = MemorySaver()
graph = builder.compile(checkpointer=memory)
```

그래프 보기 (View the graph):
```python
from IPython.display import Image, display
display(Image(graph.get_graph(xray=1).draw_mermaid_png()))
```

원하는 주제 (topic) 및 구성 (configuration)으로 그래프를 실행하십시오.
```python
import uuid 
thread = {"configurable": {"thread_id": str(uuid.uuid4()),
                           "search_api": "tavily",
                           "planner_provider": "openai",
                           "planner_model": "claude-3-7-sonnet-latest",
                           "writer_provider": "anthropic",
                           "writer_model": "claude-3-5-sonnet-latest",
                           "max_search_depth": 1,
                           }}

topic = "Overview of the AI inference market with focus on Fireworks, Together.ai, Groq"
async for event in graph.astream({"topic":topic,}, thread, stream_mode="updates"):
    print(event)
    print("\n")
```

보고서 계획 (report plan)이 생성되면 그래프가 중지되고 피드백을 전달하여 보고서 계획을 업데이트할 수 있습니다.
```python
from langgraph.types import Command
async for event in graph.astream(Command(resume="Include a revenue estimate (ARR) in the sections"), thread, stream_mode="updates"):
    print(event)
    print("\n")
```

보고서 계획에 만족하면 `True`를 전달하여 보고서 생성 (report generation)을 진행할 수 있습니다.
```
# Pass True to approve the report plan and proceed to report generation (보고서 계획을 승인하고 보고서 생성을 진행하려면 True를 전달하십시오)
async for event in graph.astream(Command(resume=True), thread, stream_mode="updates"):
    print(event)
    print("\n")
```

### Running LangGraph Studio UI locally (LangGraph Studio UI 로컬에서 실행)

Clone (클론) 저장소 (repository):
```bash
git clone https://github.com/langchain-ai/open_deep_research.git
cd open_deep_research
```

API 키 (API keys)로 `.env` 파일을 편집하십시오 (예: 기본 선택 항목에 대한 API 키는 아래에 나와 있습니다).

```bash
cp .env.example .env
```

모델 (model) 및 검색 도구 (search tool) 선택에 필요한 API를 설정하십시오.
```bash
export TAVILY_API_KEY=<your_tavily_api_key>
export ANTHROPIC_API_KEY=<your_anthropic_api_key>
export OPENAI_API_KEY=<your_openai_api_key>
export PERPLEXITY_API_KEY=<your_perplexity_api_key>
export EXA_API_KEY=<your_exa_api_key>
export PUBMED_API_KEY=<your_pubmed_api_key>
export PUBMED_EMAIL=<your_email@example.com>
```

LangGraph 서버 (LangGraph server)를 로컬에서 실행하여 어시스턴트 (assistant)를 시작하십시오. 브라우저에서 열립니다.

#### Mac

```bash
# Install uv package manager (uv 패키지 관리자 설치)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies and start the LangGraph server (종속성 설치 및 LangGraph 서버 시작)
uvx --refresh --from "langgraph-cli[inmem]" --with-editable . --python 3.11 langgraph dev
```

#### Windows

```powershell
# Install dependencies (종속성 설치)
pip install -e .
pip install langgraph-cli[inmem]

# Start the LangGraph server (LangGraph 서버 시작)
langgraph dev
```

다음 명령어를 사용하여 Studio UI (Studio UI)를 여십시오.
```
- 🚀 API: http://127.0.0.1:2024
- 🎨 Studio UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
- 📚 API Docs: http://127.0.0.1:2024/docs
```

(1) `Topic (주제)`을 제공하고 `Submit (제출)`을 누르십시오.

<img width="1326" alt="input" src="https://github.com/user-attachments/assets/de264b1b-8ea5-4090-8e72-e1ef1230262f" />

(2) 보고서 계획 (report plan)이 생성되어 사용자 검토를 위해 제공됩니다.

(3) 피드백이 포함된 문자열 (`"..."`)을 전달하여 피드백을 기반으로 계획을 다시 생성할 수 있습니다.

<img width="1326" alt="feedback" src="https://github.com/user-attachments/assets/c308e888-4642-4c74-bc78-76576a2da919" />

(4) 또는 `true`를 전달하여 계획을 수락할 수 있습니다.

<img width="1480" alt="accept" src="https://github.com/user-attachments/assets/ddeeb33b-fdce-494f-af8b-bd2acc1cef06" />

(5) 수락되면 보고서 섹션 (report sections)이 생성됩니다.

<img width="1326" alt="report_gen" src="https://github.com/user-attachments/assets/74ff01cc-e7ed-47b8-bd0c-4ef615253c46" />

보고서는 마크다운 (markdown)으로 생성됩니다.

<img width="1326" alt="report" src="https://github.com/user-attachments/assets/92d9f7b7-3aea-4025-be99-7fb0d4b47289" />

## 📖 Customizing the report (보고서 사용자 정의)

여러 파라미터 (parameters)를 통해 연구 보조 도구의 동작을 사용자 정의할 수 있습니다.

- `report_structure`: 보고서에 대한 사용자 정의 구조를 정의합니다 (기본값은 표준 연구 보고서 형식).
- `number_of_queries`: 섹션 당 생성할 검색 쿼리 수 (기본값: 2)
- `max_search_depth`: 최대 반영 및 검색 반복 횟수 (기본값: 2)
- `planner_provider`: 계획 단계 (planning phase)를 위한 모델 제공자 (기본값: "openai", "groq" 가능)
- `planner_model`: 계획을 위한 특정 모델 (기본값: "o3-mini", "deepseek-r1-distill-llama-70b"와 같은 Groq 호스팅 모델 가능)
- `writer_model`: 보고서 작성 (writing the report)을 위한 모델 (기본값: "claude-3-5-sonnet-latest")
- `search_api`: 웹 검색 (web searches)에 사용할 API (기본값: "tavily", 옵션: "perplexity", "exa", "arxiv", "pubmed")

이러한 구성을 통해 연구 심층도 조정부터 보고서 생성의 다양한 단계에 대한 특정 AI 모델 선택에 이르기까지 필요에 따라 연구 프로세스를 미세 조정할 수 있습니다.

### Search API Configuration (검색 API 구성)

모든 검색 API (search APIs)가 추가 구성 파라미터 (configuration parameters)를 지원하는 것은 아닙니다. 지원되는 API는 다음과 같습니다.

- **Exa**: `max_characters`, `num_results`, `include_domains`, `exclude_domains`, `subpages`
  - 참고: `include_domains`와 `exclude_domains`는 함께 사용할 수 없습니다.
  - 특히 특정 신뢰할 수 있는 소스로 연구 범위를 좁히거나 정보 정확성을 보장해야 하거나 연구에 지정된 도메인 (예: 학술 저널, 정부 사이트)을 사용해야 하는 경우에 유용합니다.
  - 특정 쿼리 (query)에 맞게 조정된 AI 생성 요약 (AI-generated summaries)을 제공하므로 검색 결과에서 관련 정보를 쉽게 추출할 수 있습니다.
- **ArXiv**: `load_max_docs`, `get_full_documents`, `load_all_available_meta`
- **PubMed**: `top_k_results`, `email`, `api_key`, `doc_content_chars_max`

Exa 구성 예시:
```python
thread = {"configurable": {"thread_id": str(uuid.uuid4()),
                           "search_api": "exa",
                           "search_api_config": {
                               "num_results": 5,
                               "include_domains": ["nature.com", "sciencedirect.com"]
                           },
                           # Other configuration...
                           }}
```

### Model Considerations (모델 고려 사항)

(1) Groq를 사용하는 경우 `on_demand` 서비스 티어 (service tier)에 따라 분당 토큰 수 (TPM, token per minute) 제한이 있습니다.
- `on_demand` 서비스 티어는 `6000 TPM`으로 제한됩니다.
- Groq 모델로 섹션 작성 (section writing)을 하려면 [유료 플랜](https://github.com/cline/cline/issues/47#issuecomment-2640992272)이 필요합니다.

(2) `deepseek`은 [함수 호출 (function calling)에 적합하지 않습니다](https://api-docs.deepseek.com/guides/reasoning_model). 저희 어시스턴트 (assistant)는 함수 호출을 사용하여 보고서 섹션 (report sections) 및 각 섹션 내의 검색 쿼리 (search queries)에 대한 구조화된 출력 (structured outputs)을 생성합니다.
- 섹션 작성은 더 많은 수의 함수 호출을 수행하므로 OpenAI, Anthropic 및 Groq의 `llama-3.3-70b-versatile`과 같이 함수 호출에 능숙한 특정 OSS 모델을 사용하는 것이 좋습니다.
- 다음과 같은 오류가 발생하면 모델이 구조화된 출력 (structured outputs)을 생성할 수 없기 때문일 수 있습니다 ([trace](https://smith.langchain.com/public/8a6da065-3b8b-4a92-8df7-5468da336cbe/r) 참조).
```
groq.APIError: Failed to call a function. Please adjust your prompt. See 'failed_generation' for more details.
```

## How it works (작동 방식)
   
1. `Plan and Execute (계획 및 실행)` - Open Deep Research는 계획과 연구를 분리하는 [plan-and-execute workflow](https://github.com/assafelovic/gpt-researcher)를 따르므로 시간이 더 많이 소요되는 연구 단계 전에 사람이 보고서 계획을 승인할 수 있습니다. 기본적으로 [추론 모델](https://www.youtube.com/watch?v=f0RbwrBcFmc)을 사용하여 보고서 섹션을 계획합니다. 이 단계에서는 웹 검색 (web search)을 사용하여 보고서 주제 (report topic)에 대한 일반적인 정보를 수집하여 보고서 섹션 계획에 도움을 줍니다. 그러나 사용자의 보고서 구조 (report structure)와 보고서 계획에 대한 사람의 피드백도 허용합니다.
   
2. `Research and Write (연구 및 작성)` - 보고서의 각 섹션은 병렬로 작성됩니다. 연구 보조 도구는 [Tavily API](https://tavily.com/), [Perplexity](https://www.perplexity.ai/hub/blog/introducing-the-sonar-pro-api), [Exa](https://exa.ai/), [ArXiv](https://arxiv.org/) 또는 [PubMed](https://pubmed.ncbi.nlm.nih.gov/)를 통해 웹 검색 (web search)을 사용하여 각 섹션 주제 (section topic)에 대한 정보를 수집합니다. 각 보고서 섹션을 반영하고 웹 검색에 대한 후속 질문을 제안합니다. 이 연구 "심층도"는 사용자가 원하는 만큼 여러 번 반복됩니다. 서론 및 결론과 같은 최종 섹션은 보고서의 본문이 작성된 후에 작성되므로 보고서가 응집력 있고 일관성을 유지하는 데 도움이 됩니다. 플래너 (planner)는 계획 단계에서 본문과 최종 섹션을 결정합니다.

3. `Managing different types (다양한 유형 관리)` - Open Deep Research는 [어시스턴트 (assistants)를 사용하여](https://langchain-ai.github.io/langgraph/concepts/assistants/) 구성 관리 (configuration management)를 기본적으로 지원하는 LangGraph를 기반으로 구축되었습니다. 보고서 `structure (구조)`는 그래프 구성 (graph configuration)의 필드 (field)이므로 사용자는 다양한 유형의 보고서에 대해 다양한 어시스턴트 (assistants)를 만들 수 있습니다.

## UX

### Local deployment (로컬 배포)

[quickstart (빠른 시작)](#quickstart)에 따라 LangGraph 서버 (LangGraph server)를 로컬에서 시작하십시오.

### Hosted deployment (호스팅 배포)
 
[LangGraph Platform](https://langchain-ai.github.io/langgraph/concepts/#deployment-options)에 쉽게 배포할 수 있습니다.
