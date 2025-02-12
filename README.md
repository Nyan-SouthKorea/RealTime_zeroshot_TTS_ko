# 1. Introduction (소개)

**Realtime TTS System**은 [OpenVoice v2](https://github.com/myshell-ai/OpenVoice)를 기반으로, 실시간으로 음성을 합성하는 모듈화된 TTS 시스템입니다. 특히, 이 프로젝트는 한국어 음성 합성에 최적화되어 있어 한국어 사용자에게 뛰어난 경험을 제공합니다.

이 시스템은 제로샷(Zero-Shot) 학습을 지원하는 TTS 모델을 사용합니다. 제로샷 TTS는 별도의 음성 데이터 학습이나 훈련 없이도 새로운 화자의 음성 특성을 반영하여 음성을 생성할 수 있습니다. 먼저 기본 음성으로 TTS가 실행되고, 생성된 음성은 미리 준비된 샘플 음성과 같은 목소리로 실시간 변조됩니다. 이는 사용자 맞춤형 음성 합성과 자연스러운 대화형 애플리케이션에 이상적인 구조입니다.

### 주요 특징:
- **실시간 처리**: 음성 합성 및 변조가 실시간으로 이루어져 즉각적인 피드백을 제공합니다.
- **제로샷 학습**: 별도의 훈련 과정 없이도 다양한 음성 톤과 스타일을 반영할 수 있습니다.
- **한국어 최적화**: 한국어에 맞춘 발음 및 억양 처리를 통해 자연스러운 음성을 생성합니다.
- **모듈화된 설계**: 각 기능이 모듈화되어 있어 손쉽게 다른 시스템에 통합하거나 확장할 수 있습니다.

이 프로젝트는 특히 음성 인터페이스를 사용하는 로봇, 가상 비서, 또는 다양한 사용자 인터랙션 시스템에 적용하기에 적합합니다. 사용자는 별도의 복잡한 설정이나 학습 과정 없이 고품질의 음성을 사용할 수 있으며, 이를 통해 서비스의 사용성을 크게 향상시킬 수 있습니다.


# 2. Getting Started (환경 설정)
## [윈도우11, 우분투 22.04 동일 세팅 방법]

1. **새로운 환경 만들기**

   - **conda 업데이트**:
     ```bash
     conda update -n base conda
     conda update --all
     python -m pip install --upgrade pip
     ```

   - **환경 생성 후 실행**:
     ```bash
     conda create -n openvoice python=3.9
     conda activate openvoice
     ```

2. **CUDA 관련 설정** (PC에 따라 다를 수 있음)

   사용자 PC 환경: 윈도우11, RTX3060, 2024년 10월 기준 최신 드라이버 상태
   
   - CUDA 및 PyTorch 설치:
     ```bash
     conda install -c conda-forge cudatoolkit=11.8 cudnn=8.9
     conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
     ```

   - **CUDA 동작 확인**:
     아래 Python 스크립트를 실행하여 CUDA가 제대로 동작하는지 확인합니다.
     ```python
     python
     import torch
     torch.cuda.is_available()
     ```
     `True`가 출력되어야 합니다. 만약 `False`가 출력된다면, 다시 설정하거나 GPU 사용을 포기하고 CPU로 계속 진행할 수 있습니다.

3. **OpenVoice 관련 환경 설정**

   - **프로젝트 클론 및 의존성 설치**:
     ```bash
     git clone https://github.com/Nyan-SouthKorea/RealTime_zeroshot_TTS_ko.git
     cd RealTime_zeroshot_TTS_ko
     pip install -r requirements.txt
     ```
     원본 OpenVoice GitHub 링크: [OpenVoice](https://github.com/myshell-ai/OpenVoice.git)

4. **MELO TTS 설치**

   - **로컬 설치** (동작이 보장된 방법):
     ```bash
     cd MeloTTS
     pip install .
     python -m unidic download
     ```

   - **원본 GitHub 링크에서 설치** (위에서 로컬로 설치했다면 건너뛰기):
     ```bash
     pip install git+https://github.com/myshell-ai/MeloTTS.git
     python -m unidic download
     ```

5. **설치 후 최종 확인**

   - CUDA가 제대로 동작하는지 최종 확인:
     ```python
     python
     import torch
     torch.cuda.is_available()
     ```
     `True`가 출력되는지 확인합니다.



# 3-1. Usage (사용 방법) - Jupyter Notebook(demo_1(scratch).ipynb)

이 섹션에서는 `Custom_TTS` 모듈을 사용하여 텍스트를 음성으로 변환하고, 생성된 음성을 변조하여 사용자 맞춤형 음성을 생성하는 과정을 단계별로 설명합니다. 이 프로젝트는 한국어를 기반으로 한 TTS 시스템을 실시간으로 실행할 수 있으며, 제로샷 음성 변조 기능을 지원합니다.

## 1. Custom_TTS 클래스 불러오기

먼저 `custom_tts` 모듈에서 `Custom_TTS` 클래스를 불러옵니다.

```python
from custom_tts import Custom_TTS
```

#### 실행 결과 예시:
```
/home/ubuntu/anaconda3/envs/openvoice/lib/python3.9/site-packages/tqdm/auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
  from .autonotebook import tqdm as notebook_tqdm
```

이 경고 메시지는 Tqdm에서 발생하며, 무시할 수 있습니다.

## 2. Custom_TTS 인스턴스 생성

`Custom_TTS` 클래스의 인스턴스를 생성합니다. 이 과정에서 기본 모델 경로가 설정되며, CUDA 환경을 자동으로 감지하여 설정합니다.

```python
tts_module = Custom_TTS()
```

#### 실행 결과 예시:
```
본 코드를 개발한 Repo. 입니다: https://github.com/Nyan-SouthKorea/RealTime_zeroshot_TTS_ko
다음 Repo.를 참조하여 개발한 모듈입니다: https://github.com/myshell-ai/OpenVoice
사용 환경(cuda): cuda:0
```

## 3. TTS 및 음성 변조 모델 설정

TTS 모델과 음성 변조 모델을 로드합니다. 이 과정에서 사전 학습된 체크포인트가 로컬에 없으면 자동으로 다운로드하고 압축을 해제합니다. 모델이 로드된 후, 기본 화자의 음성 임베딩이 완료됩니다.

```python
tts_module.set_model()
```

#### 실행 결과 예시:
```
checkpoints_v2_0417.zip: 100%|██████████| 116M/116M [00:18<00:00, 6.66MB/s] 
checkpoints_v2_0417.zip 다운로드 완료!
Extracting: 100%|██████████| 125M/125M [00:00<00:00, 176MB/s]
톤 변경 모델 로드 완료
TTS 모델 로드 완료
기본 화자 음성 임베딩 완료
```

## 4. 참조할 목소리 샘플 입력

참조할 화자의 음성을 제공하여, 해당 음성 톤을 임베딩합니다. 이 과정에서 음성 파일을 입력받고, 음성 내의 목소리를 탐지하여 음성의 특징을 추출합니다.

```python
tts_module.get_reference_speaker(speaker_path='sample_iena.m4a')
```

#### 실행 결과 예시:
```
OpenVoice version: v2
목소리 톤 임베딩 완료
```

## 5. 텍스트를 음성으로 변환 및 변조

텍스트를 입력하여 TTS를 수행한 후, 음성 변조를 통해 참조한 화자의 목소리로 변조된 음성을 생성합니다. 결과 파일은 지정한 경로에 저장됩니다.

```python
tts_module.make_speech('오늘 회의시간에 너무 졸아서 고수석님께 혼났다')
```

#### 실행 결과 예시:
```
> Text split to sentences.
오늘 회의시간에 너무 졸아서 고수석님께 혼났다
TTS 수행 및 음성 변조 완료
```

## 6. TTS 음성 파일 재생

생성된 음성 파일을 로컬에서 재생할 수 있습니다. 먼저 TTS로 생성된 음성을 재생한 후, 변조된 음성을 확인할 수 있습니다.

```python
from IPython.display import Audio

# TTS된 음성 재생 (목소리 변조 이전)
Audio('./output/tmp.wav')
```

```python
# 변조된 목소리로 생성된 음성 재생
Audio('./output/result.wav')
```

## 7. 경고 메시지 정리

실행 중에 나타나는 다양한 경고 메시지는 `torch.load` 및 기타 라이브러리의 최신 업데이트와 관련이 있습니다. 이러한 경고는 주로 모델 로드 시 발생하며, 다음과 같은 내용이 포함됩니다:

- `torch.load` 사용 시, pickle 모듈과 관련된 보안 경고가 발생할 수 있습니다. 이 경고는 신뢰할 수 없는 모델 파일을 로드할 때 발생할 수 있는 잠재적 보안 문제를 나타냅니다. 
  - 이 문제는 PyTorch의 최신 업데이트에서 `weights_only=True` 설정을 권장함으로써 해결될 수 있습니다.
- `tqdm` 경고는 Jupyter 노트북 환경에서 발생하는 경고로, 최신 버전의 `ipywidgets` 설치를 통해 해결할 수 있습니다.

이러한 경고는 시스템의 작동에 영향을 미치지 않으며, 실제 TTS 및 음성 변조 작업은 정상적으로 수행됩니다. 필요한 경우 위 경고 메시지를 참고하여 최신 버전으로 업데이트하거나 설정을 변경해주는 것이 좋습니다.

# 3-2. Usage (사용 방법) - demo_2(gui).py
이 색션에서는 gui버전으로 실행하여 버튼 클릭을 통해 작동을 확인할 수 있습니다. 세팅한 가상환경을 실행한 상태로 아래 명령어로 demo파일을 켭니다.
```python
python demo_2(gui).py
```

그럼 아래와 같이 GUI 프로그램이 켜지게 되며 키보드와 마우스 입력을 통하여 demo를 실행해 볼 수 있습니다.
![alt text](readme_img/1.png)  
작동 영상도 첨부하오니 확인 바랍니다.  
YouTube 링크: https://youtu.be/1OdkNfU_sHQ?si=g9UoR35tEih_PDDq  
![alt text](readme_img/2.png)  

# 4. Features (기능 설명)

이 프로젝트는 실시간 TTS 시스템을 모듈화하여, 한국어 기반의 자연스러운 음성을 생성하고 제로샷 학습을 통해 음성 톤을 변조하는 기능을 제공합니다. 다음은 각 기능에 대한 설명입니다:

### 1. **Custom_TTS 클래스**
`Custom_TTS` 클래스는 전체 시스템의 핵심 기능을 구현하며, TTS 모델과 음성 변조 모델을 로드하고, 이를 통해 텍스트를 음성으로 변환한 후 원하는 화자의 음성 스타일로 변조할 수 있습니다.

#### `__init__(self, model_path='checkpoints_v2')`
- **기능**: 클래스 초기화 시 모델의 경로를 설정하고, CUDA(또는 CPU) 환경을 자동으로 탐지하여 설정합니다.
- **인자**
  - `model_path`: TTS 및 음성 변조 모델이 저장된 경로.
- **출력**: 모델이 저장된 경로와 사용 중인 장치(CPU 또는 GPU)를 출력.

#### `check_cuda(self)`
- **기능**: 시스템에서 CUDA가 사용 가능한지 확인하고, 사용 환경(CUDA 또는 CPU)을 설정합니다.
- **출력**: 사용 환경(CUDA 또는 CPU)을 출력.

#### `checkpoint_download(self)`
- **기능**: TTS 및 음성 변조를 위한 사전 학습된 모델이 로컬에 존재하지 않으면, 해당 모델을 다운로드하고 압축을 해제합니다.
- **출력**: 다운로드 및 압축 해제 상태를 출력하며, 에러가 발생할 경우 에러 로그를 출력.

#### `set_model(self, language='KR')`
- **기능**: 사전 학습된 TTS 및 음성 변조 모델을 로드하고, 기본 화자의 음성 임베딩을 설정합니다.
- **인자**
  - `language`: 사용할 언어 모델 (예: KR, EN, JP 등).
- **출력**: TTS 모델과 음성 톤 변조 모델의 로드 완료 메시지 및 기본 화자 음성 임베딩 완료 메시지.

#### `get_reference_speaker(self, speaker_path, vad=True)`
- **기능**: 참조할 화자의 음성 파일을 입력받아, 해당 화자의 목소리 스타일을 임베딩합니다. 제로샷 TTS를 위해 필요한 샘플 화자의 음성을 처리하는 단계입니다.
- **인자**
  - `speaker_path`: 참조할 화자의 음성 파일 경로.
  - `vad`: Voice Activity Detection(음성 감지) 활성화 여부. 켜면 음성 내에서 목소리가 있는 부분만 처리합니다.
- **출력**: 샘플 음성의 톤 임베딩 완료 메시지.

#### `make_speech(self, text, output_path='output', speed=1.1)`
- **기능**: 입력된 텍스트를 TTS를 통해 음성으로 변환한 후, 변조된 목소리로 결과물을 생성합니다. 결과는 로컬에 mp3 파일로 저장됩니다.
- **인자**
  - `text`: 변환할 텍스트.
  - `output_path`: 음성 파일이 저장될 경로.
  - `speed`: 음성 재생 속도. 기본값은 1.1로, 자연스러운 속도를 유지합니다.
- **출력**: TTS 변환 및 목소리 변조 과정을 통해 결과 파일이 저장됩니다.

### 2. **Down_and_extract 클래스**
`Down_and_extract` 클래스는 모델의 체크포인트 파일을 다운로드하고 압축을 해제하는 기능을 수행합니다.

#### `do(self, url, filename)`
- **기능**: 주어진 URL에서 파일을 다운로드하고, 다운로드 진행 상황을 `tqdm`으로 시각화하여 사용자에게 보여줍니다. 다운로드 후 압축 파일을 해제하고, 압축 해제 과정도 시각적으로 보여줍니다.
- **인자**
  - `url`: 다운로드할 파일의 URL.
  - `filename`: 다운로드할 파일의 이름.
- **출력**: 파일 다운로드 및 압축 해제 상태를 출력하며, 성공 여부를 반환합니다.


# 5. On-Device (온 디바이스)  
Jetson Orin Nano Developer Kit, Raspberry Pi 4 등의 싱글 보드에서의 구동 가능 여부도 아래 같이 시연합니다.  
싱글보드 환경에서는 Anaconda와 달리 venv 환경을 사용하여 구동하시오. Jetson과 Raspberry Pi 플랫폼에는 Anaconda 설치가 안되기 때문입니다.    
  
Jetson Orin Nano Developer Kit: https://youtu.be/IGO6aIywQJI  
![alt text](readme_img/3.png)  

# 6. 블로그
보다 자세한 기술 리뷰, 시행착오는 아래 블로그 링크 확인 바랍니다.  
https://blog.naver.com/112fkdldjs/223623704761  