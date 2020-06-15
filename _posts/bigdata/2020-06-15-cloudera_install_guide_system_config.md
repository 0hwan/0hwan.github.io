---
title: "Cloudera Manager Install Guide"
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

# 1. System Configuration

## 1.1 Disabling the Firewall
  * RHEL/CentOS 7 compatible REH:
    ```console
    $ sudo systemctl disable firewalld
    $ sudo systemctl stop firewalld
    ```

## 1.2 Setting SELinux mode
  * Check the SELinux state:
    ```console
    $ getenforce
    ```
    결과 값이 Disabled 가 아니면 수정을 한다
    ```console
    $ sudo vi /etc/selinux/config
    ```
    `SELINUX=enforcing` 를 `SELINUX=disabled` 로 변경한다
    System reboot 을 하면 적용 될 것이다

## 1.3 Setting the vm.swappiness Linux Kernel Parameter
  * Check the vm.swappiness:
    ```console
    $ sysctl vm.swappiness
    ```
    결과 값이 `vm.swappiness = 1` 인지 확인한다
    Cloudera recommends value 는 1 ~ 10 이 지만, 
    `1`을 설정 해주는 것을 추천한다.

    ```console
    $ sudo vi /etc/sysctl.conf
    ```
    `vm.swappiness=1` 를 추가한다
    System reboot 을 하면 적용 될 것이다

## 1.4 Disabling Transparent Hugepage Compaction
  * Check the vm.swappiness:
    ```console
    $ cat /sys/kernel/mm/transparent_hugepage/enabled
    ```
    ```console
    $ cat /sys/kernel/mm/transparent_hugepage/defrag
    ```
    * `[always] never` 이면 `enabled` 상태
    * `always [never]` 이면 `disabled` 상태
    
  * 설정 해제
    ```console
    $ sudo vi /etc/rc.local
    ```
    다음과 같이 추가 해준다
    ```console
    echo naver > /sys/kernel/mm/transparent_hugepage/enabled
    echo naver > /sys/kernel/mm/transparent_hugepage/defrag
    ```
    실행 권한을 준다
    ```console
    $ sudo chmod +x /etc/rc.local
    ```
    System reboot 을 하면 적용 될 것이다

    설정 확인 방법
    ```console
    $ sudo cat /proc/meminfo
    ```
    결과값 상태
    ```console
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    ```

## 1.5 User Process Limit  & Open Files Limit 
  * User Process Limit
    ```console
    $ ulimit -u
    ```
    `65535` 이하면 설정 변경 해준다
    ```console
    $ sudo vi /etc/security/limits.conf
    ```

    ```console
    * hard noproc 65535
    * soft noproc 65535
    ```
  * Open Files Limit
    ```console
    $ ulimit -n
    ```
    `1048576` 이하면 설정 변경 해준다
    ```console
    $ sudo vi /etc/security/limits.conf
    ```

    ```console
    * hard nofile 1048576
    * soft nofile 1048576
    ```
    System reboot 을 하면 적용 될 것이다

## 1.6 Diable an TUNED Service
  * 모든 튜닝 금지
    ```console
    $ sudo tuned-adm off
    ```
  * 서비스 정지
    ```console
    $ sudo systemctl stop tuned
    ```
  * 서비스 비활성
    ```console
    $ sudo systemctl disable tuned
    ```
    
## 1.7 Enable an NTP Service
  * Install the ntp package:
    ```console
    $ yum install ntp
    ```
  * Edit the /etc/ntp.conf file to add NTP servers
    ```bash
    server time.nist.gov  iburst
    ```
  * Start Service
    ```console
    $ sudo systemctl start ntpd
    ```
  * Enable Service
    ```console
    $ sudo systemctl enable ntpd
    ```

## 1.8 Java Install
  * Java 1.8
  * set JAVA_HOME