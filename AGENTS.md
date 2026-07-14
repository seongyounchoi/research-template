# AGENTS.md

## 1. 프로젝트 성격과 우선순위

이 저장소는 AI/ML 연구, NLP/LLM/Agent 실험, 논문 구현 및 재현, 머신러닝 대회를 위한 프로젝트다.

다음 우선순위를 따른다.

1. 정확성
2. 실험 재현성
3. 명확하고 이해하기 쉬운 코드
4. 빠른 실험 반복
5. 실험 수정과 비교의 용이성

연구 코드를 과도하게 엔지니어링하지 않는다. 명시적으로 요청하지 않는 한 복잡한 클래스 계층, 불필요한 추상화, 디자인 패턴, 서비스 지향 구조를 만들지 않는다.

---

## 2. 작업 시작 전 확인

코드를 수정하기 전에 다음을 먼저 확인한다.

1. 현재 저장소와 디렉터리 구조
2. 관련 기존 코드와 README
3. Git 상태와 사용자의 미커밋 변경 사항
4. 현재 OS와 Python 환경
5. Conda 설치 여부와 기존 환경 목록
6. 설치된 핵심 패키지와 버전
7. 기존 config 및 실험 관리 방식
8. GPU가 필요한 작업이라면 현재 GPU/드라이버 상태

기존 구현을 이해하기 전에 새로운 구조를 만들지 않는다.

요청하지 않은 대규모 리팩토링은 하지 않는다. 기존 사용자 변경 사항을 임의로 reset, discard, overwrite하지 않는다.

---

## 3. Python 및 Conda 환경

이 프로젝트는 Conda가 설치된 Linux 연구실 GPU 서버에서 실행하는 것을 기본 환경으로 한다.

Python 환경은 프로젝트별 Conda 환경을 사용한다.

- Conda `base` 환경에 프로젝트 패키지를 설치하지 않는다.
- 시스템 Python에 패키지를 전역 설치하지 않는다.
- `.venv` 또는 Python `venv`를 생성하지 않는다.
- Anaconda, Miniconda 또는 Conda를 새로 설치하지 않는다.
- 서버에 이미 설치된 Conda를 사용한다.
- 현재 프로젝트와 무관한 기존 Conda 환경을 임의로 변경하지 않는다.

새 프로젝트에서 Python 실행이 필요한 경우:

1. 현재 Conda 환경 목록을 확인한다.
2. 현재 프로젝트에 적합한 기존 환경이 있는지 확인한다.
3. 프로젝트 전용 환경이 없다면 새 Conda 환경을 생성한다.
4. 프로젝트 목적과 주요 라이브러리 호환성을 고려해 Python 버전을 선택한다.
5. 생성한 프로젝트 환경에서 모든 Python 실행과 패키지 설치를 수행한다.

환경 이름은 프로젝트 이름을 기반으로 짧고 명확하게 정한다.

예:

