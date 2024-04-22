# Linux 초기 설정

CLI환경에서의 작업으로 서술[^1]

## 작업

1. [(선택) 시스템 정보 확인](#선택-시스템-정보-확인)
2. [관리자 계정 활성화](#관리자-계정-활성화)
3. [네트워크 설정](#네트워크-설정)
4. [패키지 업데이트](#패키지-업데이트)
5. [(선택) 시스템 언어 설정](#선택-시스템-언어-설정)
6. [시스템 시간 동기화](#시스템-시간-동기화)
7. [SSH 원격 연결 활성화](#ssh-원격-연결-활성화)
8. [(선택) 기타 편의 설정](#선택-기타-편의-설정)
   1. 키보드 레이아웃
   2. 시스템 로케일
   3. 응용 계층 서비스 설정

### (선택) 시스템 정보 확인

현재 사용/접속 중인 계정/서버/장비의 정보를 모르거나 인계받지 못한 경우  
미리 확인하여 기초 작업 진행 및 차후 문제 발생시 대비

추가적으로 **라이센스 계약을 확인**하여 이용하는데에 문제가 없는 지 확인  

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
비밀번호가 없을 경우 사용할 수 없으니[^2] 비밀번호를 설정하여 활성화

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

### 패키지 업데이트

설치된(기본제공되는) 프로그램들의 기능/보안 사항을 최신상태로 업데이트  
수동설치보다는 패키지 관리자를 통해 필요한 프로그램을 설치하는 것을 권장  

#### (선택) 패키지 저장소 미러 변경

기본 저장소 위치[^3]가 다운로드하는 지역과 멀 경우, 업데이트하는 데에 시간이 오래 걸림  
빠르게 업데이트하기 원할 경우, 가까운 저장소 미러 주소로 변경

우리나라로 등록된 저장소 미러 주소는 <https://launchpad.net/ubuntu/+archivemirrors> 에서 확인가능

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

### (선택) 시스템 언어 설정

### 시스템 시간 동기화

### SSH 원격 연결 활성화

유지보수 및 추가 개발을 위해 최소한 관리자 용도로라도 원격으로 연결할 수 있게 설정

### (선택) 기타 편의 설정

1. 키보드 레이아웃
2. 시스템 로케일
3. 응용 계층 서비스 설정

## References

- [configuring_basic_system_settings, redhat.com, RHEL 8 documentation, Accessed:240418](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/8/html-single/configuring_basic_system_settings/index)
- [Ubnutu 22.03 Server 초기설정, tistory blog, 220602, Accessed:240418](https://zosystem.tistory.com/323)

[^1]: 서버구축과 같이 GUI환경이 필요로 하지 않는 환경도 포함하기 위함. GUI,TUI와 같은 환경에서는 제시한 작업목록을 참고하여 진행  
[^2]: "Authentication failure"가 뜨면서 서비스가 거부될 것  
[^3]: 위치를 한국으로 설정하면 초기 기본 주소: kr.archive.ubuntu.com, 카이스트 서버  
