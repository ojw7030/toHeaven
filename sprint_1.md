Psd 센서와 Cds 센서를 데이터를 Oled와 Pixel Display로 데이터로 전환하여 시각화하는 알고리즘 구현 Digital signning 에 적용 가능하다고 생각됨

Psd 거리값에 따라 Oled 애니메이션의 화면밝기를 조절하려 했으나 Oled의 화면이 작고 색감이 단순하여 Pixel Display로 구현하기로 했음.

Cds 센서로 조도 데이터를 수집하여 조도 수준에 따라서Oled에 레벨을 표시하고 Pixel Display에 다른 색깔과 밝기로 차별화하여 시각화를 구현하였음

=========================================================================================================================

from pop import PixelDisplay, Psd, Switches, Oled, Cds, Leds import time, BlynkLib

blynk = BlynkLib.Blynk('biP_PjOT8IwSz8sHBI9_X1024L7el5Og', server = "127.0.0.1", port ="8080")

pixel = PixelDisplay() psd = Psd() cds = Cds() oled = Oled() leds = Leds() ws = Switches() ret = 0.0 state = 0

cds_data = [] # 조도 데이터 리스트

========================Psd ===========================

class ptpsd(Psd): @blynk.VIRTUAL_READ(1) def on_psd(): global psd global pixel

    n = psd.readAverage()
    ret = psd.calcDist(n)
    blynk.virtual_write(1, ret)
    pixel.clear()

    brt = int(12+int(ret))

    pixel.setBrightness(35+int(ret*0.5))

    for x in range(8):
        for y in range(8):
            pixel.setColor(x, y, [20 + int(brt*0.8),brt,brt])
    

    print(ret)
    return ret
======================== Switch ===========================

class ptswitch(Switches): @blynk.VIRTUAL_WRITE(0) def on_off(sta): global state # print(type(sta), sta) if sta[0] == '1': state = 1 elif sta[0] == '0': state = 0

@blynk.VIRTUAL_WRITE(2)
def on_off(sta):
    global state
    # print(type(sta), sta)
    if sta[0] == '3':
        state = 3
    elif sta[0] == '2':
        state = 2
========================Cds =========================== class ptCds(Cds): @blynk.VIRTUAL_READ(3) def on_cds(): global cds n2 = cds.readAverage() blynk.virtual_write(3, n2) return n2

class gos(Cds):

def PD1():
    global pixel

    pixel.setBrightness(90)
    pixel.fill([255,0,0])
    time.sleep(.2)

    pixel.fill([0,0,0])

    # pixel.setBrightness(90)
    # pixel.setColor(4,4,[255,0,0])

    # pixel.fill([0,0,0])

def PD2():
    global pixel

    pixel.setBrightness(70)
    pixel.fill([255,255,0])
    time.sleep(.2)

    pixel.fill([0,0,0])

    # pixel.setBrightness(70)
    # pixel.setColor(4,4,[255,255,0])

    # pixel.fill([0,0,0])


def PD3():
    global pixel

    pixel.setBrightness(50)
    pixel.fill([255,0,255])
    time.sleep(.2)

    pixel.fill([0,0,0])

    # pixel.setBrightness(50)
    # pixel.setColor(4,4,[255,0,255])

    # pixel.fill([0,0,0])

def PD4():
    global pixel

    pixel.setBrightness(30)
    pixel.fill([0,153,0])
    time.sleep(.2)

    pixel.fill([0,0,0])

    # pixel.setBrightness(30)
    # pixel.setColor(4,4,[0,153,0])

    # pixel.fill([0,0,0])

def PD5():
    global pixel

    pixel.setBrightness(10)
    pixel.fill([51,0,255])
    time.sleep(.2)

    pixel.fill([0,0,0])

    # pixel.setBrightness(10)
    # pixel.setColor(4,4,[51,0,255])

    # pixel.fill([0,0,0])

def OL1():
    global oled

    oled.setCursor(10,30)
    oled.print("level 1")
    oled.clearDisplay()
    return

def OL2():
    global oled

    oled.setCursor(10,30)
    oled.print("level 2")
    oled.clearDisplay()
    return

def OL3():
    global oled

    oled.setCursor(10,30)
    oled.print("level 3")
    oled.clearDisplay()
    return

def OL4():
    global oled

    oled.setCursor(10,30)
    oled.print("level 4")
    oled.clearDisplay()
    return

def OL5():
    global oled

    oled.setCursor(10,30)
    oled.print("level 5")
    oled.clearDisplay()
    return

def cds_run():
    global leds
    global cds
    global cds_data
    global state
    
    if state == 3:
        val = cds.readAverage()
        if val < 250:
            gos.OL1(), gos.PD5()
        elif 250 <= val < 500 :
            gos.OL2(), gos.PD4()
        elif 500 <= val < 850 :
            gos.OL3(), gos.PD3()
        elif 850 <= val  < 1300 :
            gos.OL4(), gos.PD2()
        elif val>= 1300 :
            gos.OL5(), gos.PD1()
        print(val)


    return
====================== 실행문 ===========================

while True:

blynk.run()

if state == 1:
    ptpsd.on_psd()
elif state == 0 or state == 2:
    pixel.clear()
elif state == 3:
    gos.cds_run()


# pixel.setBrightness(int(ret*1.2))
# pixel.setColor(4, 4, [int(ret*1.2),0,0])
time.sleep(0.6)
