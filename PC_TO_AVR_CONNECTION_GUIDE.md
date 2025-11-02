# Leiðbeiningar: Tölva í Harman Kardon Magnara

**Markmið**: Tengja tölvu við Harman Kardon AVR435/230 magnara fyrir hágæða hljóð og fjarstýringu.

**Besta lausnin er að nota tvær aðskildar tengingar:**
1.  **Hljóð**: Stafræn tenging í gegnum XGIMI myndvarpann.
2.  **Fjarstýring**: Með USB-í-RS232 breyti.

---

## Heildar tengimynd

```
┌──────────┐                                  ┌────────────────┐
│          │   HDMI (Mynd + Hljóð)            │                │
│  Tölva   ├──────────────────────────────────►     XGIMI      │
│ (Lenovo) │                                  │    Myndvarpi   │
│          │   USB-A (Fjarstýring)            │                │
└─────┬────┘                                  └───────┬────────┘
      │                                              │
      │                                              │ Ljósleiðari (Hljóð)
      │ USB í RS-232 breytir + RS-232 kapall         │
      │                                              │
      └─────────────────┐                            │
                        ▼                            ▼
                  ┌───────────────────────────────────┐
                  │                                   │
                  │   Harman Kardon AVR435/230        │
                  │         AV Móttakari              │
                  │                                   │
                  └───────────────────┬───────────────┘
                                      │
                                      │ Hátalarakaplar
                                      │
                                      ▼
                              ┌──────────────┐
                              │              │
                              │ 7.1 Hátalarar │
                              │              │
                              └──────────────┘
```

---

## Innkaupalisti

Þú þarft aðeins tvær vörur:

1.  **Ljósleiðari (Optical/Toslink snúra)**
    *   **Tilgangur**: Fyrir hljóð.
    *   **Lengd**: 2-3 metrar.
    *   **Verð**: ~2.000 - 4.000 kr.
    *   **Fæst í**: ELKO, Tölvulistinn, o.fl.

2.  **USB í RS-232 breytir**
    *   **Tilgangur**: Fyrir fjarstýringu.
    *   **Mikilvægt**: Mælt er með breyti með **FTDI** kubbasetti fyrir besta samhæfni við Linux.
    *   **Verð**: ~3.000 - 5.000 kr.
    *   **Fæst í**: Computer.is, Amazon.

---

## Uppsetningarleiðbeiningar

### Hljóðtenging (5 mínútur)

1.  **Tengdu ljósleiðara**:
    *   Annar endinn í **OPTICAL ÚT** á XGIMI myndvarpanum.
    *   Hinn endinn í **OPTICAL INN** á Harman Kardon magnaranum (t.d. inngangur #31).

2.  **Stilltu XGIMI**:
    *   Farðu í `Stillingar` → `Hljóð` → `Hljóðútgangur`.
    *   Veldu **"SPDIF"** eða **"Ytri hátalari"**.
    *   Farðu í `Stillingar` → `Hljóð` → `Úttakssnið`.
    *   Veldu **"Sjálfvirkt"**.

3.  **Stilltu Magnarann**:
    *   Ýttu á **"Input Source Selector"** þar til rétt inntak er valið (þar sem ljósleiðarinn er tengdur).
    *   Ýttu á **"Digital Select"** þar til **"OPTICAL"** birtist á skjánum.

4.  **Stilltu Tölvuna**:
    *   Gakktu úr skugga um að hljóðútgangur tölvunnar sé stilltur á **HDMI**. Þetta er yfirleitt sjálfgefið þegar hún er tengd við myndvarpann.

### Fjarstýringartenging (10 mínútur)

1.  **Tengdu USB í RS-232 breytinn**:
    *   Stingdu USB endanum í laust USB-A tengi á tölvunni.
    *   Tengdu RS-232 endann (DB9) við **RS-232** tengið á Harman Kardon magnaranum.

2.  **Stilltu Tölvuna (Linux)**:
    *   **Bættu notanda í `dialout` hópinn** til að fá leyfi til að nota raðtengið:
        ```bash
        sudo usermod -aG dialout $USER
        ```
        (Þú gætir þurft að endurræsa tölvuna).
    *   **Settu upp Python serial safnið**:
        ```bash
        pip install pyserial
        ```

3.  **Prófaðu fjarstýringuna**:
    *   Notaðu `avr-control.py` skriftuna til að prófa tenginguna:
        ```bash
        cd ~/fedora-setup/scripts
        ./avr-control.py on
        ```
    *   Magnarinn ætti að kveikja á sér.

---

## Bilanaleit

*   **Ekkert hljóð?**
    *   Athugaðu hvort kveikt sé á öllum tækjum.
    *   Gakktu úr skugga um að rétt inntak sé valið á magnaranum (bæði "Input Source" og "Digital Select").
    *   Skoðaðu hvort rautt ljós sé sýnilegt í enda ljósleiðarans sem tengdur er við magnarann.

*   **Fjarstýring virkar ekki?**
    *   Gakktu úr skugga um að notandinn sé í `dialout` hópnum.
    *   Prófaðu aðra USB tengi á tölvunni.
    *   Staðfestu að `pyserial` sé uppsett.

---

## Tengd skjöl

*   [Ítarlegar tengingar magnara](./AVR435_CONTROLS_AND_CONNECTIONS.md)
*   [Yfirlit yfir búnað](./TECH_ROOM_EQUIPMENT.md)
*   [Innkaupalisti fyrir fjarstýringu](./AVR_RS232_SHOPPING_LIST.md)