```bash
conda create -n agent-research python=3.11
conda activate agent-research

## 4. NVIDIA GPU 및 CUDA

GPU 작업 전 실제 시스템 환경을 확인한다.

가능하면 다음을 확인한다.

- `nvidia-smi`
- GPU 모델과 개수
- GPU별 메모리 사용량
- GPU별 utilization
- NVIDIA driver version
- driver가 지원하는 CUDA version
- PyTorch version
- `torch.version.cuda`
- `torch.cuda.is_available()`
- `torch.cuda.device_count()`
- 감지된 GPU 이름

일반적인 PyTorch 사용을 위해 시스템 CUDA Toolkit 전체가 반드시 필요하다고 가정하지 않는다.

다음을 임의로 변경하지 않는다.

- NVIDIA driver
- 시스템 CUDA 설치
- 시스템 cuDNN
- 서버 전역 GPU 설정

custom CUDA extension 컴파일 등 명확한 필요성이 확인되지 않는 한 CUDA Toolkit을 설치하거나 변경하지 않는다.

PyTorch를 설치할 때는 현재 NVIDIA driver 및 프로젝트 요구사항과 호환되는 공식 CUDA 지원 빌드를 사용한다.

정상적인 CUDA 지원 PyTorch를 CPU-only 빌드로 임의 교체하지 않는다.

GPU 설정 후 가능하면 실제 CUDA tensor 연산으로 동작을 검증한다.

---

## 5. 공용 멀티 GPU 서버

Linux 연구실 GPU 서버 또는 공용 멀티 GPU 서버에서는 서버 환경과 연구실 규칙을 최우선으로 존중한다.

GPU 작업 전에 현재 사용 현황을 확인한다.

예:

```bash
nvidia-smi
```

또는 설치되어 있다면:

```bash
nvtop
```

다음 원칙을 따른다.

- GPU 0이 비어 있다고 가정하지 않는다.
- 다른 사용자의 GPU 프로세스를 종료하거나 방해하지 않는다.
- 빈 GPU 또는 사용자에게 할당된 GPU를 사용한다.
- 연구실의 GPU 배정 규칙이나 스케줄러가 있다면 이를 우선한다.
- 단일 GPU 실험에 여러 GPU를 점유하지 않는다.
- 멀티 GPU 실행은 실험상 필요한 경우에만 사용한다.
- 서버의 시스템 환경을 관리자 권한으로 임의 변경하지 않는다.

직접 GPU를 선택하는 서버라면 `CUDA_VISIBLE_DEVICES` 사용을 우선 고려한다.

예:

```bash
CUDA_VISIBLE_DEVICES=2 python scripts/train.py
```

이 경우 프로그램 내부에서 보이는 `cuda:0`은 물리 GPU 2번일 수 있다는 점을 고려한다.

코드에서 device 선택은 명시적으로 처리한다.

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

모델과 tensor가 동일한 device에 위치하는지 확인한다.

---

## 6. 장시간 서버 작업

SSH 연결이 끊어져도 계속 실행되어야 하는 장시간 학습은 서버의 기존 운영 방식을 확인한다.

Slurm 또는 다른 job scheduler가 있으면 해당 스케줄러를 우선 사용한다.

별도 스케줄러가 없고 연구실에서 허용한다면 `tmux`를 사용할 수 있다.

예:

```bash
tmux new -s train
```

장시간 학습을 단순 VS Code SSH 터미널 세션에만 의존하지 않는다.

학습 실행 시 사용 환경, GPU, config, output 경로를 확인한다.

---

## 7. 연구 코드 작성 원칙

단순하고 명시적인 구현을 우선한다.

중요한 실험 로직은 코드나 config에서 쉽게 확인할 수 있어야 한다.

다음과 같은 핵심 설정을 숨기지 않는다.

- model name
- learning rate
- batch size
- epochs
- optimizer
- scheduler
- random seed
- train/validation split
- maximum sequence length
- loss function
- evaluation metric
- checkpoint selection criterion

설명 없는 magic number를 피한다.

중요한 연구 로직에는 이유를 이해하는 데 도움이 되는 주석을 작성한다. 코드를 그대로 반복 설명하는 주석은 만들지 않는다.

---

## 8. 재현성

무작위성이 있는 실험은 random seed를 관리한다.

필요에 따라 다음 seed를 설정한다.

- Python `random`
- NumPy
- PyTorch CPU
- PyTorch CUDA

예:

```python
def set_seed(seed: int = 42):
    import random
    import numpy as np
    import torch

    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)

    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
