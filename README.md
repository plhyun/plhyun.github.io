# ESPnet model execute for Windows

Linux 운영체제에서 학습된 `ESPnet` ASR 모델을 Windows에서 실행하는 과정입니다.


### before starting

우선 `ESPnet`은 기본적으로 Windows를 지원하지 않습니다.
`Kaldi`, `Cuda` 등 여러 툴킷들을 사용하기에 많은 제약이 있기 때문에 손쉽게 Windows에서 Linux 시스템을 이용할 수 있는 WSL을 사용하였습니다.

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
$ make -j clean depend; make -j <NUM-CPU>
```
***
