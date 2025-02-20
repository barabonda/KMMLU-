# KMMLU Criminal-Law Agent System

본 프로젝트는 KMMLU Criminal-Law 테스트셋을 활용하여, GPT‑4o‑mini와 text‑embedding‑small 모델 기반의 LLM 에이전트를 구축하고 평가하는 파이프라인입니다. 이 시스템은 법률 문서(예: PDF, QA 데이터, 판례 등)에서 필요한 정보를 검색(FAISS, BM25 기반)하여 LLM 프롬프트에 포함하고, OpenAI Batch API를 통해 평가 결과를 산출합니다.

## 1. 프로젝트 개요

### 1.1 시스템 목적 및 주요 기능
- **목적**: KMMLU Criminal-Law 테스트셋을 사용해 에이전트의 질의응답 성능을 평가하고, 법률 문서에서 필요한 정보를 신속하게 추출하여 LLM 응답의 품질을 높이는 것을 목표로 합니다.
- **주요 기능**:
  - **즉시 생성 모드 (Immediate Mode)**: CSV 파일에 기록된 질문을 순차적으로 처리하여 LLM 응답을 생성하고 평가 결과를 산출합니다.
  - **배치 평가 모드 (Batch Mode)**: JSONL 형식의 질문 데이터를 OpenAI Batch API를 통해 일괄 평가하고, 최종 평가 결과 파일을 생성합니다.
  - **문서 검색**: FAISS와 BM25 기반의 앙상블 리트리버를 활용해 법률 문서에서 관련 정보를 추출합니다.
- **제한 사항**: 모든 코드는 Docker 컨테이너 내에서 실행되어야 하며, Poetry와 pyproject.toml을 사용해 의존성을 관리합니다. LLM은 반드시 GPT‑4o‑mini, 임베딩은 text‑embedding‑small을 사용합니다.

### 1.2 전체 시스템 구성
아래 도식은 전체 파이프라인의 구성과 각 모듈의 역할을 추상화한 개요도입니다.

```mermaid
flowchart TD
    A[데이터 로더 (data_loader.py)]
    B[에이전트 (agent.py)]
    C[메인 실행 스크립트 (main.py)]
    D[평가 스크립트 (evaluation.py)]
    E[법률 문서 (PDF, QA, 판례 등)]
    F[질문 데이터 (CSV/JSONL)]
    G[검색 인덱스 (FAISS, BM25)]
    
    E --> A
    A --> G
    F --> C
    G --> B
    B --> C
    C --> D
```

- **data_loader.py**: 다양한 소스의 문서를 로드하고 전처리합니다.
- **agent.py**: LLM 응답 생성 및 문서 검색(FAISS, BM25)을 통해 프롬프트를 구성합니다.
- **main.py**: 전체 파이프라인의 진입점으로, 즉시 생성 모드와 배치 평가 모드를 선택하여 실행합니다.
- **evaluation.py**: OpenAI Batch API를 활용한 평가 과정을 구현하며, 평가 결과 파일을 생성합니다.

## 2. 설치 및 환경 설정

### 2.1 폴더 이동
```bash
cd kmmlu-criminal-law-agent
```

### 2.2 Poetry를 통한 의존성 설치
```bash
poetry install
```
> `pyproject.toml` 및 `poetry.lock` 파일에 명시된 의존성이 설치됩니다.

### 2.3 환경 변수 설정
OpenAI API를 사용하기 위해 발급받은 API 키를 설정합니다.
```bash
export OPENAI_API_KEY="sk-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```
Windows PowerShell:
```powershell
$Env:OPENAI_API_KEY="sk-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```

## 3. Docker 컨테이너 셋업

### 3.1 Docker 이미지 빌드
프로젝트 디렉토리 내의 Dockerfile을 기반으로 이미지를 빌드합니다.
```bash
docker build -t kmmlu-agent .
```

### 3.2 Docker 컨테이너 실행

#### 즉시 생성 모드 실행
CSV 파일(`data/Criminal-Law-test.csv`)에 기록된 질문을 순차적으로 처리합니다.
```bash
docker run -it --rm -e OPENAI_API_KEY=$OPENAI_API_KEY kmmlu-agent python main.py immediate
```

