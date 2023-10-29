# Dataset Configuration
---
## [데이터셋 구성]
---
### 📁구성 현황
- Glass, Can, Plastic 3가지로 구성
- Train과 Val데이터 숫자의 기준 총 Train수의 약 20%의 비율

Train데이터 구성

-Train데이터 경로: 

|데이터 종류|개수[개]|비율[%]|
|--------|---|---|
|Glass|425|33.3|
|Can|425|33.3|
|Plastic|425|33.3|

Val데이터 구성

-Val데이터 경로: 

|데이터 종류|개수[개]|비율[%]|
|--------|---|---|
|Glass|75|33.3|
|Can|75|33.3|
|Plastic|75|33.3|

-이렇게 총 각 클래스별 500개의 숫자로 이루어짐
---
- Detect.py 파일에서 노트북으로 간단한 시험용이므로 모터제어, 로그기록 및 출력에 관한 부분은 빠져있다.
- Train.py 파일을 실행하기위해선 필수 라이브러리를 설치해야한다.
- YOLOv5의 설치 방법은 https://github.com/ultralytics/yolov5 링크를 참고하면된다.


### 모터제어 및 로그 출력 관련 코드
---
``` python
import RPi.GPIO as GPIO #GPIO핀 관련 라이브러리 import
# 라즈베리파이 GPIO 핀 설정
DIR_PIN = 12
CP_PIN = 18
SUB_PIN = 13

# 객체 클래스 이름
class_names = ['Glass', 'Can', 'Plastic']
# 라즈베리파이 GPIO 핀 설정
DIR_PIN=12 #스텝모터의방향제어를위한핀
CP_PIN = 18 # 스텝모터의 펄스 제어(스텝 수)를 위한 핀 SUB_PIN=13 #서보모터제어를위한핀
# GPIO 모드 설정 GPIO.setmode(GPIO.BCM)
# 모터 제어를 위한 GPIO 핀 설정
GPIO.setup(DIR, GPIO.OUT) # 방향 제어 핀 설정 GPIO.setup(CP, GPIO.OUT) # 펄스 제어 핀 설정 GPIO.setup(SUB, GPIO.OUT) # 서보 모터 제어 핀 설정
delay=0.001 #스텝모터의스텝간격(초)
steps_per_revolution = 1600 # 스텝 모터의 1회전 당 필요한 스텝 수
angle_per_step = 360 / steps_per_revolution # 스텝 모터의 1스텝 당 회전 각도 계산
# 서브모터 PWM 설정
pwm = GPIO.PWM(SUB, 50) # 주파수 50Hz pwm.start(2.3) # 초기 각도 설정 (90도) GPIO.setup(SUB, GPIO.IN) #서보 모터 작동 정지
number5 = None #고유번호 초기화
# 객체 클래스 이름, 동시에 고유 번호 매칭 class_names = ['Glass', 'Can', 'Plastic']
# 결과 출력
for c in det[:, 5].unique():
n=(det[:,5]==c).sum() #각클래스별탐지수 number5 = int(c)
print(number5) #감지된 클래스 로그 출력
if number5 == 0: # 유리
- 21 -
      # 시계 방향으로 120도 회전
steps = int(120 / angle_per_step) move_stepper_motor(steps, GPIO.HIGH, delay)
# 서보 모터를 90도로 움직임 GPIO.setup(SUB, GPIO.OUT) #서보모터 작동 pwm.ChangeDutyCycle(8) # 90도 (1.5ms) time.sleep(1.5)
# 서보 모터를 원래 위치로 돌림 pwm.ChangeDutyCycle(2.3) # 0도 (0.5ms) time.sleep(1)
GPIO.setup(SUB, GPIO.IN) #서보모터 작동 정지
# 스텝 모터를 원래 위치로 돌림 move_stepper_motor(steps, GPIO.LOW, delay)
if number5 == 1: # 캔
# 반시계 방향으로 120도 회전
steps = int(120 / angle_per_step) move_stepper_motor(steps, GPIO.LOW, delay)
# 서보 모터를 90도로 움직임 GPIO.setup(SUB, GPIO.OUT) #서보모터 작동 pwm.ChangeDutyCycle(8) # 90도 (1.5ms) time.sleep(1.5)
# 서보 모터를 원위치로 돌림 pwm.ChangeDutyCycle(2.3) # 0도 (0.5ms) time.sleep(1)
GPIO.setup(SUB, GPIO.IN) #서보모터 작동 정지
# 스텝 모터를 원위치로 돌림 move_stepper_motor(steps, GPIO.HIGH, delay)
if number5 == 2: # 플라스틱
# 서보 모터를 90도로 움직임 GPIO.setup(SUB, GPIO.OUT) #서보모터 작동 pwm.ChangeDutyCycle(8) # 90도 (1.5ms) time.sleep(1.5)
  - 22 -

# 서보 모터를 원위치로 돌림 pwm.ChangeDutyCycle(2.3) # 0도 (0.5ms) time.sleep(1)
GPIO.setup(SUB, GPIO.IN) #서보모터 작동 정지

```

### 객체 탐지 알고리즘 함수
---
``` python
def run(
weights=ROOT / 'yolov5s.pt', #학습된 모델의 웨이트 파일 source=ROOT / 'data/images', #실행할 소스 본 코드에선 카메라 사용 data=ROOT / 'data/coco128.yaml', # dataset.yaml 경로
imgsz=(640, 640), # 추론 크기 (높이, 넓이)
conf_thres=0.25, # 신뢰도 임계값
iou_thres=0.45, # NMS IOU 임계값
max_det=1000, # 이미지당 최대 감지 개수
device='', # 그래픽 카드 사용여부 0: no, 1: yes
view_img=False, # 결과 보기 #사용안함
save_txt=False, # *.txt로 결과 저장 #사용안함
save_conf=False, # --save-txt 레이블에 신뢰도 저장 사용안함 save_crop=False, # 예측 박스를 자르기하여 저장 사용안함 nosave=False, # 이미지/비디오 저장하지 않음 사용안함 classes=None, # 클래스로 필터링: --class 0 또는 --class 0 2 3 agnostic_nms=False, # 클래스에 상관없는 NMS
augment=False, # 증강된 추론 #사용안함
visualize=False, # 특징 시각화 #사용안함
update=False, # 모든 모델 업데이트 #사용안함
project=ROOT / 'runs/detect', # 결과를 project/name에 저장 name='exp', # 결과를 exp라는 파일명에 저장
exist_ok=False, # 기존의 project/name 허용, 증가하지 않음 사용안함 line_thickness=3, # 경계 상자 두께 (픽셀)
hide_labels=False, # 레이블 숨기기 #사용안함
hide_conf=False, # 신뢰도 숨기기 #사용안함
half=False, # FP16 반정밀도 추론 사용 #사용안함
dnn=False, # ONNX 추론에 OpenCV DNN 사용 사용안함 vid_stride=1, # 비디오 프레임 레이트 스트라이드
```

### 평가 
---
> ![PR_curve](https://user-images.githubusercontent.com/130136862/278864795-67f7f6be-ec4e-4945-98a9-3df4e1955b6e.png)
학습시 PR커브 곡선을 볼수있는데 값을 보면 mAP성능이 64.1%정도로 나오게 되는 것을 알수있다. 
