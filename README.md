다음은 위 내용을 다듬어 정리한 README.md 예시입니다:

---

# KMMLU 형법 에이전트 시스템

본 프로젝트는 KMMLU 형법 데이터셋을 활용하여, LLM 기반의 법률 질의응답 및 평가 시스템을 구현한 연구용 에이전트입니다. 이 시스템은 검색 기반 보강과 대규모 언어 모델(OpenAI API 등)을 결합해, 법률 문제(특히 형법 분야)의 해답을 생성하고 평가하는 데 목적을 두고 있습니다.

## 1. 프로젝트 개요

### 1.1 시스템 목적
- **LLM 기반 질의응답**: OpenAI의 GPT 계열 모델을 활용하여 형법 관련 질문에 대해 답변을 생성합니다.
- **정보 검색 통합**: FAISS 벡터 검색과 BM25 키워드 검색을 조합해, 질문과 관련된 법령 조문, 판례, 해설 등 문서를 검색하고, 이를 LLM 프롬프트의 추가 정보로 제공합니다.
- **즉시 질의응답 및 배치 평가**: 단일 질문에 대해 즉각적인 응답을 생성하는 모드와, 다수의 질문을 일괄 평가하는 모드를 모두 지원합니다.
- **결과 저장 및 분석**: 생성된 응답과 평가 결과를 CSV/JSON 형식으로 저장하여, 후속 분석이나 오답 사례 검토에 활용할 수 있습니다.

### 1.2 주요 기능
- **실시간 질의응답 (Immediate Mode)**  
  사용자가 입력한 단일 질문에 대해 관련 법률 문서를 검색하고, LLM을 통해 답변을 생성합니다.
- **배치 평가 (Batch Mode)**  
  CSV 또는 JSONL 형식의 질문 목록에 대해 일괄적으로 답변을 생성한 후, 정답과 비교하여 정확도를 평가합니다.
- **검색 엔진 통합**  
  - **FAISS**: 문서 임베딩 기반의 유사도 검색을 통해 질문과 의미적으로 가까운 문서를 찾습니다.
  - **BM25**: 키워드 기반 검색으로 후보 문서를 선별하여, 검색의 재현율을 높입니다.
- **컨테이너화**  
  Docker 및 Docker Compose를 활용하여 일관된 실행 환경에서 시스템을 구동할 수 있습니다.

## 2. 설치 방법

### 2.1 리포지토리 클론
Git을 이용하여 프로젝트를 클론합니다.
```bash
git clone https://github.com/yourusername/kmmlu-criminal-law-agent.git
cd kmmlu-criminal-law-agent
```

