# Pop plus API 

Pop 은 한백전자의 장비를 제어하는 파이썬 라이브러리입니다. pip를 통해 다운로드 받아 설치가 가능합니다.

## pop-autocar3g 설치

```
from autocar3g import *
```

## 연결 설정 파일 생성

pop-autocar3g 활용에 앞서 연결 설정 파일을 생성합니다. 접속할 브로커의 주소, 카메라 서버 주소, 장비 고유번호 등의 정보를 "product" 파일로 생성하여 자신의 작업공간에 저장하고 해당 정보를 읽어 활용합니다.

**AutoCar3G product 파일 내용 확인**

product 파일 생성에 앞서 AutoCar3G (이하 Autocar) 에 작성되어 있는 내용을 확인해 보겠습니다.

AutoCar의 네트워크 연결은 Chapter1. Overview -> 1-2. Config the environment -> 무선 네트워크 연결 부분을 참고합니다.

네트워크 연결을 먼저 진행한 후 아래 내용을 진행합니다.

1. 터미널 실행
    - `Win + x`를 눌러 표시되는 작업 표시줄 메뉴에서 '터미널' 을 실행합니다.

2. SSH 접속
    - 터미널 창에서 아래와 같이 명령어를 입력합니다.
```sh
ssh soda@<장비 유/무선 IP>
```

3. 다음 명령을 통한 명칭 확인
    - 다음 명령을 입력하여 출력되는 내용중 INSITUTION_NAME 을 확인하여 작성중인 product 파일에 적용합니다.

```sh
cat /etc/product 

ex) 
BROKER_DOMAIN=127.0.0.1
CAMERA_DOMAIN=127.0.0.1
DEVICE_NAME=TB
DEV_NUM=01
INSITUTION_NAME=HBE
```

출력되는 product 파일의 내용은 AutoCar 내부에 저장된 내용이며 출력된 내용의 정보는 다음과 같습니다.

- BROKER_DOMAIN : 접속할 브로커의 주소 입니다.
    - AutoCar는 자체 설치된 브로커를 활용합니다. 따라서 IP 주소가 '127.0.0.1' 로 설정되어 있습니다.
    - **외부에서 접속하는경우에는 AutoCar 의 무선 IP를 입력합니다.**
- CAMERA_DOMAIN : AutoCar 의 Camera 서버 주소 입니다.
    - **외부에서 접속하는경우에는 AutoCar 의 무선 IP를 입력합니다.**
- DEVICE_NAME : 장비의 이름으로 AutoCar는 'autocar3g' 으로 설정되어 있습니다.
- DEV_NUM : 장치의 고유 번호로 여러개의 장비가 존재하는 경우에는 이 번호를 중복되지 않게 설정해야 합니다.
- INSITUTION_NAME : 학교 또는 기관의 명칭을 고유 키워드로 활용합니다.

이 정보를 기반으로 product 파일을 **자신의 PC의 작업공간** 에 파일을 생성합니다.

파일생성에 앞서 작업공간을 지정합니다. VSCode 에서는 File -> Open Folder 선택 후 원하는 폴더를 지정합니다. 작업공간을 지정했다면 다음과 같이 **PC 에서 접속하기위한 product 파일** 을 작성합니다.

```
BROKER_DOMAIN=<WLAN IP>
CAMERA_DOMAIN=<WLAN IP>
DEVICE_NAME=autocar3g
DEV_NUM=01
INSITUTION_NAME=HBE
```

product 파일 생성 후 작업공간에 다음 그림과 같이 파일이 존재하는것을 확인 후 프로그램 실습을 진행합니다. 각각의 명칭과 정보를 정확하게 기입해야 이후 제어 프로그램 실행하는 과정에서 정상적인 제어를 진행할 수 있습니다.

## Class Driving

구동체 제어 클래스

- `Driving.throttle` (int) : 장비 throttle 값 설정. -99 ~ 99 범위.  
- `Driving.steering` (float) : 장비 steering 값 설정. -1.0 ~ 1.0 범위

