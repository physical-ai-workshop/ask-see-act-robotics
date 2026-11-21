# 기존 EC2와 Google Colab 연결 방법

이 문서는 Windows PC에서 기존 팀 EC2에 SSH로 접속하고, EC2의 Jupyter를 Colab의 실행 환경으로 연결하는 절차를 설명합니다. 새 서버 생성이나 Isaac Sim·ROS 설치는 포함하지 않습니다.

Colab에서 코드를 작성하고 실행 버튼을 누르면 실제 코드는 EC2에서 실행됩니다. 로컬 런타임이라는 메뉴 이름을 사용하지만, SSH 포트 전달을 통해 원격 EC2를 실행 환경으로 사용하는 구성입니다.

```text
PC의 Colab 화면
    → PC 127.0.0.1:9000
    → SSH 터널
    → EC2 127.0.0.1:8888의 Jupyter
    → EC2의 Python에서 코드 실행
    → 결과를 Colab에 표시
```

## 1. 준비 사항

- 기존 팀 EC2에 접근 가능한 AWS 계정과 SSH 키 페어 파일
- Windows PowerShell과 OpenSSH의 `ssh` 명령
- EC2의 현재 퍼블릭 IPv4 주소, SSH 사용자 `ubuntu`
- EC2에 설치된 Jupyter: `/home/ubuntu/colab-venv/bin/jupyter`
- 기존 작업 폴더: `/home/ubuntu/isaac-notebooks`
- EC2 보안 그룹의 SSH TCP 22 접근 허용: 접속하는 PC의 현재 공인 IP 기준

키 파일은 각자 권한이 있는 파일을 사용합니다. 아래 예시의 `nvidia-isaac-key.pem`은 기존 환경에서 사용한 파일명이며, 팀원이 가진 키 이름과 경로에 맞게 바꿔야 합니다. 개인 키, Jupyter 토큰, 실제 토큰이 포함된 URL은 저장소에 올리지 않습니다.

Jupyter는 EC2의 `127.0.0.1`에만 바인딩하고 SSH 터널로 접속합니다. 이 연결을 위해 보안 그룹에서 8888·9000 포트를 외부에 열 필요는 없습니다.

## 2. EC2 시작 및 현재 IP 확인

1. AWS 콘솔에서 리전을 **아시아 태평양(서울)**로 선택합니다.
2. **EC2 → 인스턴스**에서 기존 팀 인스턴스를 선택합니다.
3. 중지 상태라면 **인스턴스 상태 → 인스턴스 시작**을 선택합니다.
4. 실행 중 상태와 상태 검사 통과 여부를 확인합니다.
5. 아래 **세부 정보 → 퍼블릭 IPv4 주소**를 복사합니다.

인스턴스를 중지했다가 시작하면 퍼블릭 IP가 바뀔 수 있으므로 매번 확인합니다. 아래의 `<EC2_PUBLIC_IP>`는 현재 주소로 바꿉니다. `인스턴스 종료`는 서버 삭제이므로 중지와 구분합니다.

## 3. Windows에서 SSH 접속 및 포트 전달

Windows에서 **PowerShell**을 열고 다음 명령을 실행합니다. `$ec2Address`에 현재 EC2 주소를 입력합니다.

```powershell
$ec2Address = "<EC2_PUBLIC_IP>"
$keyPath = "$env:USERPROFILE\Downloads\nvidia-isaac-key.pem"

ssh -i "$keyPath" -L 127.0.0.1:9000:127.0.0.1:8888 -o ExitOnForwardFailure=yes "ubuntu@$ec2Address"
```

첫 연결에서 호스트 확인 메시지가 나오면 AWS 콘솔의 **인스턴스 선택 → 연결 → SSH 클라이언트**에서 접속 대상이 맞는지 확인합니다. 필요한 경우 팀 관리자에게 SSH 호스트 키 지문을 확인한 뒤 연결을 승인합니다.

성공하면 `ubuntu@…:~$` 형태의 EC2 터미널 프롬프트가 나타납니다.

