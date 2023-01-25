# ESPnet model execute for Windows

Linux 운영체제에서 학습된 `ESPnet` ASR 모델을 Windows에서 실행하는 과정입니다.


### before starting

우선 `ESPnet`은 기본적으로 Windows를 지원하지 않습니다.
`Kaldi`, `Cuda` 등 여러 툴킷들을 사용하기에 많은 제약이 있기 때문에 손쉽게 Windows에서 Linux 시스템을 이용할 수 있는 WSL을 사용하였습니다.

본 글은 WSL2를 사용하므로 WSL2설치 후 개발환경 구축과 ESPnet 설치 및 실행 과정은 Linux에서와 거의 동일합니다. 
***


## 1. WSL2
WSL2은 Windows Subsystem for Linux 2 의 줄임말로, Windows의 가상화 기능을 통해 Windows에서 리눅스를 사용합니다.

[WSL을 사용하여 Windows에 Linux 설치][id]

[id]: URL "https://learn.microsoft.com/ko-kr/windows/wsl/install#step-4---download-the-linux-kernel-update-package"

WSL2 설치 후 OS는 Microsoft Store 에서 Ubuntu를 다운받아 사용합니다. (다른 OS도 사용가능합니다.)

[Ubuntu 다운로드](https://www.microsoft.com/store/productId/9PDXGNCFSCZV)

***
## 2. 개발환경 구축
Linux 환경 설정이 끝나면 Ubuntu 터미널이나 Windows 터미널을 이용하여 개발환경을 구축합니다.

### 1) Install Anaconda
1. [Anaconda 설치파일](https://www.anaconda.com/distribution/) 다운로드

2. 터미널에서 Anaconda 설치
```
# $ bash <설치파일명>

$ bash Anaconda3-2022.10-Linux-x86_64.sh
```
> 설치파일이 있는 경로로 이동하여 실행해야 합니다.


3. 경로 설정
```
$ source ~/.bashrc
```

### 2) Install Kaldi

#### 1. Git에서 Kaldi를 가져옵니다.
```
$ cd <설치할 경로>
$ git clone https://github.com/kaldi-asr/kaldi
```

#### 2. tool 설치
```
$ cd <kaldi-root>/kaldi/tools
$ ./extras/check_dependencies.sh
```
* `$ ./extras/check_dependencies.sh` 이 구문을 실행하고 나면 필요한 패키지 목록들을 가르쳐 주는데, 안내에 따라 모두 설치합니다.
* `$ apt install <패키지이름>` 으로 설치할 수 있습니다.

```
$ cd <kaldi-root>/kaldi/tools
$ make

# make 에 사용되는 CPU 개수를 설정해주어 더욱 빠르게 설치할 수 있습니다.
# 사용자 컴퓨터 하드웨어에 따라 다르지만, 대체로 4로 설정합니다.(낮은 사양에서는 사용 시 make 도중 멈춤이 발생합니다.)
# $ make -j <NUM-CPU>
```
* **BLAS 라이브러리는 Intel의 MKL을 사용합니다. (Intel CPU 사용 시)**
  * AMD CPU 사용 시 OpenBLAS 사용을 권장합니다.
  * 위의 필요한 패키지 목록 안내에 따라 `$ Run extras/install_mkl.sh`로 설치할 수 있습니다.

#### 3. Compile & Install
```
$ cd ../src

# 기본값인 mkl을 사용하므로 별다른 옵션 없이 configure 합니다.
# OpenBLAS 사용 시 별도의 옵션이 필요합니다.
$ sudo ./configure --shared
```
* 보통 이 과정에서 에러가 많이 발생합니다. (대다수는 openfst, mkl or OpenBLAS 등의 설치 오류입니다.)
* 관련 아티클들을 잘 찾아보고 디버깅하여 다시 `$ make` 해주어야 합니다.
* configure 결과가 모두 success인 것을 꼭 확인하시길 바랍니다.

```
# 시간이 오래 걸리니 <NUM-CPU>를 설정하시는 것을 권장드립니다.
$ make -j clean depend; make -j <NUM-CPU>
```
***

## 3. Install ESPnet
개발환경이 모두 구축되면, `ESPnet`을 설치합니다.

#### 1) Git에서 ESPnet을 가져옵니다.
```
# kaldi 설치 경로와 동일하게 설정하는 것을 권장드립니다.
$ cd <설치할 경로>
$ git clone https://github.com/espnet/espnet
```

#### 2) ESPnet에서 kaldi 사용
```
# espnet의 tools폴더에 kaldi를 연결합니다.
$ cd <espnet 경로>/tools
$ ln -s <kaldi 경로> .
```

#### 3) Python 환경을 구축합니다.
```
$ cd <espnet 경로>/tools
$ ./setup_anaconda.sh anaconda espnet 3.8
```

#### 4) ESPnet을 설치합니다.
```
$ cd <espnet 경로>/tools
$ make
```
***

## 4. 미리 학습된 ESPnet 모델 옮기기

_**가장 중요하고도 단순한 과정입니다.**_

수일에 걸쳐 많은 아티클들을 찾아보고 여러 방법을 시도해보았으나, 어디에도 **Linux 환경에서 학습된 사용자 생성 모델을 Windows 로 옮겨 실행하는 선례는 찾을 수 없었습니다.**

또한, 모델을 학습하고 성능평가 하는 예제들은 많이 찾아볼 수 있으나 학습과 성능평가가 끝난 모델을 실제 음성인식에 사용 할 수 있는    
**" 단일입력 <-> 단일출력 "** 의 과정은 시도하고 있지 않습니다.

이것이 위 글의 목적이며 따라서 이의 대한 과정을 공유합니다.

### 1) 미리 학습된 모델 추출

미리 학습된 모델에서 추출해야 하는 것은 총 2가지 입니다.
* `cmvn.ark`
* `model.acc.best`

> `cmvn.ark` 는 학습에 쓰인 데이터 신호에 대한 cepstral-domain 에서의 평균과 분산입니다. 미리 학습된 데이터 신호에서의 CMVN을 이용해 입력 데이터를 decoding 합니다.

> `model.acc.best` 는 모델이 학습과정에서 가장 높은 정확도를 보였던 epoch에서의 snapshot을 압축형태로 저장한 파일입니다. 쉽게 말하면 가장 정확한 모델이 저장된 파일입니다.

`cmvn.ark`과 `model.acc.best` 모두 `~/espnet/egs/librispeech/asr1/` 내부에 저장되어 있습니다.

### 2) 추출한 모델 옮기기

추출한 두가지 파일을 경로를 동일하게 해주어 Windows 내로 옮깁니다.

* `model.acc.best`는 `~exp/train_pytorch_<모델명>/results` 에 저장되어 있습니다.   
`train_pytorch_<모델명>`폴더 자체를 잘라내기 하여 붙여넣기 하면 경로 설정에 편리합니다.

### 3) `recog_wav.sh` 수정

`recog_wav.sh`를 이용해 단일입력을 decoding 합니다.

제가 생성한 모델은 lang model 을 사용하지 않으므로 `recog_wav.sh` 에서 LM 옵션을 모두 제거하였습니다.

ESPnet에서 제공하는 사전 학습된 모델을 사용하지 않고, 사용자가 생성한 모델을 사용하여 decoding 하기 위해서는 코드 수정이 필요합니다.

* 사전 학습된 모델을 사용하는 코드부분은 삭제하였으며 LM 옵션 제거와 텍스트 후처리 부분을 일부 추가하였습니다.
* [recog_wav2.sh](https://github.com/plhyun/KoreanASR-ESPnet/blob/main/recog_wav2.sh) 를 확인하시길 바랍니다.

### 4) Decoding 진행

터미널에서 `recog_wav.sh`를 다음과 같은 옵션으로 실행하여 decoding을 진행합니다.

```
# 어디서든 실행가능 하지만 아래 경우를 예시로 합니다.
$ cd <espnet 경로>/espnet/egs/librispeech/asr1

# 실행 옵션
# $ bash ../../../utils/<recog_wav.sh-name> --cmvn <cmvn.ark-root>/cmvn.ark --recog_model <model.acc.best-root>/model.acc.best --decode_config <decode.yaml-root>/decode.yaml wavefile

# ex)
$ bash ../../../utils/recog_wav2.sh --cmvn /root/espnet/egs/librispeech/asr1/data/train/cmvn.ark --recog_model /root/espnet/egs/librispeech/asr1/exp/train_pytorch_korean/results/model.acc.best --decode_config /root/espnet/egs/librispeech/asr1/conf/decode.yaml /root/wav/quiz3.wav
```