```

deterministic 설정이 학습 속도 또는 동작에 의미 있는 영향을 주면 trade-off를 설명한다.

주요 실험 설정과 결과를 기록한다.

---

## 9. 데이터 누수

학습 및 validation 설계를 확정하기 전에 데이터 누수 가능성을 확인한다.

특히 다음을 점검한다.

- target 정보 노출
- 미래 정보 사용
- timestamp와 시간 순서
- sequence ordering
- duplicated samples
- entity overlap
- user overlap
- group overlap
- 전체 데이터로 학습된 preprocessing
- validation/test 통계를 사용한 normalization
- 미래 관측값을 사용한 feature engineering

시계열 또는 순서 예측 문제에서는 시간 구조를 확인하기 전에 random split을 사용하지 않는다.

그룹 구조가 있으면 group-aware split을 고려한다.

전처리기는 필요한 경우 train data에만 fit한다.

의심스러운 leakage 가능성을 발견하면 명시적으로 보고한다. 조용히 무시하고 실험을 진행하지 않는다.

---

## 10. 학습, 검증, 추론

가능하면 다음 책임을 명확하게 구분한다.

- training
- validation
- inference

validation metric은 실제 연구 목적 또는 대회 evaluation metric과 최대한 일치시킨다.

실제 평가 지표와 다른 편리한 metric을 최적화한다면 그 차이를 설명한다.

가능하면 다음을 기록한다.

- training loss
- validation loss
- primary validation metric
- learning rate
- epoch

best checkpoint 기준과 metric의 방향성(higher/lower is better)을 명확히 한다.

training set 성능만으로 모델 성능을 판단하지 않는다.

---

## 11. 실험 관리

여러 실험을 비교할 수 있도록 최소한 다음 정보를 관리한다.

- experiment name 또는 ID
- model
- 주요 hyperparameter
- seed
- validation score
- 필요한 notes

간단한 CSV, JSON, YAML 형식을 우선 고려한다.

기존 프로젝트가 MLflow, Weights & Biases, TensorBoard 등을 사용한다면 기존 방식을 따른다.

사용자가 요청하지 않는 한 무거운 experiment tracking platform을 새로 도입하지 않는다.

이전 실험 결과를 임의로 덮어쓰지 않는다.

---

## 12. 프로젝트 구조

기존 저장소 구조를 불필요하게 변경하지 않는다.

새로운 중소 규모 연구 프로젝트에서는 필요에 따라 다음 구조를 선호한다.

```text
project/
├── AGENTS.md
├── README.md
├── requirements.txt
├── .gitignore
├── configs/
├── data/
├── notebooks/
├── src/
├── scripts/
└── outputs/
```

일반적인 역할:

- `configs/`: 실험 설정
- `data/`: 로컬 데이터셋
- `notebooks/`: EDA, 분석, 빠른 실험
- `src/`: 재사용할 핵심 구현
- `scripts/`: train, evaluate, inference 실행 진입점
- `outputs/`: 실험 결과와 checkpoint

작은 실험은 더 단순한 구조를 사용한다.

즉시 사용할 목적이 없는 디렉터리를 불필요하게 만들지 않는다.

---

## 13. Jupyter Notebook

Notebook은 다음 용도에 적합하다.

- EDA
- 데이터 확인
- 시각화
- 빠른 아이디어 검증
- 모델 동작 디버깅

반복 실행할 장기 학습 pipeline 전체를 notebook에만 두지 않는다.

재사용할 로직은 적절한 시점에 Python module로 분리한다.

Notebook의 hidden state와 셀 실행 순서 의존성을 주의한다.

---

## 14. 의존성 관리

프로젝트 의존성을 관리한다.

기존 프로젝트의 표준이 없다면 `requirements.txt`를 사용할 수 있다.

Conda 환경 재현이 중요한 프로젝트라면 `environment.yml` 사용을 고려한다.

필요한 패키지만 추가한다.

관리된 dependency 파일에 무관한 패키지가 대량 추가될 수 있다면 `pip freeze > requirements.txt`를 무조건 실행하지 않는다.

재현성에 중요한 패키지는 버전 차이의 영향을 고려한다.

특히 다음 패키지의 버전을 주의한다.

- torch
- torchvision
- transformers
- accelerate
- datasets
- numpy
- pandas
- scikit-learn
- xgboost
- lightgbm
- deepspeed
- bitsandbytes
- flash-attn

기존 프로젝트의 주요 dependency를 이유 없이 업그레이드하지 않는다.

---

## 15. Git 및 대용량 파일

`.gitignore`를 확인하고 로컬 또는 생성 파일을 적절히 제외한다.

일반적으로 다음 항목을 검토한다.

```gitignore
.venv/
__pycache__/
*.pyc
.env

data/
outputs/
checkpoints/
wandb/

*.pt
*.pth
*.ckpt
```

저장소가 특정 파일을 의도적으로 추적한다면 기존 정책을 존중한다.

다음을 Git에 커밋하지 않는다.

- API key
- access token
- password
- `.env`
- private credential
- 대용량 dataset
- 불필요한 model checkpoint

광범위한 Git 작업 전에 `git status`를 확인한다.

사용자의 미커밋 변경을 명시적 허가 없이 삭제하지 않는다.

force push는 명시적으로 요청받지 않는 한 하지 않는다.

---

## 16. 비밀정보와 인증정보

비밀정보를 코드에 hardcode하지 않는다.

예:

- OpenAI API key
- Hugging Face token
- Dacon credential
- WandB API key
- cloud credential

필요하면 환경변수를 사용한다.

```python
import os

