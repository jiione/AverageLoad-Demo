# 🖥️ 시스템 평균 부하 이해 및 모니터링 🚀
## 0. 개요 🎈
**서버 부하 테스트**는 시스템이 예상되는 트래픽 또는 작업량을 감당할 수 있는지를 사전에 확인하는 절차입니다. 이는 서버의 성능을 미리 측정하고, 실제 운영 환경에서 발생할 수 있는 문제를 예방하기 위해 필요합니다.

결국, 서버의 평균 부하와 부하 테스트는 서버가 최적의 성능을 발휘하도록 보장하는 중요한 방법이며, 시스템 관리와 확장 계획 수립에 필수적인 요소입니다.

---

## 1. 평균 부하 설명 📊

**평균 부하**은 특정 시간 동안 **실행 가능**(CPU를 사용 중이거나 대기 중인 프로세스) 또는 **인터럽트 불가**(I/O 대기 중인 프로세스) 상태에 있는 프로세스들의 평균 숫자를 나타냅니다.

- **1분 평균**: 지난 1분간의 평균 부하을 나타냅니다.
- **5분 평균**: 지난 5분간의 평균 부하을 나타냅니다.
- **15분 평균**: 지난 15분간의 평균 부하을 나타냅니다.

예를 들어:

```
$ uptime
02:34:03 up 2 days, 20:14, 1 user, load average: 0.63, 0.83, 0.88
```

위 명령어는 다음과 같은 정보를 보여줍니다:
- 현재 시간: **02:34:03**
- 시스템 가동 시간: **2일 20시간 14분**
- 접속 중인 사용자 수: **1명**
- 평균 부하: **0.63 (1분), 0.83 (5분), 0.88 (15분)**

---

## 2. 적절한 평균 부하란? 🤔

**적절한 평균 부하**는 시스템의 CPU 수에 따라 달라집니다. 예를 들어, 시스템에 2개의 CPU가 있다면:
- 평균 부하이 **2**라면 CPU가 완전히 사용되고 있다는 의미입니다.
- 평균 부하이 **2**를 초과하면 CPU 과부하 상태임을 의미합니다.
- CPU 수는 다음 명령어로 확인할 수 있습니다:

```
$ grep 'model name' /proc/cpuinfo | wc -l
```

---

## 3. 평균 부하 vs. CPU 사용률 ⚡

- **평균 부하**는 실행 가능 상태 또는 인터럽트 불가 상태의 프로세스 수를 측정합니다.
- **CPU 사용률**은 CPU가 실제로 얼마나 바쁜지를 나타냅니다.

**중요한 차이점**:
- 높은 평균 부하는 CPU 사용률이 높거나, I/O 대기 시간이 길어질 수 있음을 나타냅니다.
- `mpstat`, `pidstat`, `iostat`와 같은 도구를 사용해 로드 증가 원인을 분석할 수 있습니다.

---

## 4. 평균 부하 실습 예제 🧪

### CPU-intensive 프로세스 예제 🖥️

1. `stress` 도구를 사용해 CPU-intensive형 프로세스를 실행합니다:

```
$ stress --cpu 1 --timeout 600
```

2. `uptime` 명령어로 시스템 로드를 모니터링합니다:

```
$ watch -d uptime

Every 2.0s: uptime              servername: Mon Sep 23 14:41:58 2024

14:41:58 up  5:35,  3 users,  load average: 0.25, 0.87, 0.85
```

3. `mpstat`로 CPU 사용률을 분석합니다:

```
$ mpstat -P ALL 5

02:30:45 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice                                                                                      %idle
02:30:51 PM  all    0.78    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00                                                                                      99.22
02:30:51 PM    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00                                                                                     100.00
02:30:51 PM    1  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00                                                                                       0.00
```
- 02:30:45 PM: 측정 시각
- all: 모든 CPU의 평균
- %usr: 사용자 프로세스가 사용하는 CPU 비율
- %idle: CPU가 놀고 있는 비율
- CPU 0: 100% 대기 상태
- CPU 1: 100% 사용 중 (stress 명령어로 CPU 1이 100% 활용됨).
---

### I/O-intensive 프로세스 예제 💾

1. `stress` 도구를 사용해 I/O-intensive 프로세스를 실행합니다:

```
$ stress -i 1 --timeout 600 # 1개의 I/O 작업을 실행하며, 600초(10분) 동안 시스템에 I/O 부하를 가합니다.
```