| 옵션 | 역할 |
| --- | --- |
| `-i` | SSH 인증에 사용할 개인 키 경로 |
| `127.0.0.1:9000:127.0.0.1:8888` | PC의 9000 포트를 EC2 내부 Jupyter 8888 포트에 연결 |
| `ExitOnForwardFailure=yes` | 로컬 포트 전달 설정에 실패하면 SSH 연결을 종료 |

### 선택: RViz 원격 화면 통로도 함께 열기

나중에 RViz/noVNC를 사용할 때는 위 SSH 명령 대신 다음 명령을 사용합니다. 기존 터널과 동시에 실행하면 같은 로컬 포트가 충돌할 수 있습니다.

```powershell
ssh -i "$keyPath" -L 127.0.0.1:9000:127.0.0.1:8888 -L 127.0.0.1:6080:127.0.0.1:6080 -o ExitOnForwardFailure=yes "ubuntu@$ec2Address"
```

6080 포트 전달은 화면 접속 통로입니다. RViz 화면 컨테이너를 시작하는 작업은 별도로 필요하며, EC2–Colab 연결 검증 단계에서는 필요하지 않습니다.

## 4. EC2에서 Jupyter 실행

아래 명령은 Windows PowerShell 명령이 아니라 **SSH 접속 후 EC2의 `ubuntu@…:~$` 프롬프트에서** 실행합니다.

```bash
/home/ubuntu/colab-venv/bin/jupyter notebook \
  --ip=127.0.0.1 \
  --port=8888 \
  --ServerApp.port_retries=0 \
  --ServerApp.allow_origin=https://colab.research.google.com \
  --ServerApp.allow_credentials=True \
  --ServerApp.root_dir=/home/ubuntu/isaac-notebooks
```

| 설정 | 역할 |
| --- | --- |
| `colab-venv/bin/jupyter` | 기존 전용 Python 환경에 설치된 Jupyter 사용 |
| `--ip=127.0.0.1` | EC2 내부 접속만 수락 |
| `--port=8888` | EC2의 Jupyter 포트 지정 |
| `--ServerApp.port_retries=0` | 8888 사용 중이면 다른 포트로 이동하지 않고 오류 표시 |
| `--ServerApp.allow_origin` | Colab에서 오는 연결 허용 |
| `--ServerApp.allow_credentials=True` | Colab 연결에 자격 증명을 사용할 수 있도록 설정 |
| `--ServerApp.root_dir` | Jupyter 작업 폴더 지정 |

실행 로그에서 `http://127.0.0.1:8888/…?token=…` 형태의 URL을 확인합니다. Jupyter 실행 중 PowerShell 창은 계속 열어둡니다.

## 5. Colab에서 로컬 런타임 연결

1. 브라우저에서 사용할 Colab 노트북을 엽니다.
2. 오른쪽 위 **연결 옆 ▼ → 로컬 런타임에 연결**을 선택합니다.
3. Jupyter 출력 URL의 포트 **8888을 9000으로 바꿉니다.** 호스트는 `127.0.0.1`, 토큰은 출력된 값을 그대로 사용합니다.
4. 변경한 URL을 입력하고 **연결**을 누릅니다.

연결 URL 예시이며 실제 토큰은 본인의 Jupyter 출력에서 확인합니다.

```text
http://127.0.0.1:9000/?token=<JUPYTER_TOKEN>
```

별도로 Jupyter 웹사이트의 로그인 화면을 열 필요는 없습니다. EC2에서 Jupyter를 재시작하면 토큰이 달라질 수 있으므로 현재 실행 로그의 값을 사용합니다.

## 6. Colab 코드가 EC2에서 실행되는지 확인

**Colab → + 코드**로 셀을 추가하고 다음 코드를 실행합니다.

```python
import socket
import sys
import subprocess

print("서버:", socket.gethostname())
print("Python:", sys.executable)

result = subprocess.run(
    ["nvidia-smi", "--query-gpu=name,driver_version,memory.total", "--format=csv"],
    capture_output=True,
    text=True,
)
print(result.stdout + result.stderr)
print("종료 코드:", result.returncode)
```

### 기존 EC2에서 확인한 결과

2026-11-22 사용자 실행 결과를 전달받아 확인했습니다. 문서 작성 과정에서 새 SSH 세션으로 직접 재실행한 결과는 아닙니다.

