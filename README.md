# linux-stress-test
 `stress`와 `sysstat` 도구를 사용하여 CPU와 I/O 부하를 시뮬레이션하고, `mpstat` 및 `pidstat`로 시스템 성능 지표를 확인
<br>
<br>
- ## 개요 📋
  - **Linux 시스템**에서 부하 테스트를 진행하고 성능을 모니터링하는 방법을 다룬다.
  - `stress`와 `sysstat` 도구를 사용하여 CPU와 I/O 부하를 시뮬레이션하고, `mpstat` 및 `pidstat`로 시스템 성능 지표를 확인한다
  - load average(평균 부하)와 CPU사용률, I/O대기 (iowait) 등을 통합적으로 관찰하여 부하의 원인을 정확하게 진단 할 수 있도록 한다.



- ## 필수 도구 📦
  - `stress`
  - `sysstat` (`mpstat`, `pidstat` 포함)

- ## 설치 방법
  - ```bash
    $ sudo apt install stress sysstat
    
- ## 실습 환경 🖥️
  - Ubuntu 22.04.5 LTS
  - 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz (2core)
    

- ## 부하 측정 지표 📈
  - uptime: 시스템의 평균 부하를 1분, 5분, 15분 간격으로 확인.
  - mpstat: CPU 사용량 및 각 코어별 통계 확인.
  - pidstat: 개별 프로세스의 CPU 및 메모리 사용률을 모니터링.


## 📝시나리오 1 : cpu 집약적인 프로세스
### 목표: CPU 부하가 100%일 때 시스템의 반응을 관찰.
   ### 1. 첫 번째 터미널에서 stress 명령을 실행하여 CPU 사용량이 100%인 시나리오를 시뮬레이션

   ```
   $ stress --cpu 1 --timeout 600
   ```

   <br>
   <br>
   
   ### 2. 그런 다음 두번째 터미널에서 uptime 을 실행하여 평균 부하의 변화를 확인한다.
   ```
   $ watch -d uptime
   ```
   watch : 주기적으로 명령어를 실행하고 결과를 화면에 갱신
   
   -d : 출력의 변경된 부분을 하이라이팅
   <br>
   ![image](https://github.com/user-attachments/assets/78a32251-0828-440c-a349-5174a222bbea)
   <br>
   **1분간 부하가 점점 증가하더니 최대 부하가 된 모습**

   <br>
   <br>
   
   ### 3. 세 번째 터미널에서 mpstat을 실행하여 CPU 사용량의 변화를 관찰
   - P ALL : 모든 CPU 코어의 통계정보  표시 ex) -P 0은 0번 코어
   - 5 : 5초에 한번 갱신
   - 1 : 최초 1회만 보여줌
   ```
   $ mpstat -P ALL 5 1
   ```
   0번 cpu의 사용률이 100%에 가까워졌다.
   

   ### 4. 그렇다면 어떤 프로세스가 CPU 사용량을 100%로 만드는가?
   ```
   $ pidstat -u 5 1
   ```
   ![image](https://github.com/user-attachments/assets/847cfca5-91dd-40f2-9b31-9c3b1b605524)
   stress 프로세스의 cpu 사용량이 100% 라는 것을 확인


   <br>
   <br>
   

## 📝시나리오 2: I/O 집약적인 프로세스 
### 목표: CPU 부하가 100%일 때 시스템의 반응을 관찰.

   ### 1. stress 명령을 계속 실행하지만 sync를 계속 실행하여 I/O 부하를 시뮬레이션
   ```
   $ stress -i 1 --timeout 600
   ```

   ### 2. 다시 두 번째 터미널에서 uptime을 실행하여 평균 부하의 변화를 관찰
   ```
   $ watch -d uptime
   ```
   ![image](https://github.com/user-attachments/assets/45471ae5-b103-4873-b2fd-52a1b203d3ba)
   <br>
   **1분 평균 부하는 마찬가지로 최대치에 가깝게 나타난다.**

   ### 3. 그런 다음 세 번째 터미널에서 mpstat을 실행하여 CPU 사용량의 변화를 관찰.
   ```
   $ mpstat -P ALL 5 1
   ```
   ![image](https://github.com/user-attachments/assets/360f7243-83a6-4f02-a68b-1c7822b415c4)

  cpu 사용량은 낮지만 평균 부하는 높게 나타난 이유는
  <br>
  평균 부하가 대기중인 프로세스를 기반으로 계산되는데,
  <br>
  i/o 병목 현상 때문에 작업 대기중인 프로세스가 많아진 것이다.

  ### 4. 어떤 프로세스가 높은 iowait을 유발하는가?  
  ```
  $ pidstat -u 5 1
  ```
  ![image](https://github.com/user-attachments/assets/3ec57de1-3759-4ffc-a540-72428e55dcc1)
  
  <br>
  
  **여전히 stress프로세스로 인해 발생**

## 📝시나리오 3: 프로세스 수가 많은 시나리오

### 목표: 다수의 프로세스가 CPU 자원을 경합할 때의 부하 확인.
   시스템에서 실행 중인 프로세스 수가 CPU 처리 용량을 초과하면 CPU를 기다리는 프로세스가 발생한다.
   <br>
   stress 를 계속 사용하지만 이번에는 8개의 프로세스를 시뮬레이션한다.


   ### 1. 8개의 cpu 코어에 부하를 주도록 한다.
   
   ```
   $ stress -c 8 --timeout 600
   ```

  ### 2. 평균 부하 확인.
  ```
  $ uptime
  ```
  ![image](https://github.com/user-attachments/assets/51cd456f-ad43-4b36-957c-84f6424d2efe)

  시스템에는 8개의 프로세스보다 훨씬 적은 2개의 CPU만 있으므로 시스템의 CPU는 평균 부하 7.81로 심각한 과부하를 받는다.


  ### 3. 각 cpu 사용률 확인.
  ```
  $ mpstat -P ALL 5 1
  ```
  ![image](https://github.com/user-attachments/assets/ae4e62c8-d463-4752-a65a-8d2dcdbb0b89)
  2개의 cpu의 사용률이 최대로 과부하 상태인 것을 알 수 있다.

  ### 4. 각 프로세스 상황을 확인.
  ```
  $ pidstat -u 5 1
  ```
  ![image](https://github.com/user-attachments/assets/e865fa90-0210-475b-a1d7-8a85efbdd8c0)
  8개의 프로세스가 2개의 CPU를 놓고 경합하고 있으며 각 프로세스는 최대 75%의 시간 동안 CPU를 기다리고 있음을 알 수 있다.
  <br>
  CPU의 계산 용량을 초과하는 이러한 프로세스는 궁극적으로 CPU의 과부하로 이어진다.

## 정리
- 높은 평균 부하는 CPU 집약적인 프로세스로 인해 발생할 가능성이 높다.
  - 실제 cpu를 사용중인 프로세스와 CPU를 기다리고 있는 프로세스의 수를 기반으로 계산된다.
  - 프로세스가 리소스를 적게 사용하더라도 CPU나 I/O 리소스를 대기하는 상태가 되면 평균 부하값이 증가한다.

<br>

- 그러나 100개의 I/O 관련 작업을 하는 프로세스들이 각각 작은 디스크 읽기 작업을 수행한다고 하더라도, 이들이 모두 디스크 I/O 작업을 기다리고 있다면, 평균 부하가 증가한다.
  - 따라서 평균 부하가 높다고 해서 반드시 CPU 사용량이 높다는 의미는 아니다.
  - 단순히 CPU 사용률이 높지 않더라도 높은 평균 부하는 I/O병목 또는 디스크 작업이 과도할 때 나타날 수도 있다.

- 높은 부하를 발견하면 mpstat, pidstat 등과 같은 도구를 사용하여 로드 소스를 분석하는 데 도움을 받을 수 있다.
  - 많은 프로세스가 CPU에 경합하게 되면 `%wait`가 높아지며 이 경우 CPU리소스 부족으로 인해 과도한 컨텍스트 스위칭과 성능 저하가 발생할     수 있다.
  - 이러한 상황에서 CPU코어 수와 프로세스 수의 불균형을 조정하는 것이 성능 최적화에 중요한 역할을 한다.

- 평균 부하와 CPU사용률, I/O대기 (iowait) 메모리 사용량, 디스크 성능, 네트워크 성능 등을 함께 관찰하여 부하의 원인을 정확하게 진단하는   것이 중요하다.

높은 부하 상황에서 시스템이 어떻게 반응하는지 미리 실험해 보고, 문제를 진단하는 방법을 익히면 실제 시스템 운영 중에 발생할 수 있는 문제를 보다 신속하게 파악하고 대응할 수 있을 것이다.
---
### 참고 📚
- stress documentation
- sysstat documentation
