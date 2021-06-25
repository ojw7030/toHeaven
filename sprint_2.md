- serBot 주행
- blynk 연결을 통해 수동 제어
- 조이스틱(4방향), 방향전환(좌 우), 음성기능
- 움직일 때 현재 속도 표시
- 자율/수동 제어 버튼 -> 제외

<img src = "https://user-images.githubusercontent.com/81665497/123360566-7e975100-d5a8-11eb-8d16-df4d4521dc5d.jpg" width=420 height=640>

####
```
import multiprocessing as mp
import BlynkLib
from pop import Pilot, LiDAR
from gtts import gTTS
import subprocess
import random
import time
import math




# 10.10.13.202

tts = gTTS("b", lang='en') # 영어
tts.save("candy.mp3")

tts = gTTS("감사합니다", lang='ko') # 한국 성우
tts.save("thanks.mp3")

tts = gTTS("빵빵", lang='ja') # 일본 성우
tts.save("horn.mp3")


BLYNK_AUTH = 't3JgvIovkmLmbGC7jRhWEjFIzrA16C8X'
blynk = BlynkLib.Blynk(BLYNK_AUTH) 

x_pos = 0
y_pos = 0
speed = 0
auto = False

bot = Pilot.SerBot()
lidar = LiDAR.Rplidar()
lidar.connect()


@blynk.VIRTUAL_WRITE(0)             # 좌회전
def left_button(n):
    global speed
    if auto == False:
        if int(n[0]) == 1:
            bot.forward(10)
            bot.steering = -1.0
            speed = 10
            #print("왼쪽회전")
        else:
            bot.stop()
            #print("정지")
            speed = 0

@blynk.VIRTUAL_WRITE(1)             # 우회전
def right_button(n):
    global speed
    if auto == False:
        if int(n[0]) == 1:
            bot.forward(10)
            bot.steering = 1.0
            speed = 10
            #print("오른쪽회전")
        else:
            bot.stop()
            #print("정지")
            speed = 0


@blynk.VIRTUAL_WRITE(2)             # 음성 1(b)
def candy(n):
    if int(n[0])==1:
        with subprocess.Popen(['play', 'candy.mp3']) as p:  
            p.wait()

@blynk.VIRTUAL_WRITE(3)             # 음성(감사합니다)
def candy(n):
    if int(n[0])==1:
        with subprocess.Popen(['play', 'thanks.mp3']) as p:  
            p.wait()




@blynk.VIRTUAL_WRITE(5)             # 자율주행-수동조작 전환 ## 현재 안됨
def auto_button(n):
    global auto

    if int(n[0])==1:
        auto = True
        auto_move()
    else:
        bot.stop()
        lidar.stopMotor()
        auto = False
    #print(auto)


@blynk.VIRTUAL_WRITE(6)             # 조이스틱(x축)
def joystick1(n):
    global x_pos
    x_pos = int(n[0])-512
    move()

@blynk.VIRTUAL_WRITE(7)             # 조이스틱(y축)
def joystick2(n):
    global y_pos
    y_pos = int(n[0])-512
    move()

@blynk.VIRTUAL_READ(8)              # 현재 속력 표시
def guage():
    blynk.virtual_write(8,int(speed))


# @blynk.VIRTUAL_READ(10)
# def cam():
    #Blynk.setProperty(V1, "url", "http://my_new_video_url");
    #blynk.setProperty(10, "https://youtu.be/PCz17d87W1E", "https://youtu.be/PCz17d87W1E")


def move():
    global speed
    if auto == False:
        if x_pos**2 >10000 or y_pos**2 >10000:
            if -100<x_pos<100 and y_pos>0:
                degree = 0
            elif -100<x_pos<100 and y_pos<0:
                degree = 180
            else:
                if x_pos > 0:
                    degree = (90 - (math.atan((y_pos/x_pos)) * (180/math.pi)))%360
                else:
                    degree = (90 - (math.atan((y_pos/x_pos)) * (180/math.pi)))%360 + 180

            speed = int(((x_pos**2 + y_pos**2)**(1/2))) /10 *1.1
            bot.move(degree, speed)
            #print ("각도 = %.2f, 속력 = %.2f" %(degree, speed))

        if x_pos**2 < 500 and y_pos**2 < 500:
            speed = 0
            bot.stop()
            #print("정지")
        
        # value = bot.getGyro()    # 회전각 측정
        # print(value)


def auto_move():
    auto = False
    print(auto)
    lidar.startMotor()
    direction = 0

    collision = True

    while collision:
        collision = False
        vectors = lidar.getVectors()
        
        for v in vectors:
            degree = v[0]    # 각도
            distance = v[1]  # 거리

            left_hand = (direction-60)%360   # 360을 넘어서지 않도록
            right_hand = (direction+60)%360

            disc = None
            if left_hand > right_hand:      # 진행방향쪽의 거리만 사용하기위함
                disc = degree >= left_hand or degree <= right_hand
            else:
                disc = degree >= left_hand and degree <= right_hand

            if disc:
                if distance <= 430:
                    collision = True
                    bot.stop()
                    break
        
        if collision:
            direction += 30
            direction %= 360
    
    bot.move(direction, 40)


def impact():                       # 가속도센서 - 충격시 정지 ?
    while True:
        value = bot.getAccel()
        print ((value['x']**2 + value['y']**2)**(1/2))

        if (value['x']**2 + value['y']**2)**(1/2) > 15000:
            bot.stop()
            lidar.stopMotor()
            with subprocess.Popen(['play', 'horn.mp3']) as p:  
                p.wait()

def xy():                           # Gyro센서
    while True:
        value = bot.getGyro()
        #print(value)


def blynk_():
    while True:
        blynk.run()

if __name__ == '__main__':
#     main()
    p1 = mp.Process(target=blynk_)
    p2 = mp.Process(target=impact)
    p3 = mp.Process(target=xy)

    p1.start()
    p2.start()
    p3.start()

    p1.join()
    p2.join()
    p3.join()