#### 배치 평가 모드 실행 (최종 결과 산출)
OpenAI Batch API를 활용해 배치 평가를 진행하고, 최종 결과 파일을 생성합니다.
```bash
docker run -it --rm -e OPENAI_API_KEY=$OPENAI_API_KEY kmmlu-agent python main.py batch --input batchinput.jsonl --output output.jsonl
```

### 3.3 Docker Compose를 통한 실행
```bash
docker-compose up --build
```

## 4. 에이전트 평가 파이프라인 실행

### 4.1 즉시 생성 모드 (Immediate Mode)
- **기능**: CSV 파일에 기록된 질문들을 불러와, 각 질문에 대해 LLM 응답을 생성하고, 평가 결과를 `evaluation_results_immediate.csv`로 저장합니다.
- **실행 명령어 (Poetry 사용)**:
  ```bash
  poetry run python main.py immediate
  ```

### 4.2 배치 평가 모드 (Batch Mode)
- **기능**: JSONL 형식의 질문 데이터를 기반으로 OpenAI Batch API를 통해 일괄 평가를 진행합니다.
- **실행 명령어 (Poetry 사용)**:
  ```bash
  poetry run python main.py batch --input batchinput.jsonl --output output.jsonl
  ```


## 5. 트러블슈팅 및 학습 내용

### 5.1 트러블슈팅
- **API Rate Limit**: OpenAI API 호출 시 Rate Limit에 걸리지 않도록 재시도 로직(최대 3회)을 agent.py 내에 구현하였습니다.
- **파일 경로 문제**: 데이터 파일 경로 설정과 glob 패턴 사용 시 경로가 올바른지 확인하여 문제를 해결했습니다.
- **컨테이너 환경 변수 전달**: Docker 컨테이너 실행 시 `-e OPENAI_API_KEY` 옵션을 통해 환경 변수가 올바르게 전달되도록 수정했습니다.

### 5.2 과제 수행 중 배운 점
- **전체 파이프라인 통합의 중요성**: 데이터 로딩부터 평가까지 모든 단계가 유기적으로 연동되어야 함을 직접 체감했습니다.
- **Docker와 Poetry의 효율성**: 개발 환경을 통일하고 의존성을 명확히 관리함으로써, 재현성과 유지보수에 큰 도움을 받았습니다.
- **OpenAI Batch API 활용**: 대규모 평가를 위한 배치 처리의 효율성을 배우고, API 호출 최적화 및 에러 핸들링에 대해 심도 있게 학습했습니다.
- **RAG 파이프라인의 깊이**: 짧은 시간안에 몰입하여 파이프라인을 짜면서 

## 6. 제출 및 평가 관련 주의사항

- 전체 파일 크기는 2GB 이하로 관리해야 합니다.
- 최종 평가에 사용된 batch API의 input, output JSONL 파일과 벤치마크 점수를 반드시 제출해야 합니다.
- 모든 코드는 Docker 컨테이너 내에서 실행되어야 하며, Poetry로 의존성을 관리해야 합니다.
- 전체 파이프라인(에이전트 시스템 구축부터 KMMLU 평가까지)은 1시간 이내에 실행되어야 합니다(배치 API 응답 시간 제외).

7. 고찰 및 선택 이유
본 파이프라인에서 여러 정보 검색 및 응답 생성 기법을 결합한 이유는, 법률 문서와 같이 전문 분야에서 정확도와 포괄성을 동시에 만족시키기 위함입니다.

BM25
BM25는 오랜 시간 동안 정보 검색 분야에서 검증된 전통적인 키워드 기반 검색 알고리즘입니다.

