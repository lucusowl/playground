# Linux 초기 설정

CLI환경에서의 작업으로 서술[^1]

## 작업

1. [(선택) 시스템 정보 확인](#선택-시스템-정보-확인)
2. [관리자 계정 활성화](#관리자-계정-활성화)
3. 네트워크 설정
4. 패키지 업데이트
5. (선택) 시스템 언어 설정
6. 시스템 시간 동기화
7. SSH 원격 연결 활성화
8. (선택) 기타 편의 설정
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

## References

- [configuring_basic_system_settings, redhat.com, RHEL 8 documentation, Accessed:240418](https://access.redhat.com/documentation/ko-kr/red_hat_enterprise_linux/8/html-single/configuring_basic_system_settings/index)
- [Ubnutu 22.03 Server 초기설정, tistory blog, 220602, Accessed:240418](https://zosystem.tistory.com/323)

[^1]: 서버구축과 같이 GUI환경이 필요로 하지 않는 환경도 포함하기 위함. GUI,TUI와 같은 환경에서는 제시한 작업목록을 참고하여 진행  
[^2]: "Authentication failure"가 뜨면서 서비스가 거부될 것  
