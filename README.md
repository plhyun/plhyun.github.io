# plhyun.github.io

Linux 운영체제에서 학습된 ASR모델을 Windows에서 실행하는 과정입니다.

## before starting

우선 ESPnet는 기본적으로 Windows를 지원하지 않습니다. Kaldi, Cuda 등 여러 툴킷들을 사용하기에 많은 제약이 있기 때문에 손쉽게 Windows에서 Linux 시스템을 이용할 수 있는 WSL을 사용하였습니다.

## WSL2
WSL2은 Windows Subsystem for Linux 2 의 줄임말로, Windows의 가상화 기능을 통해 Windows에서 리눅스를 사용할 수 있게 합니다.
