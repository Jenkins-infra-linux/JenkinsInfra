# 🚀서버 부하 체크 프로젝트

## 🤝 Team Members
| <img src="https://github.com/kcklkb.png" width="200px"> | <img src="https://github.com/woody6624.png" width="200px"> | <img src="https://github.com/parkjhhh.png" width="200px"> | <img src="https://github.com/unoYoon.png" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김창규](https://github.com/kcklkb) | [김우현](https://github.com/woody6624) | [박지혜](https://github.com/parkjhhh) | [윤원호](https://github.com/unoYoon) |

### 📖 프로젝트 소개

이 프로젝트는 **서버 부하 모니터링**을 자동화하는 시스템입니다. 

**Jenkins**를 사용하여 매 1분마다 서버의 **uptime** 정보를 확인하고, 이를 기반으로 서버의 부하 상태를 실시간으로 체크합니다. 

**로그 파일**에 해당 부하 정보를 기록하여, 시스템 관리자에게 **서버 상태**를 지속적으로 모니터링할 수 있는 환경을 제공합니다.

### 🛠️ 프로젝트 목표

- **Jenkins**와 스크립트를 활용하여 **1분 간격**으로 **크론 규칙**을 이용해 서버 부하를 자동으로 체크.
  
- 스크립트 실행을 위해 **실행 권한**과 **sudo 권한**을 부여하여 서버에서의 부하 체크가 원활히 이루어지도록 설정
  
- **grep** 명령어를 사용해 **uptime**의 출력에서 부하 값을 추출하고, 이를 기반으로 서버의 현재 상태를 판단.
  
- **서버 부하** 정보가 정상일 때와 임계값을 초과할 경우를 구분하여 **로그 파일**에 기록.

### 📘 개발환경
<img src="https://img.shields.io/badge/VMware-607078?style=for-the-badge&logo=vmware&logoColor=white" alt="VMware" /> ![VirtualBox](https://img.shields.io/badge/VirtualBox-2F61B4?style=for-the-badge&logo=VirtualBox&logoColor=white) <img src="https://img.shields.io/badge/ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu"> <img src="https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white" alt="Jenkins" />

## 1. Jenkins 설치
현재 실행 환경이 Ubuntu 24.0.2 LTS 이라 가정합니다.
우선 Jenkins 사이트에 접속하여 실행에 필요한 설치를 진행하여줍니다.

다만 JDK-17에 대한 설치가 완료된 상태라면 아래 과정을 생략하여줍니다.
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

이후 로그인 유저를 만들어주는 과정을 진행해줍니다.

![image](https://github.com/user-attachments/assets/6950b01d-f5b8-40b4-a85a-f6a98365a405)

해당 화면에서 파이프라인을 선택하고 진행해주며 파이프라인 스크립트에서 아래의 과정을 진행해줍니다.

## 2. Jenkins 서버 부하 체크 스크립트 자동 실행

<br>

아래 순서대로 설정하면, Jenkins에서 서버 부하 체크 스크립트를 자동으로 실행할 수 있습니다.

<br>

1. **Jenkins 사용자로 전환**

    sudo su - jenkins
    
    - Jenkins 설치 후, `jenkins` 사용자 환경으로 진입합니다.
    
2. **test.sh와 server_uptime.log 소유권 변경**
    
    ```bash
    sudo chown jenkins:jenkins /home/ubuntu/server/test.sh
    sudo chown jenkins:jenkins /home/ubuntu/server/server_uptime.log
    ```
    
    - `/home/ubuntu/server/test.sh`과 /home/ubuntu/server/server_uptime.log 파일의 소유자와 그룹을 `jenkins`로 변경하여 Jenkins가 해당 파일에 접근/실행할 수 있도록 합니다.
      
3. **sudoers 설정**
    
    ```bash
    sudo visudo
    ```
    
    - `visudo`로 sudoers 파일을 연 뒤, 아래 내용을 추가합니다.
        
        ```
        # Allow members of group sudo to execute any command
        %sudo   ALL=(ALL:ALL) ALL
        jenkins ALL=(ALL) NOPASSWD: ALL
        ```
        
    - Jenkins 사용자가 sudo 명령을 비밀번호 없이 실행할 수 있도록 허용합니다.
      
4. **테스트 스크립트 생성**
    
    ```bash
    mkdir server
    cd server
    vi test.sh
    ```
    
    - 아래와 같이 `test.sh`에 로직을 작성합니다.
        
        ```bash
        LOG_FILE="/home/ubuntu/server/server_uptime.log"
        THRESHOLD=5.0
        LOAD=$(uptime | grep -o 'load average: .*' | awk '{print $3}' | tr -d ',')
        
        # 부하 비교를 정확하게 하기 위해 수정
        if (( $(echo "$LOAD > $THRESHOLD" | bc -l) )); then
            echo "$(date): CPU 과부화가 인지되었습니다 - $LOAD" >> $LOG_FILE
        else
            echo "$(date): CPU 과부화 문제가 발견되지 않았습니다.: $LOAD" >> $LOG_FILE
        fi

        ```
        
5. **Jenkins 파이프라인 구성**

    - Jenkins 웹 UI(8080포트 접속)에서 새로운 파이프라인을 생성하거나 기존 파이프라인을 수정합니다.
    - 파이프라인 스크립트 예시:
        
        ```groovy
        pipeline {
            agent any
            triggers {
                cron('* * * * *') // 매 1분마다 실행
            }
        
            stages {
                stage('Check Server Load') {
                    steps {
                        script {
                            // sudo를 사용하여 test.sh 스크립트 실행
                            sh 'sudo /home/ubuntu/server/test.sh'
                        }
                    }
                }
            }
        
            post {
                failure {
                    echo "cpu과부화로 인하여 에러 메시지 발생"
                }
            }
        }
        ```
        
    - 이 스크립트를 저장하면 매 분마다 `test.sh`를 실행하여 서버 부하를 확인하고 로그를 남기도록 동작합니다.


위 단계를 모두 완료하면 Jenkins 파이프라인이 지속적으로 cron값을 기준으로 서버 부하 체크 스크립트를 실행하며, CPU 부하가 기준치를 초과할 경우 로그 파일에 기록하고 필요에 따라 알림을 보낼 수 있습니다.

## 3. 테스트
<br>

지금 빌드를 클릭하여서 파이프라인을 실행합니다.

<br>

![image](https://github.com/user-attachments/assets/c9fb52c2-3c1e-4e1e-810a-fd7337427768)

### 변경된 server_uptime.log 를 cat 명령어를 이용해 확인

```bash
ubuntu@ubuntu:~/server$ stress-ng --cpu 8 --vm 2 --vm-bytes 95% --io 8 --timeout 60s
stress-ng: info:  [11173] setting to a 1 min, 0 secs run per stressor
stress-ng: info:  [11173] dispatching hogs: 8 cpu, 2 vm, 8 io
stress-ng: info:  [11184] io: this is a legacy I/O sync stressor, consider using iomix instead
stress-ng: info:  [11173] skipped: 0
stress-ng: info:  [11173] passed: 18: cpu (8) vm (2) io (8)
stress-ng: info:  [11173] failed: 0
stress-ng: info:  [11173] metrics untrustworthy: 0
stress-ng: info:  [11173] successful run completed in 1 min, 0.07 secs

ubuntu@ubuntu:~/server$ cat server_uptime.log
Mon Mar 17 05:03:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.00
Mon Mar 17 05:04:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.00
Mon Mar 17 05:05:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.16
Mon Mar 17 05:06:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.38
Mon Mar 17 05:07:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.54
Mon Mar 17 05:08:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.28
Mon Mar 17 05:09:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.18
Mon Mar 17 05:10:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.14
Mon Mar 17 05:11:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.21
Mon Mar 17 05:12:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.16
Mon Mar 17 05:13:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.30
Mon Mar 17 05:14:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.43
⚠️
Mon Mar 17 05:15:01 AM UTC 2025: CPU 과부화가 인지되었습니다 - 9.91
⚠️
Mon Mar 17 05:16:01 AM UTC 2025: CPU 과부화가 인지되었습니다 - 5.76
Mon Mar 17 05:17:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 2.43
Mon Mar 17 05:18:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.97
Mon Mar 17 05:19:01 AM UTC 2025: CPU 과부화 문제가 발견되지 않았습니다.: 0.51

```













## 4. Trouble Shooting

➡️ Jenkins에 권한을 부여하고 빌드했을때도 계속해서 **권한문제가 발생하고 Failed**가 뜨며 실패합니다.

![Image](https://github.com/user-attachments/assets/53ad0ea4-7a9f-4d06-ac4a-8fe574ae4892)




➡️ Jenkins pipeline script를 확인해보니 sudo 명령어가 제대로 쓰이지 않았음을 확인할 수 있었습니다.

![Image](https://github.com/user-attachments/assets/a5a1249e-966a-4139-96ef-639d3753700c)

➡️ **sudo 명령어**를 통해 Jenkins pipeline script 수정 후 빌드했습니다.

![스크린샷 2025-03-17 143059](https://github.com/user-attachments/assets/bc96270e-565b-43f4-8bc7-9296e5ae981f)


![정상실행](https://github.com/user-attachments/assets/f855742d-187a-4cc8-9d44-ffb7b231c041)


➡️ 정상적으로 작동하는것을 확인할 수 있습니다.

![Image](https://github.com/user-attachments/assets/8758a699-c81b-4d3d-86b6-383c759680fd)




➡️ 최종적으로 cat 명령어를 통해 로그 결과를 확인했습니다.


![Image](https://github.com/user-attachments/assets/f1ea2570-6f34-4940-a775-d91c3cd259f7)


--- 

![image](https://github.com/user-attachments/assets/d6b12447-80ff-4dee-ac01-699931735744)

- **문제 원인**
  
    - `server.uptime.log`와 `test.sh` 파일에 적절한 권한이 없었습니다.
    - 두 파일의 내용(경로, 스크립트 등)이 정확히 작성되지 않았습니다.
- **해결 방법**
  
    1. **파일 권한 설정**
        
        ```bash
        
        chmod +x /home/ubuntu/server/test.sh
        sudo chown jenkins:jenkins /home/ubuntu/server/test.sh
        sudo chown jenkins:jenkins /home/ubuntu/server/server.uptime.log
        ```
        
    2. **파일 내용 및 경로 재확인**
 
        - `test.sh` 내 스크립트 로직 및 실행 경로 확인
        - `server.uptime.log` 생성/경로 확인
        - Jenkins 파이프라인에서 호출되는 경로와 실제 파일 경로가 일치하는지 검증

이 과정을 통해 권한 문제와 파일 경로/내용 오류를 수정하여 정상적으로 파이프라인을 실행할 수 있었습니다.



