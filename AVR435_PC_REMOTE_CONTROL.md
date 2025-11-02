# Tölvufjarstýring fyrir Harman Kardon AVR435/230

**Stjórnunaraðferð**: RS-232 raðtengisamskipti
**Tölva**: Lenovo IdeaPad Flex 5 16IAU7 (Fedora Linux)
**Magnari**: Harman Kardon AVR435/230
**Samskiptareglur**: Tvíátta RS-232
**Dagsetning**: 1. nóvember 2025

---

## Efnisyfirlit

1. [Yfirlit](#yfirlit)
2. [Vélbúnaðarkröfur](#vélbúnaðarkröfur)
3. [Tæknilýsing RS-232 samskiptareglna](#tæknilýsing-rs-232-samskiptareglna)
4. [Uppsetning tenginga](#uppsetning-tenginga)
5. [Linux stillingar](#linux-stillingar)
6. [Python stýriskrifta](#python-stýriskrifta)
7. [Tiltækar skipanir](#tiltækar-skipanir)
8. [Prófanir og bilanaleit](#prófanir-og-bilanaleit)

---

## Yfirlit

Harman Kardon AVR435/230 styður **tvíátta RS-232 raðtengisamskipti**, sem gerir fulla stjórn frá tölvu kleift, þar á meðal:

- Kveikja/slökkva
- Hljóðstyrksstjórnun
- Val á inntaki
- Val á umgerðarhljóðsstillingu (Surround mode)
- Hljóðnema/afhljóðnema
- Stöðuvöktun (endurgjöf frá magnara)

**Kostir:**
- ✅ Sjálfvirk stjórnun magnara
- ✅ Búa til sérsniðnar skriftur
- ✅ Samþætta við heimasjálfvirkni
- ✅ Engin þörf á beinni sjónlínu (ólíkt innrauðu)
- ✅ Tvíátta - fá stöðuendurgjöf

---

## Vélbúnaðarkröfur

### Nauðsynlegur vélbúnaður

#### 1. USB í RS-232 raðtengisbreytir
Tölvan er ekki með innbyggt RS-232 tengi, svo þú þarft USB breyti.

**Mælt með breytum:**
- **Byggt á Prolific PL2303**: ~$8-12 (góður Linux stuðningur)
- **Byggt á FTDI FT232**: ~$15-25 (besti Linux stuðningurinn)
- **Byggt á CH340**: ~$5-8 (ásættanlegur Linux stuðningur)

**Dæmi um vörur:**
- StarTech ICUSB232V2 (FTDI kubbasett): ~$20
- TRENDnet TU-S9 (Prolific): ~$12
- Sabrent CB-DB9P (Prolific): ~$10

#### 2. RS-232 kapall
**Tegund**: **Beintengdur DB-9 kapall** (EKKI null modem!)
- **Mikilvægt**: AVR435 víxlar tengingum innbyrðis, svo notaðu beintengdan kapal
- **Tengingar**: Pinni 3 (TxD) í Pinna 3, Pinni 5 (GND) í Pinna 5
- **Lengd**: 1-3 metrar (fer eftir fjarlægð milli tölvu og magnara)

**Kostnaður**: ~$8-15

**Heildarkostnaður vélbúnaðar**: ~$15-40

---

## Tæknilýsing RS-232 samskiptareglna

### Samskiptastillingar

| Breytu | Gildi |
|-----------|-------|
| **Baud-hraði** | 38,400 bps |
| **Gagnabitar** | 8 |
| **Parity** | Ekkert |
| **Stöðvunarbitar** | 1 |
| **Flæðistýring** | Engin (óvirk) |
| **Lágmarksbil milli skipana** | 50ms |

### Upplýsingar um samskiptareglur

#### Sendingarsnið

Hver sending samanstendur af:

1. **Auðkenniskóði** (6 bæti ASCII):
   - `PCSEND` - Skipanir frá tölvu til magnara
   - `MPSEND` - Staða frá magnara til tölvu

2. **Gagnategund** (1 bæti):
   - `2` - Fjarstýringarskipun
   - `3` - Stöðuskjáupplýsingar

3. **Lengd** (1 bæti):
   - `4` bæti fyrir skipanir
   - `48` bæti fyrir stöðu

4. **Upplýsingasvið**:
   - Fjögurra bæta sextánskur kóði fyrir skipanir
   - 48 bæti fyrir stöðuskjá

5. **Prófsumma** (2 bæti):
   - XOR af sléttum og oddatölu bætaröðum

#### Dæmi um skipanabyggingu

```
PCSEND 2 4 [CMD1] [CMD2] [CMD3] [CMD4] [CHK1] [CHK2]
```

Þar sem:
- `PCSEND` = Auðkenniskóði
- `2` = Tegund fjarstýringarskipunar
- `4` = Lengd (4 bæti)
- `CMD1-4` = Fjögurra bæta sextánskur skipun
- `CHK1-2` = Prófsummubæti

---

## Uppsetning tenginga

### Líkamleg tenging

```
┌──────────────┐  USB kapall  ┌─────────────────┐  RS-232     ┌──────────────┐
│      Tölva     │─────────────│ USB-í-RS232     │─────────────│  AVR435/230  │
│   (Lenovo)   │             │    Breytir      │  DB-9 kapall │   Magnari    │
└──────────────┘             └─────────────────┘             └──────────────┘
```

### Skref-fyrir-skref tenging

1. **Finndu RS-232 tengi magnarans**:
   - Á bakhlið magnarans, finndu **RS-232** tengið (númer #24)
   - DB-9 karlkyns tengi

2. **Tengdu USB-í-RS232 breytinn**:
   - Stingdu USB breytinum í USB-tengi tölvunnar
   - Athugið: Mun birtast sem `/dev/ttyUSB0` (eða álíka) í Linux

3. **Tengdu RS-232 kapalinn**:
   - Notaðu **beintengdan** DB-9 kapal (EKKI null modem)
   - Tengdu breytinn við RS-232 tengi magnarans

4. **Staðfestu tengingu**:
   ```bash
   # Athugaðu hvort breytir sé greindur
   lsusb | grep -i "serial\|ftdi\|prolific\|ch340"

   # Athugaðu tækjanúmer
   ls -la /dev/ttyUSB*
   ```

---

## Linux stillingar

### Notendaheimildir

Bættu notanda við `dialout` hópinn fyrir aðgang að raðtengi:

```bash
# Bættu notanda við dialout hópinn
sudo usermod -aG dialout $USER

# Staðfestu
groups $USER

# Skráðu þig út og aftur inn til að breytingar taki gildi
```

### Settu upp nauðsynlega pakka

```bash
# Settu upp Python serial safnið
sudo dnf install python3-serial python3-pyserial

# Eða með pip
pip3 install pyserial

# Valfrjálst: Tól fyrir raðtengi
sudo dnf install minicom screen
```

### Prófaðu raðtengið

```bash
# Athugaðu upplýsingar um tæki
dmesg | tail -20  # Eftir að hafa tengt USB breytinn

# Ættir að sjá eitthvað á þessa leið:
# usb 1-1: FTDI USB Serial Device converter now attached to ttyUSB0

# Prófaðu með minicom
minicom -D /dev/ttyUSB0 -b 38400

# Stillingar í minicom:
# Ctrl+A, Z, O (Stillingar)
# Uppsetning raðtengis:
#   E - Bps/Par/Bits: 38400 8N1
#   F - Vélbúnaðarflæðistýring: Nei
#   G - Hugbúnaðarflæðistýring: Nei
```

---

## Python stýriskrifta

### Einföld stýriskrifta

Búðu til `/home/gudro/fedora-setup/scripts/avr-control.py`:

```python
#!/usr/bin/env python3
"""
Harman Kardon AVR435/230 RS-232 stjórnun
Stýrir magnara í gegnum raðtengi frá tölvu
"""

import serial
import time
import sys

class AVR435Controller:
    """Stýrir Harman Kardon AVR435 í gegnum RS-232"""

    def __init__(self, port='/dev/ttyUSB0', baudrate=38400):
        """Frumstilla raðtengingu"""
        self.ser = serial.Serial(
            port=port,
            baudrate=baudrate,
            bytesize=serial.EIGHTBITS,
            parity=serial.PARITY_NONE,
            stopbits=serial.STOPBITS_ONE,
            timeout=1
        )
        time.sleep(0.1)  # Leyfa tengingu að komast á

    def _calculate_checksum(self, data):
        """Reikna XOR prófsummu fyrir skipun"""
        even_xor = 0
        odd_xor = 0
        for i, byte in enumerate(data):
            if i % 2 == 0:
                even_xor ^= byte
            else:
                odd_xor ^= byte
        return bytes([even_xor, odd_xor])

    def send_command(self, cmd_bytes):
        """
        Senda skipun til magnara
        cmd_bytes: 4-bæta skipanaröð
        """
        # Byggja upp heildarsendingu
        recognition = b'PCSEND'
        data_type = bytes([2])  # Fjarstýring
        length = bytes([4])     # 4 bæti
        checksum = self._calculate_checksum(cmd_bytes)

        # Heill pakki
        packet = recognition + data_type + length + cmd_bytes + checksum

        # Senda
        self.ser.write(packet)
        time.sleep(0.05)  # 50ms lágmark milli skipana

        # Lesa svar (ef einhver)
        if self.ser.in_waiting:
            response = self.ser.read(self.ser.in_waiting)
            return response
        return None

    # Þæginda aðferðir fyrir algengar skipanir

    def power_on(self):
        """Kveikja á magnara"""
        self.send_command(bytes([0x80, 0x70, 0xC0, 0x3F]))
        print("Magnari: Kveikt")

    def power_off(self):
        """Slökkva á magnara"""
        self.send_command(bytes([0x80, 0x70, 0xCF, 0x30]))
        print("Magnari: Slökkt")

    def mute_toggle(self):
        """Víxla hljóðnema"""
        self.send_command(bytes([0x80, 0x70, 0xC1, 0x3E]))
        print("Magnari: Víxla hljóðnema")

    def volume_up(self):
        """Hækka hljóðstyrk"""
        self.send_command(bytes([0x80, 0x70, 0xC7, 0x38]))
        print("Magnari: Hljóðstyrkur UPP")

    def volume_down(self):
        """Lækka hljóðstyrk"""
        self.send_command(bytes([0x80, 0x70, 0xC6, 0x39]))
        print("Magnari: Hljóðstyrkur NIÐUR")

    def input_dvd(self):
        """Velja DVD inntak"""
        self.send_command(bytes([0x80, 0x70, 0xD2, 0x2D]))
        print("Magnari: Inntak -> DVD")

    def input_cd(self):
        """Velja CD inntak"""
        self.send_command(bytes([0x80, 0x70, 0xD7, 0x28]))
        print("Magnari: Inntak -> CD")

    def input_video1(self):
        """Velja Video 1 inntak"""
        self.send_command(bytes([0x80, 0x70, 0xD1, 0x2E]))
        print("Magnari: Inntak -> Video 1")

    def input_video2(self):
        """Velja Video 2 inntak"""
        self.send_command(bytes([0x80, 0x70, 0xD0, 0x2F]))
        print("Magnari: Inntak -> Video 2")

    def input_video3(self):
        """Velja Video 3 inntak"""
        self.send_command(bytes([0x80, 0x70, 0xD6, 0x29]))
        print("Magnari: Inntak -> Video 3")

    def close(self):
        """Loka raðtengingu"""
        self.ser.close()

def main():
    """Skipanalínuviðmót"""
    if len(sys.argv) < 2:
        print("Notkun: avr-control.py <skipun>")
        print("\nTiltækar skipanir:")
        print("  on          - Kveikja")
        print("  off         - Slökkva")
        print("  mute        - Víxla hljóðnema")
        print("  vol-up      - Hljóðstyrkur upp")
        print("  vol-down    - Hljóðstyrkur niður")
        print("  dvd         - Velja DVD inntak")
        print("  cd          - Velja CD inntak")
        print("  video1      - Velja Video 1")
        print("  video2      - Velja Video 2")
        print("  video3      - Velja Video 3")
        sys.exit(1)

    try:
        avr = AVR435Controller()
        command = sys.argv[1].lower()

        commands = {
            'on': avr.power_on,
            'off': avr.power_off,
            'mute': avr.mute_toggle,
            'vol-up': avr.volume_up,
            'vol-down': avr.volume_down,
            'dvd': avr.input_dvd,
            'cd': avr.input_cd,
            'video1': avr.input_video1,
            'video2': avr.input_video2,
            'video3': avr.input_video3,
        }

        if command in commands:
            commands[command]()
        else:
            print(f"Óþekkt skipun: {command}")
            sys.exit(1)

        avr.close()

    except serial.SerialException as e:
        print(f"Villa í raðtengi: {e}")
        print("Gakktu úr skugga um að USB-í-RS232 breytir sé tengdur")
        print("Athugaðu: ls /dev/ttyUSB*")
        sys.exit(1)
    except Exception as e:
        print(f"Villa: {e}")
        sys.exit(1)

if __name__ == '__main__':
    main()
```

### Gerðu skriftuna keyranlega

```bash
chmod +x ~/fedora-setup/scripts/avr-control.py
```

### Notkunardæmi

```bash
# Kveikja á magnara
./avr-control.py on

# Hækka hljóðstyrk
./avr-control.py vol-up

# Velja DVD inntak
./avr-control.py dvd

# Víxla hljóðnema
./avr-control.py mute

# Slökkva á magnara
./avr-control.py off
```

---

## Tiltækar skipanir

### Aflskipanir

| Skipun | Sextánskur kóði | Virkni |
|---------|----------|----------|
| Power ON | 80 70 C0 3F | Kveikja á magnara |
| Power OFF | 80 70 CF 30 | Slökkva á magnara |
| Power Toggle | 80 70 CB 34 | Víxla aflstöðu |

### Hljóðstyrksskipunir

| Skipun | Sextánskur kóði | Virkni |
|---------|----------|----------|
| Volume UP | 80 70 C7 38 | Hækka hljóðstyrk |
| Volume DOWN | 80 70 C6 39 | Lækka hljóðstyrk |
| Mute Toggle | 80 70 C1 3E | Víxla hljóðnema |

### Inntaksvalsskipunir

| Skipun | Sextánskur kóði | Virkni |
|---------|----------|----------|
| DVD | 80 70 D2 2D | Velja DVD inntak |
| CD | 80 70 D7 28 | Velja CD inntak |
| Video 1 | 80 70 D1 2E | Velja Video 1 |
| Video 2 | 80 70 D0 2F | Velja Video 2 |
| Video 3 | 80 70 D6 29 | Velja Video 3 |
| Tuner (AM/FM) | 80 70 D3 2C | Velja útvarp |
| Tape | 80 70 D4 2B | Velja segulbandsupptökutæki |

### Umgerðarhljóðsskipunir

| Skipun | Sextánskur kóði | Virkni |
|---------|----------|----------|
| Surround Mode Up | 80 70 C9 36 | Næsta umgerðarhljóðsstilling |
| Surround Mode Down | 80 70 C8 37 | Fyrri umgerðarhljóðsstilling |

### Ítarlegri skipanir

| Skipun | Sextánskur kóði | Virkni |
|---------|----------|----------|
| Sleep | 80 70 E0 1F | Svefntímastillir |
| Delay | 80 70 DD 22 | Stilla seinkun |
| Channel Select | 80 70 DC 23 | Velja rás |
| Test Tone | 80 70 DB 24 | Virkja prófunartón |

---

## Prófanir og bilanaleit

### Fyrstu prófanir

1. **Prófaðu greiningu USB breytis**:
   ```bash
   # Tengdu USB-í-RS232 breytinn
   lsusb
   dmesg | tail -20

   # Ættir að sjá tæki á /dev/ttyUSB0
   ls -la /dev/ttyUSB*
   ```

2. **Prófaðu raðtengisamskipti**:
   ```bash
   # Einföld prófun með Python
   python3 << EOF
   import serial
   ser = serial.Serial('/dev/ttyUSB0', 38400)
   print(f"Tengi: {ser.port}, Baud: {ser.baudrate}")
   ser.close()
   EOF
   ```

3. **Prófaðu stjórnun magnara**:
   ```bash
   # Prófaðu að kveikja
   ./avr-control.py on

   # Bíddu í nokkrar sekúndur, prófaðu síðan hljóðstyrk
   sleep 2
   ./avr-control.py vol-up
   ```

### Algeng vandamál

#### Aðgangur hafnaður villa

```
serial.serialutil.SerialException: [Errno 13] could not open port /dev/ttyUSB0: [Errno 13] Permission denied: '/dev/ttyUSB0'
```

**Lausn**:
```bash
# Bættu notanda við dialout hópinn
sudo usermod -aG dialout $USER

# Skráðu þig út og aftur inn, staðfestu síðan
groups | grep dialout

# Eða notaðu sudo til að prófa
sudo ./avr-control.py on
```

#### Tæki fannst ekki

```
serial.serialutil.SerialException: [Errno 2] could not open port /dev/ttyUSB0: [Errno 2] No such file or directory: '/dev/ttyUSB0'
```

**Lausn**:
```bash
# Athugaðu hvaða tæki var búið til
ls /dev/ttyUSB* /dev/ttyACM*

# Uppfærðu skriftuna til að nota rétt tæki
# Breyttu avr-control.py, breyttu port='/dev/ttyUSB0' í raunverulegt tæki
```

#### Ekkert svar frá magnara

**Gátlisti**:
1. ✅ Kapall er **beintengdur** (EKKI null modem)
2. ✅ Kapall er fasttengdur á báðum endum
3. ✅ Kveikt er á magnara
4. ✅ Baud-hraði er 38400
5. ✅ Notar rétt tæki (/dev/ttyUSB0)
6. ✅ Bíður í 50ms milli skipana

**Afkóðun**:
```bash
# Prófaðu með minicom til að sjá hrá gögn
minicom -D /dev/ttyUSB0 -b 38400

# Sláðu inn skipanir handvirkt til að prófa tengingu
```

---

## Ítarlegri notkun

### Búðu til kerfisþjónustu (sjálfvirk ræsing)

Búðu til `/etc/systemd/system/avr-control.service`:

```ini
[Unit]
Description=AVR435 raðtengisstjórnunarþjónusta
After=network.target

[Service]
Type=simple
User=gudro
WorkingDirectory=/home/gudro/fedora-setup/scripts
ExecStart=/home/gudro/fedora-setup/scripts/avr-control.py on
ExecStop=/home/gudro/fedora-setup/scripts/avr-control.py off
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Virkja:
```bash
sudo systemctl daemon-reload
sudo systemctl enable avr-control.service
sudo systemctl start avr-control.service
```

### Búðu til flýtilykla

Í GNOME Stillingar → Lyklaborð → Sérsniðnir flýtilyklar:

```
Nafn: Magnari hljóðstyrkur upp
Skipun: /home/gudro/fedora-setup/scripts/avr-control.py vol-up
Flýtilykill: Super+F12

Nafn: Magnari hljóðstyrkur niður
Skipun: /home/gudro/fedora-setup/scripts/avr-control.py vol-down
Flýtilykill: Super+F11

Nafn: Magnari hljóðnemi
Skipun: /home/gudro/fedora-setup/scripts/avr-control.py mute
Flýtilykill: Super+M
```

### Vefviðmót (valfrjálst)

Búðu til einfalt Flask vefforrit fyrir stjórnun í vafra:

```python
from flask import Flask, render_template_string
import subprocess

app = Flask(__name__)

HTML = """
<!DOCTYPE html>
<html>
<head><title>Magnarastjórnun</title></head>
<body>
    <h1>AVR435 stjórnun</h1>
    <button onclick="location.href='/cmd/on'">Kveikja</button>
    <button onclick="location.href='/cmd/off'">Slökkva</button>
    <br><br>
    <button onclick="location.href='/cmd/vol-up'">Hljóðstyrkur +</button>
    <button onclick="location.href='/cmd/vol-down'">Hljóðstyrkur -</button>
    <button onclick="location.href='/cmd/mute'">Hljóðnemi</button>
    <br><br>
    <button onclick="location.href='/cmd/dvd'">DVD</button>
    <button onclick="location.href='/cmd/cd'">CD</button>
    <button onclick="location.href='/cmd/video1'">Video 1</button>
</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(HTML)

@app.route('/cmd/<command>')
def control(command):
    subprocess.run(['/home/gudro/fedora-setup/scripts/avr-control.py', command])
    return f"Skipun send: {command}"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## Innkaupalisti

### Nauðsynlegur vélbúnaður

- [ ] **USB-í-RS232 breytir** - ~$10-25
  - Mælt með FTDI-byggðum fyrir besta Linux stuðning
  - StarTech ICUSB232V2 (~$20)
  - Sabrent CB-DB9P (~$10)

- [ ] **Beintengdur DB-9 kapall** - ~$8-15
  - 1-3 metra lengd
  - **EKKI** null modem kapall
  - Karl-í-karl DB-9

### Heildarkostnaður: ~$20-40

---

## Tengd skjöl

- [AVR435_CONTROLS_AND_CONNECTIONS.md](./AVR435_CONTROLS_AND_CONNECTIONS.md) - Ítarleg tilvísun fyrir tengingar
- [PC_TO_AVR_CONNECTION_GUIDE.md](./PC_TO_AVR_CONNECTION_GUIDE.md) - Leiðbeiningar um hljóðtengingu
- [TECH_ROOM_EQUIPMENT.md](./TECH_ROOM_EQUIPMENT.md) - Yfirlit yfir búnað

---

**Skjal búið til**: 1. nóvember 2025
**Samskiptareglur**: Harman Kardon AVR RS-232 (tvíátta)
**Staða**: Tilbúið til útfærslu með USB-í-RS232 breyti