## Class Encoder

엔코더 클래스

- `Encoder.read()` : 엔코더 값 반환. 0~65535 범위

## Class Led 

LED 제어 클래스

- `Led.onoff(front, rear)` : AutoCar 전,후면 LED 제어 클래스. 1 설정 시 켜짐, 0 설정 시 꺼짐
  - ex) `Led.onoff(1,0)` : 전면 LED 켜짐 설정, 후면 LED 꺼짐 설정

## Class Ultrasonic

초음파 센서 클래스

- `Ultrasonic.read()` : AutoCar 전체 부착된 초음파 센서 값 읽기
  - ex) (84,100)

## Class Imu

Imu 센서 클래스

- `Imu.accel()` : 가속도 값 출력
  - ex) (0.01025390625, 0.9326171875, -0.33349609375)
- `Imu.magnetic()` : 지자기 값 출력
  - ex) (699, -5392, -4171)
- `Imu.gyro()` : 자이로 값 출력
  - ex) (0.06103515625, 0.0, 0.0)
- `Imu.euler()` : 오일러 값 출력
  - ex) (109.6710205078125, -0.6207275390625, 7.0587158203125)
- `Imu.quat()` : 쿼터니언 값 출력
  - ex) (0.5686954589301, 0.8210322519761, 0.03814111835858, 0.03219893317444)

## Class Camera

카메라 제어 클래스

- `Camera.SERVER_IP` (str) : 카메라 이미지를 받을 장비의 도메인 주소
- `Camera.start()` : 원격 카메라 클라이언트 동작 실행.  
- `Camera.read()` : 현재 카메라 이미지(np.ndarray 타입) 출력. 클라이언트 동작 미실행 (`start` 함수 미호출) 및 연결 오류 시 `None` 반환

## Class AI

AI 클래스

### AI.Object_Follow

ultralytics yolov8 객체 탐지 AI 추상화 클래스

- `Object_Follow.__init__(camera: Camera)` : 객체 탐지 클래스 생성
  - 카메라 인스턴스 생성 후 파라미터로 전달
- `Object_Follow.load_model(path: str)` : yolov8 모델 로드
  - yolov8 모델 경로 파라미터로 전달
- `Object_Follow.detect(image: object = None)` (List[Dict]) : 객체 추론 후 결과 반환
  - `np.ndarray` 이미지 파라미터로 전달.
  - 파라미터 미전달 시 인스턴스 생성 시에 받았던 카메라 데이터 자동으로 구한 후 전달됨
  - Return List[(key:value)]
    - 'index' : 감지된 객체의 인덱스 (classes.txt 참조)
    - 'label' : 감지된 객체의 지정 이름 (classes.txt 참조)
    - 'x' : 감지된 객체의 2차원 이미지 상 x 위치 값
    - 'y' : 감지된 객체의 2차원 이미지 상 y 위치 값
    - 'size_rate' : 감지된 객체의 객체 사이즈

### AI.Track_Follow_TF

Resnet 차선 감지 AI 추상화 클래스

- `Track_Follow_TF.__init__(camera: Camera)` : 차선 탐지 클래스 생성
  - 카메라 인스턴스 생성 후 파라미터로 전달
- `Track_Follow_TF.load_model(path: str)` : Resnet(h5) 모델 로드
  - Resnet(h5) 모델 경로 파라미터로 전달
- `Track_Follow_TF.run(value: object = None)` (Dict) : 차선 추론 후 결과 반환화
  - `np.ndarray` 이미지 파라미터로 전달.
  - 파라미터 미전달 시 인스턴스 생성 시에 받았던 카메라 데이터 자동으로 구한 후 전달됨
  - Return (key:value)
    - 'x' : 감지된 객체의 2차원 이미지 상 x 위치 값
    - 'y' : 감지된 객체의 2차원 이미지 상 y 위치 값

