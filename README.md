# AI/ML 연구 프로젝트 템플릿

이 저장소는 AI/ML 연구, NLP/LLM/Agent 실험, 논문 구현 및 재현, 머신러닝 대회 프로젝트를 빠르게 시작하기 위한 GitHub Template Repository입니다.

이 템플릿은 연구 코드에 필요한 최소한의 구조만 제공합니다. 처음부터 복잡한 구조를 만들기보다 단순한 baseline에서 시작하고, 실험에 실제로 필요한 경우에만 구조와 기능을 확장하는 것을 원칙으로 합니다.

## 사용 환경

이 템플릿은 기본적으로 다음 환경에서 사용하는 것을 전제로 합니다.

- Linux 연구실 GPU 서버
- 서버에 기존 Conda 설치
- 프로젝트별 Conda 환경 사용
- NVIDIA GPU 사용
- VS Code Remote SSH를 통한 원격 개발
- GitHub를 통한 코드 버전 관리

서버의 시스템 Python, Conda `base` 환경, NVIDIA Driver, 시스템 CUDA 환경은 임의로 변경하지 않습니다.

## 사용 방법

### 1. 이 템플릿으로 새 GitHub Repository 생성

GitHub에서 이 Template Repository의 `Use this template` 버튼을 사용해 새 Repository를 생성합니다.

예:

```text
research-template
        ↓
Use this template
        ↓
agent-research
```

### 2. 연구실 서버에 접속

VS Code Remote SSH 등을 사용해 연구실 Linux GPU 서버에 접속합니다.

### 3. 서버에서 새 Repository clone

프로젝트를 보관할 디렉터리로 이동한 뒤 새 Repository를 clone합니다.

```bash
cd ~/projects
git clone <repository-url>
cd <project-name>
```

프로젝트 디렉터리를 VS Code에서 엽니다.

### 4. Codex에게 초기 환경 설정 요청

프로젝트 루트의 `AGENTS.md` 지침을 기준으로 환경을 설정합니다.

예시 요청:

```text
이 프로젝트는 Agent AI 연구 프로젝트야.

AGENTS.md 지침을 확인하고 현재 서버와 프로젝트 환경을 점검한 뒤
Python 개발 환경을 세팅해줘.

환경 세팅 후 확인한 환경, 생성한 Conda 환경, 설치한 패키지,
수정한 파일, GPU 사용 가능 여부를 정리해줘.
```

Codex는 프로젝트와 서버 환경을 먼저 확인하고, 필요한 경우 프로젝트별 Conda 환경을 생성합니다.

Conda 자체를 새로 설치하거나 `base` 환경에 프로젝트 패키지를 설치하지 않습니다.

## Conda 환경

각 프로젝트는 별도의 Conda 환경을 사용하는 것을 기본 원칙으로 합니다.

예:

```bash
conda create -n agent-research python=3.11
conda activate agent-research
```

프로젝트에 필요한 Python 패키지는 활성화된 프로젝트 Conda 환경 안에 설치합니다.

예:

```bash
pip install torch transformers accelerate
```

실제 Python 버전과 패키지 버전은 프로젝트 목적 및 라이브러리 호환성을 확인한 뒤 결정합니다.

## GPU 사용

GPU 작업 전 현재 GPU 사용 상태를 확인합니다.

```bash
nvidia-smi
```

또는 서버에 설치되어 있다면:

```bash
nvtop
```

공용 멀티 GPU 서버에서는 GPU 0이 비어 있다고 가정하지 않습니다.

연구실의 GPU 사용 규칙을 우선하며, 직접 GPU를 선택하는 환경에서는 필요에 따라 `CUDA_VISIBLE_DEVICES`를 사용합니다.

예:

```bash
CUDA_VISIBLE_DEVICES=2 python scripts/train.py
```

단일 GPU 실험에서는 불필요하게 여러 GPU를 점유하지 않습니다.

NVIDIA Driver, 시스템 CUDA, cuDNN 등 서버 전역 환경은 임의로 설치하거나 변경하지 않습니다.

## 기본 작업 흐름

일반적인 연구 프로젝트 흐름은 다음과 같습니다.

```text
GitHub Template로 새 Repository 생성
        ↓
연구실 서버에 SSH 접속
        ↓
서버에서 git clone
        ↓
프로젝트 폴더 열기
        ↓
Codex가 AGENTS.md 확인
        ↓
프로젝트별 Conda 환경 구성
        ↓
데이터 및 문제 분석
        ↓
단순 baseline 구현
        ↓
smoke test
        ↓
학습 및 평가
        ↓
실험 결과 기록
        ↓
Git commit / push
```

