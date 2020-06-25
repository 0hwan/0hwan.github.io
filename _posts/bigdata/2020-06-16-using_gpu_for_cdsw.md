---
title: "Using NVIDIA GPUs for Cloudera Data Science Workbench Projects"
# header:
#   image: https://live.staticflickr.com/8084/8396909762_813a2b1829_h.jpg
categories:
  - bigdata
tags:
  - Cloudera
  - Intall Guide
toc: true
toc_label: "목차"
toc_icon: "heart"
toc_sticky: true
---

# 1. [NVIDIA Driver 설치 (CentOS/RedHat 7.x)](https://docs.cloudera.com/documentation/data-science-workbench/1-6-x/topics/cdsw_gpu.html)
  * [Refer the NVIDIA documentation: CUDA Compatibility.](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

      |CDSW|OS&Kernel|NVIDIA Driver|CUDA|
      |---|:---:|:---:|:---:|
      |1.6.x|RHEL 7.4 3.10.0-862.9.1.el7.x86_64|418.56|CUDA 10.0|
      |1.6.x|RHEL 7.6 3.10.0-957.12.2.el7.x86_64|418.56|CUDA 10.0|

## 1.1 GPU 정보확인
  * lspci 명령으로 PCI Driver 정보를 확인하기 위해 pciutils 패키지 설치:
    ```console
    $ sudo yum install pciutils
    ```
  * VGA Driver 확인
    ```
    $ sudo lspci | grep VGI
    ```
    01:00.0 VGA compatible controller: `NVIDIA Corporation GM107 [GeForce GTX 750]` (rev a2)

## 1.2 Driver 설치를 위한 Build 패키지 설치
  * Linux에서 Dirver 설치를 위해서는 컴파일 해야 한다. 필요한 패키지를 설치:
    ```console
    $ sudo yum install -y flex gcc gcc-c++ redhat-rpm-config strace \
        rpm-build make pkgconfig gettext automake \
        gdb bison libtool autoconf gcc-c++ gcc-gfortran \
        binutils rcs patchutils wget
    ```
  * Kernel의 헤더, 개발용 헤더 및 스크립트 패키지 설치해야 하는데 자신의 커널 버전과 동일해야 하므로 uname -r 명령어를 사용:
    ```
    $ sudo yum install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
    ```

## 1.3 nouveau 비활성화
  * 비활성 설정:
    ```console
    $ sudo cat <<EOT >> /etc/modprobe.d/blacklist.conf
    blacklist nouveau
    options nouveau modeset=0
    EOT
    ```
    nouveau는 리눅스에 기본으로 탑재된 그래픽 드라이버인데 NVIDIA 드라이버와 충돌일 발생하여 설치가 제대로 되지 않는 문제가 발생한다
    커널 모듈 비활성화 할때는 /etc/modprobe.d/ 에 해당 정보를 저장 하면 된다, NVIDIA Driver 설치 파일을 실행하면 nouveau 때문에 설치 할수 없다고 나오고,
    disabled 시키기 위해 설정파일을 생성 할까요 나온다, yes 를 선택하면 `/etc/modprobe.d/nvidia-installer-disable-nouveau.conf` 생성 된다,
    그리고 재부팅 후 다시 드라이버 설치를 진행하면 OK, 수동으로 설정파일을 생성해서 드라이버를 설치하고 싶다면 다음과 같이 한다.
    
  * initramfs 업데이트 후 재부팅:
    ```console
    $ sudo mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
    $ sudo dracut -v /boot/initramfs-$(uname -r).img $(uname -r)
    $ sudo reboot
    ```
    init RAM Disk는 메모리상의 가상의 디스크로써 initramfs이란 파일 시스템을 가지고 있다. 이 파일 시스템에는 커널이 초기에 동작할때 필요한 드라이버나 프로그램, 바이너리 파일 등을 가지고 있다.

## 1.4 NVIDIA Driver Download & Install
  * Dowonload:
    ```console
    $ export NVIDIA_DRIVER_VERSION=418.56
    $ wget http://us.download.nvidia.com/XFree86/Linux-x86_64/${NVIDIA_DRIVER_VERSION}/NVIDIA-Linux-x86_64-${NVIDIA_DRIVER_VERSION}.run
    ```
  * Driver install
    ```console
    $ sudo chmod +x NVIDIA-Linux-x86_64-${NVIDIA_DRIVER_VERSION}.run
    $ sudo ./NVIDIA-Linux-x86_64-${NVIDIA_DRIVER_VERSION}.run -asq
    ```
    -a : --accept-license  
    -s : --silent  
    -q : --no-questions
  * Driver 활성 및 정보확인
    ```console
    $ sudo /usr/bin/nvidia-smi -pm 1
    $ sudo /usr/bin/nvidia-smi
    ```
    -pm: --persistence-mode=   Set persistence mode: 0/DISABLED, 1/ENABLED

## 1.5 CM(Cloudera Manager)에서 CDSW GPU 활성
  * CM > CDSW > Configuration(구성) > Enable GPU Support 체크(활성화) > 서비스 재기동