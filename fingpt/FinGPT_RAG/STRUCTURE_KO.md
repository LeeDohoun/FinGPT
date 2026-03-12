# FinGPT_RAG 구조 가이드 (한국어)

이 문서는 `fingpt/FinGPT_RAG` 폴더를 기준으로,
논문 코드의 **구성 요소와 실행 흐름**을 빠르게 파악할 수 있도록 정리한 안내서입니다.

---

## 1) 이 프로젝트가 푸는 문제

FinGPT_RAG는 금융 감성분석에서 LLM이 놓치기 쉬운 맥락을,
외부 정보 검색(Retrieval)로 보강해 예측 품질을 높이는 것을 목표로 합니다.

핵심 아이디어는 다음 2단계입니다.
1. 질의와 관련된 정보를 검색해 문맥을 확장한다.
2. 확장된 문맥을 프롬프트에 넣어 LLM이 더 근거 있는 출력을 하게 한다.

---

## 2) 디렉토리 구조

- `README.md`
  - 프로젝트 개요, 성능, 모듈 설명, 실행 가이드.
- `requirements.txt`
  - 의존성 패키지 목록.
- `multisource_retrieval/`
  - 검색(리트리벌) 파이프라인 관련 코드.
- `instruct-FinGPT/`
  - 인스트럭션 튜닝(학습/추론) 프레임워크.
  - 내부에 `training/`, `inference/`, `training_scripts/` 등이 존재.

---

## 3) 코드 읽는 추천 순서

RAG를 처음 파악할 때는 아래 순서가 가장 빠릅니다.

1. `fingpt/FinGPT_RAG/README.md`
   - 전체 개념, 태스크, 실험 결과 확인.
2. `fingpt/FinGPT_RAG/instruct-FinGPT/README.md`
   - 학습/추론 프레임워크 범위 확인.
3. `fingpt/FinGPT_RAG/instruct-FinGPT/training/supervised_finetuning/main.py`
   - 학습 엔트리포인트(파라미터/학습 루프) 파악.
4. `fingpt/FinGPT_RAG/instruct-FinGPT/training/utils/`
   - 데이터 로딩, LoRA 모듈, 공통 유틸 확인.
5. `fingpt/FinGPT_RAG/instruct-FinGPT/training/supervised_finetuning/training_scripts/`
   - 실제 실행 쉘 스크립트 예시 참고.

---

## 4) 실행 관점 아키텍처

실행 흐름을 단순화하면 다음과 같습니다.

1. **데이터 준비**
   - 금융 문장/뉴스/트윗과 라벨 또는 지시문 데이터를 준비.
2. **검색 문맥 생성(RAG)**
   - 질의 관련 외부 텍스트를 검색해 컨텍스트를 붙임.
3. **SFT(Instruction Tuning)**
   - 컨텍스트 포함 입력으로 모델을 지도학습(LoRA 등).
4. **평가/추론**
   - 감성분석 성능(Accuracy/F1) 및 응답 품질 확인.

---

## 5) 파일 역할 빠른 맵

- `instruct-FinGPT/training/supervised_finetuning/main.py`
  - 학습 실행 진입점.
- `instruct-FinGPT/training/supervised_finetuning/main_data.py`
  - 데이터 관련 학습 실행 보조.
- `instruct-FinGPT/training/utils/data/data_utils.py`
  - 데이터셋 처리/전처리 유틸.
- `instruct-FinGPT/training/utils/module/lora.py`
  - LoRA 관련 모듈/설정.
- `instruct-FinGPT/training/utils/model/model_utils.py`
  - 모델 로딩/설정 공통 유틸.

---

## 6) 처음 실행할 때 체크리스트

1. `requirements.txt` 기반 의존성 설치.
2. 사용 모델 크기(예: 7B/13B)에 맞춰 GPU 메모리 확인.
3. `training_scripts`의 실행 예시를 복사해 경로/배치사이즈 수정.
4. 먼저 소규모 샘플로 end-to-end 동작 확인 후 전체 학습 수행.

---

## 7) 논문 읽기와 코드 연결 팁

- 논문에서 말하는 "retrieval augmentation" 부분은
  코드에서 검색 모듈 + 프롬프트 문맥 결합 로직으로 대응됩니다.
- 논문의 성능표는 데이터셋/프롬프트/모델 크기에 민감하므로,
  동일 조건(템플릿/평가셋) 재현이 중요합니다.
- 코드 이해 목표라면, 성능 재현 이전에
  "입력 1건이 어떻게 검색→프롬프트→출력으로 흐르는지"를 먼저 추적하세요.