연구 및 대회 프로젝트에서는 모델을 만들기 전에 데이터 구조, 평가 지표, validation 전략, 데이터 누수 가능성을 먼저 확인하는 것을 권장합니다.

## 디렉터리 구조

```text
.
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

각 디렉터리의 기본 역할은 다음과 같습니다.

| 디렉터리 | 역할 |
| --- | --- |
| `configs/` | 모델 및 실험 설정 |
| `data/` | 로컬 데이터셋 |
| `notebooks/` | EDA, 데이터 분석, 빠른 아이디어 실험 |
| `src/` | 재사용할 모델, 데이터 처리, loss 등 핵심 구현 |
| `scripts/` | 학습, 평가, 추론 실행 코드 |
| `outputs/` | 실험 결과, 로그, checkpoint |

작은 프로젝트에서는 모든 디렉터리를 억지로 사용할 필요가 없습니다. 실제 필요에 따라 구조를 확장합니다.

## 권장 연구 방식

- 데이터 구조와 평가 지표를 먼저 확인합니다.
- validation 전략을 설계하기 전에 데이터 누수 가능성을 확인합니다.
- 단순하고 강한 baseline부터 시작합니다.
- 중요한 실험 설정은 코드 또는 config에서 명확하게 확인할 수 있도록 관리합니다.
- random seed, 모델 이름, 주요 hyperparameter, validation metric을 기록합니다.
- 장시간 학습 전에 작은 데이터 또는 짧은 실행으로 smoke test를 수행합니다.
- public leaderboard 점수만으로 모델의 일반화 성능을 판단하지 않습니다.
- 논문 구현 시 논문에 명시된 내용과 구현상 가정을 구분합니다.

## Git 및 파일 관리

일반적으로 다음 파일은 GitHub에 업로드하지 않습니다.

- 로컬 데이터셋
- 실험 결과
- 모델 checkpoint
- API key 및 access token
- `.env`
- 기타 private credential

코드를 수정한 뒤에는 변경 내용을 확인하고 다음 흐름으로 GitHub에 반영합니다.

```text
변경 내용 확인
        ↓
스테이징
        ↓
커밋 메시지 작성
        ↓
Commit
        ↓
Push / Sync Changes
```

사용자의 미커밋 변경 사항을 임의로 삭제하거나 덮어쓰지 않습니다.

## 장시간 학습

SSH 연결 종료 후에도 계속 실행되어야 하는 장시간 학습은 연구실 서버의 운영 방식을 따릅니다.

Slurm 또는 다른 job scheduler가 있다면 해당 방식을 우선합니다.

별도 스케줄러가 없고 연구실에서 허용한다면 `tmux`를 사용할 수 있습니다.

예:

```bash
tmux new -s train
```

장시간 학습을 단순 VS Code SSH 터미널 세션에만 의존하지 않는 것을 권장합니다.

## AGENTS.md

`AGENTS.md`는 Codex가 이 Repository에서 작업할 때 따를 공통 연구 및 개발 지침입니다.

주요 내용은 다음과 같습니다.

- 프로젝트별 Conda 환경 사용
- Conda `base` 및 시스템 Python 보호
- 서버의 NVIDIA Driver와 CUDA 환경 보호
- 공용 GPU 사용 상태 확인
- 재현성 및 random seed 관리
- 데이터 누수 점검
- 단순한 baseline 우선
- 실험 설정 및 결과 관리
- smoke test 수행
- 기존 Git 변경 사항 보호
- 논문 내용과 구현상 가정 구분

프로젝트별로 특별한 규칙이 필요하면 해당 프로젝트의 `AGENTS.md`를 수정해 추가합니다.

## 참고 사항

- 이 템플릿은 PyTorch 또는 다른 Python 패키지를 기본 설치하지 않습니다.
- 프로젝트별 Conda 환경은 새 프로젝트를 시작한 뒤 구성합니다.
- Python 및 주요 패키지 버전은 프로젝트 요구사항과 호환성을 확인한 뒤 결정합니다.
- 예제 모델이나 학습 코드는 포함하지 않습니다.
- 각 연구 프로젝트의 문제, 데이터, 평가 방법에 맞게 필요한 코드만 추가합니다.