api_key = os.getenv("OPENAI_API_KEY")
```

`.env`를 사용하면 Git에서 제외한다.

로그나 터미널 출력에 전체 secret을 노출하지 않는다.

---

## 17. 코드 수정 원칙

기능을 구현할 때 다음 순서를 따른다.

1. 최소 관련 파일 식별
2. 기존 code path 이해
3. 가장 작은 합리적 변경 수행
4. 요청하지 않은 기존 동작 보존
5. 변경 경로 검증
6. 변경 내용 보고

기능 작업 중 관련 없는 코드 정리를 함께 하지 않는다.

필요하지 않다면 저장소 전체의 파일명, 함수명, 클래스명을 광범위하게 변경하지 않는다.

대규모 리팩토링이 필요하다고 판단되면 이유를 설명한다.

---

## 18. 테스트 및 검증

코드를 작성한 것만으로 작업 완료라고 판단하지 않는다.

가능한 범위에서 다음을 확인한다.

1. syntax
2. import
3. 기존 test
4. 관련 기능 실행
5. smoke test

머신러닝 학습 코드는 바로 장시간 full training을 실행하지 않는다.

먼저 작은 데이터 또는 짧은 실행으로 다음을 확인한다.

- data loading
- tensor shape
- forward pass
- loss calculation
- backward pass
- optimizer step
- validation
- checkpoint logic

GPU 코드라면 가능할 때 실제 CUDA 연산을 검증한다.

실행하지 않은 코드를 작동 확인했다고 말하지 않는다.

다음 상태를 명확하게 구분한다.

- implemented
- syntax checked
- import checked
- smoke tested
- fully trained
- fully evaluated

---

## 19. 오류 해결

오류 발생 시 다음 순서를 따른다.

1. 전체 error message와 traceback 확인
2. root cause 추정
3. 관련 코드와 실제 환경 확인
4. targeted fix 수행
5. 실패한 명령 재실행
6. 원래 오류 해결 여부 확인

결과를 확인하지 않고 추측성 수정만 반복하지 않는다.

환경 문제라면 실제 Python 및 package version을 확인한 후 변경한다.

광범위한 `except Exception: pass`로 오류를 숨기지 않는다.

---

## 20. 논문 구현 및 재현

논문 구현 시 다음을 확인한다.

1. 정확한 논문과 version
2. official code 존재 여부
3. 관련 methodology section
4. 핵심 contribution
5. 논문 명시 사항과 구현상 가정의 구분
6. reproduction 목적이라면 원 평가 설정

논문에 명시되지 않은 내용을 임의로 논문의 방법이라고 표현하지 않는다.

가정과 모호한 부분을 명확히 표시한다.

실제 구현과 실험 설정이 일치하지 않으면 정확한 reproduction이라고 주장하지 않는다.

가능하면 다음 비교를 구분한다.

- baseline
- paper method
- proposed modification

---

## 21. 머신러닝 대회

대회 프로젝트에서는 다음 순서를 우선한다.

1. evaluation metric 이해
2. train/test schema 확인
3. target distribution 확인
4. missing value 확인
5. duplicate 확인
6. sequence, temporal, entity, group structure 확인
7. leakage 가능성 확인
8. 단순 baseline 구축
9. 신뢰할 수 있는 local validation 설계
10. 실험 비교

복잡한 deep learning model이 단순 baseline보다 항상 우수하다고 가정하지 않는다.

문제에 따라 다음 baseline을 고려한다.

- heuristic
- statistical model
- tree-based model
- simple neural model

public leaderboard score만으로 일반화 성능을 판단하지 않는다.

leaderboard overfitting 가능성을 고려한다.

---

## 22. 성능 최적화

정확성을 먼저 확보하고 이후 성능을 최적화한다.

GPU workload의 병목은 실제 근거를 바탕으로 확인한다.

필요에 따라 다음을 검토한다.

- batch size
- DataLoader
- `num_workers`
- pinned memory
- mixed precision
- gradient accumulation
- CPU-GPU transfer
- sequence padding
- model size
- GPU memory usage

mixed precision이 적절하면 지원되는 PyTorch AMP API를 사용한다.

모델 동작을 변경할 수 있는 최적화는 영향을 설명한다.

---

## 23. 작업 완료 보고

작업 완료 시 간결하고 구체적으로 다음을 정리한다.

### Changed Files
생성하거나 수정한 파일

### Key Changes
핵심 구현 및 변경 내용

### How to Run
실제 실행 명령

서버 GPU 실행이라면 필요 시 예를 들어 다음과 같이 제시한다.

```bash
CUDA_VISIBLE_DEVICES=2 python scripts/train.py --config configs/baseline.yaml
```

### Verification
실제로 수행한 검증과 결과

### Notes
중요한 가정, 위험, 데이터 누수 우려, 미완료 작업

full training을 수행하지 않았다면 명확히 말한다.

확인되지 않은 결과를 확정적으로 표현하지 않는다.

---

## 24. 사용자 설명 수준

사용자는 AI/NLP 연구와 실험을 수행하고 있다.

중요한 기술적 결정을 내릴 때는 이유를 짧게 설명한다.

예:

- validation split 선택 이유
- 모델 선택 이유
- Python/package version 선택 이유
- CUDA/PyTorch build 선택 이유
- leakage 가능성
- baseline 선택 이유

그러나 모든 코드 변경을 기초 강의처럼 장황하게 설명하지 않는다.

요청한 작업 완료를 우선하면서 중요한 연구 판단을 이해할 수 있게 한다.

여러 선택이 모두 합리적이라면 가장 단순하고 강한 baseline을 우선한다.