선택 이유: 법률 문서에서는 특정 용어(예: 조항 번호, 법률 용어 등)가 매우 중요합니다. BM25는 이러한 명시적 키워드의 빈도와 역문서 빈도(IDF)를 기반으로 문서 간 관련성을 산출하여, 질문과 관련된 핵심 문서를 빠르게 선별할 수 있습니다.
고찰: BM25는 단순하지만 강력한 성능을 보이며, 특히 전문 분야에서의 기본 검색 성능을 보장합니다. 다만, 단순 키워드 매칭의 한계로 인해 문맥적 의미를 완전히 반영하지는 못하므로, 다른 기법과의 조합이 필요합니다.
FAISS
FAISS는 Facebook AI Research에서 개발한 대규모 벡터 검색 라이브러리로, 문서 임베딩 간의 유사도를 효율적으로 계산할 수 있습니다.

선택 이유: LLM 기반 응답 생성에 있어서 문장의 의미적 유사도를 반영하는 것은 매우 중요합니다. FAISS를 통해 법률 문서를 벡터화하면, 질문과 의미적으로 가까운 문서들을 빠르게 검색할 수 있어, 보다 풍부한 문맥 정보를 제공할 수 있습니다.
고찰: FAISS는 임베딩 기반의 검색을 가능하게 하여, 단순 키워드 매칭의 한계를 극복합니다. 그러나 벡터 검색은 때때로 노이즈가 포함될 수 있으므로, BM25와 같이 보완적인 기법과의 조합이 필요합니다.
멀티쿼리 리트리버 (MultiQueryRetriever)
멀티쿼리 리트리버는 단일 질의에 대해 여러 관점에서 재작성된 쿼리를 생성하여, 다양한 검색 결과를 통합하는 기법입니다.

선택 이유: 법률 문제와 같이 복잡하고 다의적인 질문의 경우, 하나의 질의만으로 모든 관련 정보를 찾기는 어렵습니다. 멀티쿼리 접근법을 통해 여러 번의 재질의를 수행하면, 서로 다른 측면의 관련 문서를 보다 포괄적으로 검색할 수 있습니다.
고찰: 멀티쿼리 리트리버는 질문의 모호성을 해소하고, 누락된 정보를 보완하는 데 효과적입니다. 이를 통해 최종 LLM 프롬프트에 더욱 다양한 문맥 정보를 제공하여 응답의 품질을 향상시킬 수 있습니다.
이와 같이 각 검색 기법은 서로의 강점을 보완하도록 설계되었으며, 법률 문서와 같이 방대한 정보와 다양한 연관 관계를 지닌 도메인에서 신뢰성 높은 응답을 생성하는 데 크게 기여합니다. 최종적으로 이러한 기법의 조합은 LLM이 보다 정확하고, 다각적인 근거를 바탕으로 답변을 생성할 수 있도록 돕습니다.

# 과제를 마치며.
짧은 시간이었지만 이 과제를 하며 다뤄보지않았던 법률 도메인에 대한 RAG 역량을 향상 시킬 수 있었습니다.
다른 적용시켜보고 싶었던 기술도 많이 생겼고 이를 바탕으로 법률과 같은 도메인은 데이터도 중요하지만 그 데이터의 본질을 알 수 있게 해주는 그런 기법이 필요하다고 느꼈습니다.
벡터데이터 베이스로는 법률 도메인의 특성에 맞는 응답을 할 수 없다고 생각이 됩니다.
왜냐하면 유사도 기반의 방식으로는 법률 도메인 특유의 추론을 할 수 없기 때문입니다.
따라서, 의미간에 관계가 정의된다면 더욱 더 좋은 응답을 할 수 있을 것이라고 생각합니다.
그 기법에는 Knowledge Graph를 이용한 RAG 기법이 대표적이라고 생각하고 최근에 네이버에서 발표한 아래와 같은 GraphReady라는 기술과 같은 파이프라인을 적용시킨다면 더 나을 것이라는 생각이 듭니다.

https://clova.ai/tech-blog/%EB%B3%B5%EC%9E%A1%ED%95%9C-%EC%A0%95%EB%B3%B4%EC%9D%98-%EC%88%B2%EC%97%90%EC%84%9C-%EA%B8%B8%EC%9D%84-%EC%B0%BE%EB%8B%A4-%EC%A7%80%EC%8B%9D-%EB%82%B4%EB%B9%84%EA%B2%8C%EC%9D%B4%EC%85%98-graphready