| 확인 항목 | 결과 |
| --- | --- |
| 실행 서버 | 기존 EC2의 호스트 이름 출력 |
| Python | `/home/ubuntu/colab-venv/bin/python` |
| GPU | NVIDIA L4 |
| 드라이버 | `595.91.07` |
| GPU 메모리 | `23034 MiB` |
| 종료 코드 | `0` |

위 결과로 Colab 코드가 EC2의 Python 환경에서 실행되고 EC2 GPU를 조회할 수 있음을 확인했습니다. 이 검사만으로 Docker 내부 GPU 접근이나 Isaac Sim·cuMotion 실행까지 검증한 것은 아닙니다.

## 7. 자주 발생한 문제

| 증상 | 확인 및 조치 |
| --- | --- |
| `Permission denied (publickey)` | **EC2 → 해당 인스턴스 → 연결 → SSH 클라이언트**에서 사용자 이름과 키 페어를 확인합니다. 이름이 비슷한 다른 `.pem` 파일을 선택하지 않았는지도 확인합니다. |
| `UNPROTECTED PRIVATE KEY FILE` | 키의 Windows 접근 권한이 너무 넓은 상태입니다. **파일 탐색기 → 키 파일 우클릭 → 속성 → 보안 → 고급**에서 권한을 확인하고 팀 관리자와 조정합니다. |
| `Address already in use` 또는 포트 전달 실패 | 기존 SSH 터널이 9000·6080을 이미 사용 중인지 확인합니다. 사용 중인 정상 터널을 유지하거나, 이전 터널을 종료한 뒤 다시 연결합니다. |
| Jupyter가 8888을 사용 중이라고 표시 | Jupyter를 중복 실행하지 않습니다. **새 SSH 접속 창의 EC2 터미널**에서 `/home/ubuntu/colab-venv/bin/jupyter server list`로 기존 서버 URL을 확인합니다. 이 출력에도 토큰이 포함될 수 있습니다. |
| Colab 연결 실패 | SSH 창 유지 여부, 현재 EC2 IP, Jupyter 실행 상태, 연결 URL의 PC 포트 9000과 현재 토큰을 순서대로 확인합니다. |
| 파일·실행 파일을 찾을 수 없음 | 기존 EC2를 선택했는지 확인합니다. 이 문서는 Jupyter와 작업 폴더가 이미 준비된 환경을 전제로 합니다. |
| noVNC 페이지는 열리지만 화면 연결 실패 | SSH 통로와 별도로 `cumotion-desktop` 화면 서버가 실행 중이어야 합니다. Colab 연결 성공과 RViz 연결 성공은 별개입니다. |

## 8. 작업 종료와 다음 접속

공유 EC2를 중지하기 전에 다른 팀원이 사용 중인지 확인하고, 담당자와 중지 시간·범위를 조율합니다. 다른 작업이 진행 중이면 합의 없이 인스턴스를 중지하지 않습니다.

1. **Colab → 파일 → 저장**으로 노트북을 저장합니다.
2. AWS 콘솔에서 **EC2 → 해당 인스턴스 → 인스턴스 상태 → 중지**를 선택합니다.
3. **중지됨**을 확인하고 PowerShell·뷰어를 닫습니다. SSH가 끊어졌다는 것만으로 EC2가 중지된 것은 아닙니다.
4. 다음 작업 때 EC2를 시작하고 현재 IP를 확인한 뒤 SSH → Jupyter → Colab 연결 → 실행 환경 검사를 반복합니다.

현재 환경에는 자동 중지 타이머도 있습니다. EC2 터미널에서 다음 명령으로 남은 시간을 확인할 수 있습니다.

```bash
systemctl list-timers --all isaac-auto-stop.timer --no-pager
```

EC2 중지는 기존 EBS 파일을 보존하지만 EBS 저장 비용 등은 별도로 남을 수 있습니다. Jupyter·Colab 연결을 위해 서버를 재설치할 필요는 없습니다.

## 참고

- [팀 Franka 실습 노트북](../notebooks/cumotion-franka-ec2.ipynb)
- [Google Colab 공식 로컬 런타임·다른 머신 연결 안내](https://research.google.com/colaboratory/local-runtimes.html)
