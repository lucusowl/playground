# Linux 초기 설정

CLI환경에서의 작업으로 서술[^1]

## 작업

1. [(선택) 시스템 정보 확인](#선택-시스템-정보-확인)
2. [관리자 계정 활성화](#관리자-계정-활성화)
3. [네트워크 설정](#네트워크-설정)
4. [패키지 업데이트](#패키지-업데이트)
5. [시스템 시간 동기화](#시스템-시간-동기화)
6. [SSH 원격 연결 활성화](#ssh-원격-연결-활성화)
7. [(선택) 기타 편의 설정](#선택-기타-편의-설정)
   1. 키보드 레이아웃
   2. 시스템 언어 & 로케일
   3. 응용 계층 서비스 설정

### (선택) 시스템 정보 확인

현재 사용/접속 중인 계정/서버/장비의 정보를 모르거나 인계받지 못한 경우  
미리 확인하여 기초 작업 진행 및 차후 문제 발생시 대비

추가적으로 **라이센스 계약을 확인**하여 이용하는데에 문제가 없는 지 확인할 것  

```sh
# 서버 및 계정 정보 확인
hostnamectl

# 개별 정보 확인
cat /etc/*release* # 운영체제 정보
uname -a           # 커널 정보
hostname           # 호스트 명
whoami             # 계정 명
```

### 관리자 계정 활성화

`root`, 관리자 계정은 초기 비밀번호가 없음
비밀번호가 없을 경우 사용할 수 없으니[^2] 비밀번호를 설정하여 `root` 관리자 계정을 활성화

```sh
# 관리자 계정 root 비밀번호 설정
sudo passwd root
# 1. 현 계정 비번 입력
# 2. root 새 비번 입력
# 3. 새 비번 재입력

# 관리자 계정 접근 확인
su root # 관리자 계정으로 로그인, 뒤에 root는 생략하여도 됨
# prompt가 '$'가 아닌 '#'로 끝남
whoami # root
exit # 로그아웃
```

### 네트워크 설정

유선 연결이면 기본 설정으로 자동 연결됨  
무선 연결과 같이 추가 설정이 필요하거나 네트워크 연결이 되지 않는다면 진행

1. 네트워크 연결 확인

   1) 현재 연결되어 있는 인터페이스 확인
      ```sh
      ls /sys/class/net
      # lo, eno0, eth0, wlan0, ...
      # linux NIC 종류 참고
      
      ip link
      # 각 인터페이스 세부 정보 확인
      ```
      무선 연결을 원할 경우, `w`로 시작하는 인터페이스명이 있어야 합니다

   2) 네트워크 연결이 되는 지 확인
      ```sh
      # 1) 인터페이스 연결 정보 확인
      ip a # ifconfig 명령어와 유사
      # 해당 NIC에 inet ... 내용과 함께 출력되면 연결성공

      # 2) 라우팅 테이블 내용 확인
      ip route # route 명령어와 유사
      # ip 주소와 함께 네트워크 연결된 NIC 내용이 출력되면 연결성공

      # 3) 네트워크에 직접 연결 확인
      ping {ip-address} # 연결하고자 하는 네트워크 IP주소에 ping
      ```

      외부망 연결 확인
      ```sh
      ping 8.8.8.8
      # google DNS server 에 ping 연결 시도
      # ping이 수신된다면 연결 성공

      # NIC가 두 개 이상 연결되어 있다면, "-I {NIC}" 로 지정
      # e.g. > ping -I wlan1 8.8.8.8
      ```

2. 네트워크 설정 변경

   `networkd` 또는 `NetworkManager`으로 직접 설정하는 방법도 있지만  
   `Netplan`으로 구성하는 방법이 가장 간편하여 권장  

   1) 네트워크 설정 변경
      ```sh
      # 설정파일 확인 및 수정
      ls /etc/netplan # e.g. 50-cloud-init.yaml
      sudo vim /etc/netplan/{file-name}
      ```

      아래의 내용을 참고하여 파일 내용을 수정 (`#`뒤 내용은 주석이므로 삭제하여도 무방)
      ```yaml
      network:
        ethernets: # ethernet devices
          eth0:
            dhcp4: true
            optional: true
        wifis: # wireless devices
          wlan0:
            access-points:
              "{wifi-ssid}":
                password: "{wifi-password}"
                hidden: true # wifi SSID가 숨김일 경우
            dhcp4: true # 자동 IP 할당여부
            optional: true
        version: 2
        renderer: NetworkManager # 기본 networkd가 아닌 NetworkManager 으로 설정
      ```

      구체적인 netplan 구성 및 추가설정은 [공식문서](https://netplan.readthedocs.io/en/stable/howto/)를 참고

   2) 변경된 네트워크 설정 적용
      ```sh
      # sudo netplan try # apply test
      sudo netplan apply
      ```

      설정을 적용하고 네트워크에 정상적으로 연결되는 지 확인

   3) (선택) 네트워크 툴 설치

      세부적인 네트워크 설정을 위해 네트워크 툴을 설치해두는 것을 권장  
      라즈베리파이 제공 ubuntu server 22.04 경우, NetworkManager가 기본 설치되어 있지 않으므로 `NetworkManager` 설치  
      (본인이 많이 사용하는 네트워크 툴이 있다면 해당 툴 설치)  
      ```sh
      sudo apt install network-manager
      ```

#### 부팅시 자동 무선 네트워크 탐색 해제/설정

```
[***   ] A start job is running for wait for network to be configured (0s / no limit)
```
위 메세지가 뜨며 부팅시 네트워크 연결에 오랜 시간 대기하는 경우가 있음  
부팅시 네트워크 자동 연결이 수행되는 것  
이를 해제하여 부팅 후 수동연결을 원할 경우
```sh
# Network connection에서 기다리지 않게 비활성화 설정
sudo systemctl disable systemd-networkd-wait-online.service
# 다른 서비스에 의해 서비스가 활성화되지 않게 mask 설정
sudo systemctl mask systemd-networkd-wait-online.service
```

반대로 다시 설정하고자 한다면 역으로 진행
```sh
sudo systemctl unmask systemd-networkd-wait-online.service # mask 풀기
sudo systemctl enable systemd-networkd-wait-online.service # 서비스 활성화
```

### 패키지 업데이트

설치된(기본제공되는) 프로그램들의 기능/보안 사항을 최신상태로 업데이트  
패키지 관리자를 통해 필요한 프로그램을 설치하는 것을 권장  

#### (선택) 패키지 저장소 미러 변경

기본 저장소 위치[^3]가 다운로드하는 지역과 멀 경우, 업데이트하는 데에 시간이 오래 걸림  
빠르게 업데이트하기 원할 경우, 가까운 저장소 미러 주소로 변경

대한민국으로 등록된 저장소 미러 주소는 <https://launchpad.net/ubuntu/+archivemirrors> 에서 확인가능

- mirror.kakao.com, 카카오, 많이 설정하는 편
- ftp.kaist.ac.kr, 카이스트, 속도는 빠르지만 접속자가 많은 편

```sh
# 예시)
# {orgin-addr}      : archive.ubuntu.com
# {new-mirror-addr} : mirror.kakao.com

# 일괄 변경
sed -i 's/{orgin-addr}/{new-mirror-addr}/g' /etc/apt/sources.list

# 또는 

# 직접 변경
sudo vim /etc/apt/sources.list
# 바꿀 내용이 많으므로 vim의 치환 활용
# :%s/{orgin-addr}/{new-mirror-addr}/g
```

#### 패키지 설치 진행

```sh
sudo apt update # 패키지 최신 정보 저장소에서 가져오기(설치되지 않음)
# sudo apt list --upgradable # 업데이트가 필요한 패키지 목록
sudo apt -y upgrade # 최신 패키지로 업데이트(설치 진행)

# 출력라인 맨마지막 결과보고에서
# N upgraded, N newly installed, N to remove and N not upgraded
# 'not upgraded'인 패키지는 의존성 문제로 최신버전으로 설치가 보류된 것
# 아래 명령어를 입력해 업데이트
sudo apt -y dist-upgrade # 의존성 확인포함됨, 필요한 패키지까지 추가/삭제하며 업데이트

# 아래부터는 선택사항
sudo apt check # 패키지 업데이트 및 파손된 의존성 확인
sudo apt autoremove # 의존성으로 설치된 패키지인데 더이상 사용되지 않으면 삭제
sudo apt autoclean # 저장소에 더이상 없거나, 불완전하게 다운로드되거나, 최신버전이 존재하는 오래된 패키지/아카이브 삭제

# 기호에 맞게 필요한 툴 설치
# build-essential, unzip, tree, multitail, net-tools, glances, grc, ...
```

### 시스템 시간 동기화

### SSH 원격 연결 활성화

유지보수 및 추가 개발을 위해 최소한 관리자 용도로라도 원격으로 연결할 수 있게 설정

### (선택) 기타 편의 설정

1. 키보드 레이아웃
2. 시스템 언어 & 로케일
3. 응용 계층 서비스 설정

## References

- [configuring_basic_system_settings, redhat.com, RHEL 8 documentation, Accessed:240418](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/8/html-single/configuring_basic_system_settings/index)
- [Ubnutu 22.03 Server 초기설정, tistory blog, 220602, Accessed:240418](https://zosystem.tistory.com/323)
- [Configure networking with Netplan, tutorial, 231220, Accessed:240421](https://vitux.com/how-to-configure-networking-with-netplan-on-ubuntu/)
- [How to install Ubuntu On Your Raspberry Pi, tutorial, Accessed:240421](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi)
- [Obsoleted net-tools programs overview, The Linux Foundation Wiki,  210821, Accessed:240421](https://wiki.linuxfoundation.org/networking/net-tools)
- [How to configure your computer to connect to your home Wi-Fi network, Netplan Documentation, Accessed:240421](https://netplan.readthedocs.io/en/stable/examples/#how-to-configure-your-computer-to-connect-to-your-home-wi-fi-network)

[^1]: 서버구축과 같이 GUI환경이 필요로 하지 않는 환경도 포함하기 위함. GUI,TUI와 같은 환경에서는 제시한 작업목록을 참고하여 진행  
[^2]: "Authentication failure"가 뜨면서 서비스가 거부될 것  
[^3]: 위치를 한국으로 설정하면 초기 기본 주소: kr.archive.ubuntu.com, 카이스트 서버  
