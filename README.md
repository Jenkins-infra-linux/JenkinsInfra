# 🚀서버 부하 체크 프로젝트
### 📖 프로젝트 소개

이 프로젝트는 **서버 부하 모니터링**을 자동화하는 시스템입니다. **Jenkins**를 사용하여 매 1분마다 서버의 **uptime** 정보를 확인하고, 이를 기반으로 서버의 부하 상태를 실시간으로 체크합니다. 부하가 일정 수준을 초과할 경우, **로그 파일**에 해당 부하 정보를 기록하여, 시스템 관리자에게 **서버 상태**를 지속적으로 모니터링할 수 있는 환경을 제공합니다.

### 🛠️ 프로젝트 목표

- **Jenkins**와 스크립트를 활용하여 **1분 간격**으로 **크론 규칙**을 이용해 서버 부하를 자동으로 체크.
- 스크립트 실행을 위해 **실행 권한**과 **sudo 권한**을 부여하여 서버에서의 부하 체크가 원활히 이루어지도록 설정
- **grep** 명령어를 사용해 **uptime**의 출력에서 부하 값을 추출하고, 이를 기반으로 서버의 현재 상태를 판단.
- **서버 부하** 정보가 임계값을 초과할 경우, **>>** 연산자를 사용하여 **로그 파일**에 기록

### 📘 개발환경
![VirtualBox](https://img.shields.io/badge/VirtualBox-2F61B4?style=for-the-badge&logo=VirtualBox&logoColor=white)

# JenkinsInfra
## 1. Jenkins 설치
현재 실행 환경이 Ubuntu 24.0.2 LTS 이라 가정합니다.
우선 Jenkins 사이트에 접속하여 실행에 필요한 설치를 진행해줍니다.

```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.13" 2024-10-15
OpenJDK Runtime Environment (build 17.0.13+11-Debian-2)
OpenJDK 64-Bit Server VM (build 17.0.13+11-Debian-2, mixed mode, sharing)
```
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

우분투 환경에서 jenkins가 설치되었고 실행되었다면 다음 과정을 진행해주어야합니다.
우선 해당 우분투 서버의 ip주소(혹은 포트포워딩을 해주었다면 포트포워딩한 주소 ex. localhost):8080 으로 접속해줍니다.
접속을 완료한다면 다음과 같은 주소가 나타납니다.
![image](https://github.com/user-attachments/assets/f93794de-b586-4728-81a6-babd204cc0d6)

해당 password키는 sudo cat 명령어를 통해서 출력하여 복사-붙여넣기를 진행합니다.
이후 로그인 유저를 만들어주는 과정을 진행한다면 젠킨스를 실행 할 준비가 완료됩니다.


## Trouble Shooting

Jenkins에 권한을 부여하고 빌드했을때도 계속해서 권한문제가 발생하며 Failed가 뜸

![오류메시지](https://github.com/user-attachments/assets/726c048c-1c0a-48b7-90f0-79ab1b62ae48)




Jenkins pipeline script를 확인해보니 sudo 명령어가 제대로 적용되지 않았음
![스크린샷 2025-03-17 143001](https://github.com/user-attachments/assets/986454f2-809c-4e86-a223-00bc77fd98c8)


다시 sudo 명령어를 통해 Jenkins pipeline script 수정 후 빌드
![스크린샷 2025-03-17 143059](https://github.com/user-attachments/assets/bc96270e-565b-43f4-8bc7-9296e5ae981f)


![정상실행](https://github.com/user-attachments/assets/f855742d-187a-4cc8-9d44-ffb7b231c041)


정상적으로 작동
![정상실행콘솔](https://github.com/user-attachments/assets/6367eea1-841d-424f-b786-bb8f08301b4f)




cat 명령어를 통해 확인
![스크린샷 2025-03-17 144400](https://github.com/user-attachments/assets/9966c96c-812f-4063-91b0-7d9eb0241762)