2. `uptime` 명령어로 시스템 로드를 모니터링합니다:

```
$ watch -d uptime
Every 2.0s: uptime                             servername: Mon Sep 23 14:48:30 2024

 14:48:30 up  5:41,  3 users,  load average: 1.24, 1.04, 0.92
```

3. `mpstat`로 높은 I/O 대기 시간을 확인합니다:

```
$ mpstat -P ALL 5

02:43:43 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
02:43:48 PM  all    0.78    0.00    8.97   25.56    0.00   17.71    0.00    0.00    0.00   46.97
02:43:48 PM    0    0.60    0.00    6.75   18.25    0.00    2.18    0.00    0.00    0.00   72.22
02:43:48 PM    1    1.03    0.00   11.86   35.05    0.00   37.89    0.00    0.00    0.00   14.18

```

4. `pidstat`로 높은 I/O 대기 시간을 유발하는 프로세스를 확인합니다:

```
$ pidstat -u 5

02:47:13 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
02:47:18 PM     0        13    0.00    0.20    0.00    0.00    0.20     0  ksoftirqd/0
02:47:18 PM     0        22    0.00    0.20    0.00    0.00    0.20     1  ksoftirqd/1
02:47:18 PM     0        90    0.00    2.59    0.00    0.00    2.59     0  kworker/0:1H-kblockd
02:47:18 PM     0       105    0.00    6.97    0.00    0.00    6.97     1  kworker/1:1H-kblockd
02:47:18 PM     0       420    0.00    0.20    0.00    0.00    0.20     0  multipathd
02:47:18 PM  1000      1071    0.00    0.20    0.00    0.00    0.20     0  sshd
02:47:18 PM     0     80099    0.00    0.20    0.00    0.00    0.20     1  kworker/1:2-mm_percpu_wq
02:47:18 PM     0     91769    0.00    0.20    0.00    0.00    0.20     0  kworker/0:2-events
02:47:18 PM  1000     97169    0.00    0.20    0.00    0.00    0.20     1  watch
02:47:18 PM     0     99897    0.00    0.20    0.00    0.00    0.20     0  kworker/u4:2-events_freezable_power_
02:47:18 PM  1000    101914    0.00    6.37    0.00    9.56    6.37     1  stress
02:47:18 PM  1000    102970    0.00    0.20    0.00    0.00    0.20     0  bash

```

---

### 다수의 프로세스 실행 예제 (시스템 과부하 시뮬레이션) ⚙️

1. 8개의 프로세스를 실행해 시스템을 과부하 상태로 만듭니다:

```
$ stress -c 8 --timeout 600  # 8개의 CPU 작업을 실행하며, 600초(10분) 동안 시스템에 CPU 부하를 가합니다
```

2. `uptime` 명령어로 시스템 로드를 모니터링합니다:

```
$ uptime

14:49:39 up  5:42,  3 users,  load average: 0.92, 1.01, 0.92

```

3. `pidstat`로 CPU를 대기 중인 프로세스들을 확인합니다:

```
$ pidstat -u 5

02:49:51 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
02:49:56 PM  1000    104633   20.21    0.35    0.00   79.79   20.56     1  stress
02:49:56 PM  1000    104634   21.62    1.05    0.00   59.58   22.67     1  stress
02:49:56 PM  1000    104635   20.91    0.00    0.00   26.54   20.91     1  stress
02:49:56 PM  1000    104636   20.74    0.00    0.00   44.11   20.74     0  stress
02:49:56 PM  1000    104637   18.80    0.70    0.00   39.72   19.51     0  stress
02:49:56 PM  1000    104638   33.04    3.87    0.00   63.27   36.91     0  stress
02:49:56 PM  1000    104639   20.91    0.35    0.00   72.58   21.27     0  stress
02:49:56 PM  1000    104640   34.97    1.58    0.00   63.44   36.56     1  stress
02:49:56 PM  1000    104649    0.18    0.00    0.00    0.00    0.18     1  bash

```

---

## 5. 결론 📝

- **평균 부하**는 시스템 전체 성능을 빠르게 평가할 수 있는 지표입니다.
- 그러나 평균 부하만으로는 병목 지점을 찾을 수 없으므로, `mpstat`, `pidstat` 등의 도구를 사용해 원인을 분석해야 합니다.
- 평균 부하가 갑자기 상승하면 원인을 파악해 시스템이 반응하지 않기 전에 조치를 취하는 것이 중요합니다.
