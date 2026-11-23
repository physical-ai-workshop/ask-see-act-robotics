# Ask–See–Act Robotics

시너지팀의 로봇팔 제어와 이동 로봇 실험 저장소이다. 첫 실습은 Isaac Sim의 가상 Franka 팔을 MoveIt과 cuMotion에 연결해 목표 자세까지의 경로를 계획하고 실행하는 것입니다.

## Franka cuMotion 실습

기존 팀 서버에 접속하려면 [EC2–Colab 연결 안내](docs/EC2-Colab-setup.md)를 먼저 확인하세요. SSH 터널, Jupyter 실행, Colab 연결과 GPU 확인 절차를 설명합니다.

[Colab 노트북](notebooks/cumotion-franka-ec2.ipynb)에는 기존 팀 EC2 환경을 다시 사용하는 절차가 담겨 있습니다. Colab을 팀 EC2의 Jupyter 런타임에 연결한 뒤 셀을 실행하면 명령은 EC2에서 실행됩니다.

### 준비

- 팀 EC2 접근 권한, 본인의 SSH 키와 현재 EC2 주소
- EC2에 설치된 Isaac Sim, Jupyter, Docker와 ROS 2·cuMotion 작업 공간
- RViz 화면 컨테이너, Franka 수정 설정, ROS 2 UDP 설정, 빌드된 제어 플러그인
- Windows의 Isaac Sim WebRTC Streaming Client와 RViz 원격 화면을 열 브라우저

EC2에 있는 구성과 파일을 먼저 확인하세요. 노트북 아래의 설치·복구 셀은 필요한 항목만 골라 실행합니다.

### 실행 순서

1. 팀 EC2가 실행 중인지 확인합니다. 노트북 첫 셀의 접속 예시와 RViz 화면 준비 셀의 예시 IP를 실제 환경에 맞게 바꿉니다. SSH 터널을 열고 Colab을 EC2의 Jupyter 런타임에 연결합니다.
2. 노트북의 실행 서버 확인과 RViz 화면 준비를 순서대로 진행합니다. Isaac Sim Streaming Client에서 Franka MoveIt 예제 장면을 엽니다.
3. 관절 데이터가 들어오는지 확인한 뒤, 수정 플러그인이 포함된 실습을 시작합니다.
4. RViz에서 Franka의 목표 자세를 하나 지정합니다. 경로가 생성됐는지, Isaac Sim에서 팔이 그 경로를 따라 움직였는지를 각각 확인합니다.

### 확인 상태

기존 팀 노트북에는 Isaac Sim 화면, 관절 데이터 수신, cuMotion 초기화, 제어 플러그인 소스 빌드 성공이 기록돼 있습니다. 수정 플러그인을 적용한 뒤 경로 생성과 팔 동작 실행은 아직 미확인입니다. 이 저장소로 옮기면서 EC2에서 재실행하지 않았습니다.

[공식 cuMotion·MoveIt 튜토리얼](https://nvidia-isaac-ros.github.io/concepts/manipulation/cumotion_moveit/tutorial_isaac_sim.html) · [Colab 외부 런타임 연결 안내](https://research.google.com/colaboratory/local-runtimes.html)
