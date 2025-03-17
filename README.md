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
