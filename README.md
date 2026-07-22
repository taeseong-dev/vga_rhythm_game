# FPGA-based VGA Rhythm Game

OV7670 카메라와 VGA를 기반으로 구현한 FPGA 리듬게임입니다.

## Demo

https://github.com/user-attachments/assets/dcfb6514-df86-4189-b6a7-1a21a0237377

## System Architecture

<img src="images/vga_top.png" width="600">

### 주요 모듈

**Receiver (UART)**
- Python으로부터 노트 데이터를 수신
- Main Controller에 노트 정보와 게임 종료 신호를 전달

**Main Controller**
- 게임 state 관리 (IDLE -> SELECT -> READY -> GAMECONT -> DONE)
- 노트 생성 및 이동 제어
- 판정 및 점수 계산

**VGA Cam**
- OV7670 초기 설정 및 영상데이터 수신
- 빨간색 손모양의 Region 정보를 Main Controller로 전달
- 노트 위치와 게임 화면을 VGA에 출력

**Sender (UART)**
- 게임 상태, 버튼 입력, 판정 결과 및 점수 정보를 FIFO에 저장
- UART를 통해 Python으로 전송

## Recevier

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

<img src="images/apb_maincontroller.png" width="900">

**Main Control**
- FSM 관리 및 전체 게임 흐름 제어

- 

**linecounter**
- 노트 생성 및 이동 제어

**GameResult**
- 노트 판정 및 Combo 및 Fever 판단

**score**
- 판정 결과 기반 점수 계산

### Main Control
