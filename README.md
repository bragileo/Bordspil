# Borðspil

 Höfundar: Bragi, Hrafnhildur, Ísaella og Natalía.

Whack a mole



Það eru 6 takkar á spjaldi. það kvikna ljós á einum af handahófi og þú átt að reyna að slá hann. Fyrir það fæst eitt stig og mun þetta vera í hringrás í heila mínútu. Markmiðið er að ná sem flestum stigum, maður verður samt að passa sig, því ef slegið er í óupplýstann takka fæst mínus stig! Þú ýtir á bláa takkan til þess að byrja. 

Hátalarinn gerir mismunandi hljóð eftir hvort það sé plús stig eða mínus stig.

![Vector skrá fyrir box](https://github.com/bragileo/Bordspil/blob/main/myndir/bordspil.svg)



kóði
```python
import time, random
from machine import Pin, PWM

buna = PWM(Pin(14), freq=1000, duty=0)

def spila_tonu(tidni, dur):
    buna.freq(tidni)
    buna.duty(512)
    time.sleep_ms(dur)
    buna.duty(0)

leikja_kort = {
    21: 6,
    10: 16,
    18: 5,
    2: 15,
    17: 4,
    12: 7
}

hnappar = {}
ljos = {}
for b, l in leikja_kort.items():
    hnappar[b] = Pin(b, Pin.IN, Pin.PULL_UP)
    ljos[l] = Pin(l, Pin.OUT)
    ljos[l].value(0)

byrja_hnappur = Pin(11, Pin.IN, Pin.PULL_UP)

slagar = 0
misst = 0
nyverandi_ljos = None

def kveikja_af_ljosi():
    global nyverandi_ljos
    if nyverandi_ljos is not None:
        ljos[nyverandi_ljos].value(0)
    nyverandi_ljos = random.choice(list(ljos.keys()))
    ljos[nyverandi_ljos].value(1)
    print("Ljós kveikt á pinna", nyverandi_ljos)

def bua_til_hnapps_callback(pinna_ljos):
    def innri_callback(pin):
        global slagar, misst, nyverandi_ljos
        time.sleep_ms(50)
        if pin.value() == 0:
            if nyverandi_ljos == pinna_ljos:
                slagar += 1
                ljos[pinna_ljos].value(0)
                nyverandi_ljos = None
                spila_tonu(1000, 100)
                print("Hitti!", "Fjöldi hitta:", slagar)
            else:
                misst += 1
                spila_tonu(500, 100)
                print("Mistókst!", "Fjöldi mistaka:", misst)
    return innri_callback

for b, l in leikja_kort.items():
    hnappar[b].irq(trigger=Pin.IRQ_FALLING, handler=bua_til_hnapps_callback(l))

def spila_90s_lag():
    tonar = [(800, 150), (1200, 150), (1000, 150), (1400, 150), (1100, 300)]
    for t, d in tonar:
        spila_tonu(t, d)
        time.sleep_ms(100)

def spila_nintendo_lag():
    tonar = [(700, 100), (900, 100), (1100, 100), (900, 100), (700, 100), (500, 300)]
    for t, d in tonar:
        spila_tonu(t, d)
        time.sleep_ms(50)

def nulleit_leik():
    global slagar, misst, nyverandi_ljos
    slagar = 0
    misst = 0
    if nyverandi_ljos is not None:
        ljos[nyverandi_ljos].value(0)
    nyverandi_ljos = None

def leikhringur():
    global slagar, misst, nyverandi_ljos
    print("Ýttu á BYRJA til að hefja leik.")
    while byrja_hnappur.value() == 1:
        time.sleep(0.1)
    spila_90s_lag()
    print("Leikur hafinn! Þú hefur 60 sekúndur.")
    upphafs_timi = time.time()
    lengd = 60
    while True:
        lidinn = time.time() - upphafs_timi
        eftir = int(lengd - lidinn)
        if eftir < 0:
            eftir = 0
        print("Tími eftir:", eftir, "sek | Hitt:", slagar, "| Mistök:", misst)
        if eftir == 0:
            break
        if nyverandi_ljos is None:
            kveikja_af_ljosi()
        time.sleep(0.1)
    if nyverandi_ljos is not None:
        ljos[nyverandi_ljos].value(0)
    for _ in range(3):
        spila_tonu(800, 150)
        time.sleep(0.1)
    print("Leik Lokið!")
    spila_nintendo_lag()
    print("Niðurstaða:", "Hitt:", slagar, "| Mistök:", misst, "| Stig:", slagar - misst)

leikhringur()
```


[lóðun](https://github.com/bragileo/Bordspil/blob/main/myndir/IMG_2905.HEIC)

[box að utan](https://https://github.com/bragileo/Bordspil/blob/main/myndir/IMG_2911.HEIC)

[box að innan 1](https://github.com/bragileo/Bordspil/blob/main/myndir/IMG_2913.HEIC)

[box að innan 2](https://github.com/bragileo/Bordspil/blob/main/myndir/IMG_2906.HEIC)

[Vídeó af virkni, (klippt til þess að uppfylla stærðartakmörk á vídeói (aðeins 25MB) Fullt vídeó í skilahólfi á innu)](https://github.com/bragileo/Bordspil/blob/main/myndir/myndband-virkni.mp4)