### 2.2 Poetry를 통한 의존성 설치
[Poetry](https://python-poetry.org/)를 사용하여 필요한 패키지를 설치합니다.
```bash
poetry install
```
Poetry는 `pyproject.toml`과 `poetry.lock` 파일에 명시된 의존성을 읽어 자동으로 가상환경을 구성합니다.

### 2.3 환경 변수 설정
OpenAI API를 사용하기 위해 OpenAI API 키를 환경 변수로 설정합니다.
```bash
export OPENAI_API_KEY="sk-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```
Windows PowerShell에서는 다음과 같이 설정합니다:
```powershell
$Env:OPENAI_API_KEY="sk-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```
또는 `.env` 파일에 `OPENAI_API_KEY=<YOUR_KEY>` 형태로 저장하여 사용할 수도 있습니다.

### 2.4 추가 요구 사항
- Python 3.9 이상
- FAISS 라이브러리는 `faiss-cpu` 또는 필요에 따라 `faiss-gpu`로 설치할 수 있습니다.
- OpenAI API 사용 시 인터넷 연결 및 요금 발생에 유의하세요.

## 3. 컨테이너 셋업

Docker를 이용하여 프로젝트 환경을 컨테이너화하면, 개발 환경에 상관없이 일관된 방식으로 실행할 수 있습니다.

### 3.1 Docker 이미지 빌드
프로젝트 디렉토리 내 Dockerfile을 기반으로 이미지를 빌드합니다.
```bash
docker build -t kmmlu-agent .
```

### 3.2 Docker 컨테이너 실행
즉시 모드로 실행하려면 다음과 같이 실행합니다:
```bash
docker run -it --rm -e OPENAI_API_KEY=$OPENAI_API_KEY kmmlu-agent \
    python main.py immediate --question "형법상 정당방위의 요건은 무엇인가?"
```
옵션 설명:
- `-it`: 인터랙티브 모드
- `--rm`: 종료 후 컨테이너 자동 삭제
- `-e`: 환경 변수 전달

### 3.3 Docker Compose 이용 실행
프로젝트에는 `docker-compose.yml` 파일이 포함되어 있어, 다음 명령으로 서비스를 빌드 및 실행할 수 있습니다:
```bash
docker-compose up --build
```
Compose 파일 내에서는 빌드, 볼륨, 포트 매핑, 실행 명령 등이 미리 설정되어 있으므로, 단일 명령으로 모든 서비스를 시작할 수 있습니다.

## 4. 에이전트 시스템 실행

### 4.1 즉시 생성 모드 (Immediate Mode)
단일 질문에 대해 즉시 답변을 생성합니다.
```bash
poetry run python main.py immediate --question "형법상 미수범 처벌에 대한 설명은 무엇인가?"
```
- `--question`: LLM에 전달할 질문 텍스트를 지정합니다.
- 실행 시, 관련 문서 검색 결과와 함께 LLM이 답변을 생성하며, 콘솔에 결과가 출력됩니다.

### 4.2 배치 평가 모드 (Batch Mode)
CSV 파일 등에서 질문을 불러와 일괄 평가합니다.
```bash
poetry run python main.py batch --input batchinput.jsonl --output output.jsonl
```
- `--input`: 질문 및 정답이 포함된 입력 파일(JSONL 형식)
- `--output`: 모델의 응답 결과를 저장할 파일 경로

배치 모드에서는 각 질문에 대해 관련 정보를 검색한 후, LLM을 호출하여 답변을 생성합니다. 생성된 결과는 평가 스크립트(`evaluation.py`)를 통해 정답과 비교되어 정확도가 산출됩니다.

## 5. 평가 수행

평가 스크립트 `evaluation.py`를 통해 모델의 응답 결과를 정답과 비교하여 정확도 등 성능 지표를 계산할 수 있습니다.

예시:
```bash
poetry run python evaluation.py --pred output.jsonl --gold batchinput.jsonl --metric results.csv
```
- `--pred`: 모델 예측 결과 파일 (JSONL)
- `--gold`: 정답 파일 (JSONL)
- `--metric`: 평가 결과를 저장할 파일 경로 (CSV 등)

평가 결과는 각 질문에 대해 모델의 정답 여부와 전체 정확도 등을 확인할 수 있도록 저장됩니다.

## 6. 파일 구성 및 역할

- **agent.py**: LLM 기반 응답 생성 및 검색(FAISS, BM25) 통합 로직 구현  
  - 법령 조문, 판례 등 법률 문서를 로드 및 검색하여 LLM 프롬프트에 포함
  - OpenAI API 호출 및 답변 후처리(정답 추출)를 담당

- **data_loader.py**: 데이터셋 및 문서 로딩/전처리 모듈  
  - CSV 파일로부터 KMMLU 형법 데이터셋 로드
  - 문서 클렌징 및 필요한 전처리 수행

- **main.py**: 전체 파이프라인 실행 스크립트  
  - CLI 인자를 파싱하여 즉시 모드(immediate) 또는 배치 모드(batch)를 선택
  - 각 모드에 따라 에이전트 실행 및 결과 저장

- **evaluation.py**: 배치 평가 및 성능 측정을 위한 스크립트  
  - 입력 파일과 모델 응답 파일을 비교하여 정확도 계산 및 CSV 등으로 결과 저장

- **pyproject.toml** 및 **poetry.lock**: Poetry 패키지 관리 파일  
  - 프로젝트 의존성 및 빌드 설정 포함

- **Dockerfile** 및 **docker-compose.yml**: 컨테이너 환경 구성 파일  
  - Docker 이미지를 빌드하고, 컨테이너 실행 및 환경 변수를 설정하는 데 사용

## 7. 추가 참고 사항

### 7.1 데이터셋 형식 및 요구사항
- KMMLU 형법 데이터셋은 객관식 문제와 정답으로 구성되어 있으며, JSONL 혹은 CSV 형식으로 제공됩니다.
- 데이터 파일 내 각 질문은 고유 ID, 질문 텍스트, 선택지, 정답 등이 포함되어야 합니다.
- 데이터 전처리 시 정답 포맷(숫자 또는 문자)을 통일하여 evaluation.py가 정확하게 채점할 수 있도록 해야 합니다.

### 7.2 성능 최적화 방안
- **병렬 처리**: 배치 모드에서 다수의 질문을 병렬로 처리하여 실행 시간을 단축합니다.
- **캐싱**: 동일한 질문에 대한 응답을 반복 요청하지 않도록 결과 캐싱을 활용합니다.
- **임베딩 사전연산**: FAISS 인덱스는 최초 계산 후 저장하여 이후 로드 속도를 개선합니다.
- **토큰 및 프롬프트 관리**: LLM에 전달하는 문맥 정보의 양을 적절히 제한하여 응답 지연을 줄입니다.

### 7.3 향후 개선 방향
- **Chain-of-Thought**: 모델이 단계별로 사고 과정을 기록하도록 프롬프트를 개선해, 보다 정확한 답변 유도를 시도할 수 있습니다.
- **피드백 루프**: 틀린 사례에 대한 오답 분석 및 추가 학습을 통해 시스템 성능을 지속적으로 개선할 수 있습니다.
- **웹 인터페이스**: CLI뿐 아니라, 웹 기반 UI를 도입하여 사용자 친화적인 인터랙티브 질의응답 시스템으로 발전시킬 수 있습니다.
- **다른 법률 분야 확장**: 형법 외의 헌법, 민법 등 다른 법률 분야 데이터셋을 포함하여 시스템 범위를 확대할 수 있습니다.

---

이 README 파일은 시스템의 전체 개요, 설치 및 실행 방법, 파일 구조와 각 모듈의 역할, 그리고 향후 개선 방안까지 상세하게 다루고 있습니다. 각 단계별로 명령어와 설정 방법을 제공하므로, 프로젝트 초기 설정부터 컨테이너 실행, 평가 수행까지 원활하게 진행할 수 있습니다.
