# FPGA-based VGA Rhythm Game

OV7670 카메라와 VGA를 기반으로 구현한 FPGA 리듬게임입니다.

## Demo

https://github.com/user-attachments/assets/dcfb6514-df86-4189-b6a7-1a21a0237377

- Language : Verilog, Python
- Tool : Vivado 2020.2, Pygame
- Board : Basys3
- Camera : OV7670

## System Architecture

<img src="images/vga_top.png" width="800">

1. Python에서 노트 데이터를 UART로 전송
2. FPGA가 노트를 생성하여 VGA에 출력
3. 카메라로 Region을 검출
4. 노트와 Region을 비교하여 판정
5. 점수와 결과를 Python으로 전송

### 주요 모듈

**Receiver (UART)**
- Python으로부터 노트 데이터를 수신
- Main Controller에 노트 정보와 게임 종료 신호를 전달

**Main Controller**
- 게임 state 관리
- 노트 생성 및 이동 제어
- 판정 및 점수 계산

**VGA Cam**
- OV7670 초기 설정 및 영상데이터 수신
- 빨간색 손모양의 Region 정보를 Main Controller로 전달
- 노트 위치와 게임 화면을 VGA에 출력

**Sender (UART)**
- 게임 정보를 UART 전송 패킷으로 생성
- FIFO를 통해 UART로 전송

## Receiver

### Block Diagram

<img src="images/vga_receiver.png" width="400">

### Output Signals

- **note_start**
  - 새로운 노트 생성을 Main Controller에 요청하는 신호

- **lane_data[3:0]**
  - 생성할 노트의 Lane 정보를 전달하는 신호
  - 예) `4'b0100` : 왼쪽에서 두 번째 Lane에 노트 생성

- **game_done**
  - 게임 종료를 Main Controller에 알리는 신호
  - 종료 데이터(`4'hF`)를 수신하면 출력
 
## Main Controller

### Block Diagram

<img src="images/apb_maincontroller.png" width="1000">

#### Main Control

- 게임의 전체 흐름을 제어하는 FSM

<img src="images/vga_maincontrol_fsm.png" width="600">

- **IDLE**
  - 게임 시작 전 대기

- **SELECT**
  - 플레이할 음악 선택

- **READY**
  - 게임 초기화 및 시작 준비

- **GAME_CONT**
  - 노트 생성, 판정 및 게임 진행

- **CAPTURE**
  - 게임 종료 후 카메라 영상 캡처

- **DONE**
  - 결과 화면 출력 및 게임 종료



#### **Line count**

<img src="images/vga_linecount_bd.png" width="600">

- `note_start` 신호 입력 시 노트 생성
- Vertical Sync마다 line count + 3
- 최대 16개의 노트 동시 관리
- 화면을 벗어난 노트 비활성화

##### **GameResult, Score**

<img src="images/vga_game_bd.png" width="800">

###### GameResult

- 노트 위치와 카메라 입력을 비교하여 Perfect, Good, Miss 판정
- 콤보 및 Fever 상태 생성
- 판정 결과를 Score 모듈에 전달

###### Score

- Perfect, Good 판정에 따른 기본 점수 계산
- Fever 상태에서 점수 2배 적용
- 콤보 종료 시 추가 보너스 점수 계산

## VGAcam

### Block Diagram

<img src="images/vga_cam_bd.png">

#### Camera Interface

<img src="images/vga_ov7670.png" width = "500">

##### VGA_Decoder

- VGA 출력을 위한 Pixel Clock 생성
- VGA의 HSYNC, VSYNC 신호 생성
- x_pixel, y_pixel 및 DE신호 생성

##### OV7670_SCCB_Controller

- OV7670 초기화 및 레지스터 설정을 위한 SCCB 통신 제어 (I2C)
- 설정 데이터를 순차적으로 전송하여 카메라 동작 환경 구성

##### OV7670_Mem_Controller

- OV7670에서 수신한 픽셀 데이터를 Frame Buffer에 저장하기 위한 모듈
- Frame Buffer의 Write Address 및 Data 생성

##### Frame_Buffer

- 카메라 영상 데이터를 저장하는 Frame Buffer
- 쓰기와 읽기를 독립적인 클록으로 처리 (pclk, rclk)

#### Game Processing

<img src="images/vga_rgb_bd.png" width = "800">

##### Region Detector

- 카메라 영상에서 빨간색 마커를 검출
- 마커가 위치한 게임 영역(Region)을 판별

##### Filter Region

- 검출된 Region을 강조하여 카메라 영상에 표시

##### Filter Note

- 게임 노트를 생성하여 화면에 표시

##### Filter Game

- Lane 구분선 및 판정 영역을 화면에 표시

## Sender

<img src="images/vga_sender.png" width = "800">

### Packet Generator

- 게임 정보를 UART 전송 패킷으로 생성
- 게임 상태, 판정 결과, 콤보 및 점수 데이터 구성
- Byte 단위 패킷 데이터 출력

### FIFO

- UART 송신 데이터 저장
- 송신 데이터 버퍼 관리

### UART TX

- UART 프로토콜 기반 직렬 데이터 송신
- FIFO 데이터 -> Serial TX 신호 출력

## Python

### Modules

#### UART Handler

- FPGA와 UART 통신 수행
- FPGA에서 받은 데이터 확인
- 게임 정보를 화면으로 전달

#### Game UI

- 게임 화면 표시
- 게임 진행 상태 관리
- FPGA 상태에 따라 화면 전환
- 게임 결과 표시

#### Config

- UART 통신 설정 관리
- 게임 환경 설정
- 화면 및 상태 정보 관리
