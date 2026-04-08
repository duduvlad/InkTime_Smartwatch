# InkTime_Smartwatch
Proiect EVT - Smartwatch E-Paper Open Source

## 1. Descrierea Proiectului
Acest repository conține livrabilele pentru etapa de **Engineering Validation (EVT)** a proiectului InkTime, un smartwatch open-source și eficient energetic. Proiectul include design-ul hardware (schematic și PCB layout) realizat în Fusion 360, fișierele de fabricație (Gerber, BOM, CPL) și modelarea mecanică 3D a asamblării finale (inclusiv baterie, display și carcasă).

---

## 2. Diagrama Bloc a Sistemului
Mai jos este prezentată arhitectura hardware a smartwatch-ului și interfețele de comunicare dintre module:

    [ USB-C ] ---> [ BQ25180 (Charger) ] ---> [ Baterie LiPo 250mAh ]
                           |                          |
                           v                          v
                [ MAX17048 (Fuel Gauge) ] ----> [ DC/DC RT6160 (3.3V) ]
                           |                          |
                           +-------- (I2C) -----------+
                                                      |
[ BMA421 (IMU) ] <---(I2C)---+                        v
                             |               [ nRF52840 SoC ] <---> [ Antenă BLE 2.4GHz ]
[ DRV2605 (Haptic) ] <-(I2C)-+                        |
                                                      | (SPI)
[ Butoane x3 ] <--------------------------------------+
                                                      |
                                           [ E-Paper 1.54" Display ]

---

## 3. Funcționalitate Hardware
Dispozitivul este proiectat în jurul SoC-ului **nRF52840**, care gestionează logica principală și conectivitatea Bluetooth Low Energy (BLE). Sistemul este compus din următoarele blocuri funcționale:

* **Managementul Puterii (Power):** Încărcarea bateriei LiPo (AKYGA LP502030, 250mAh) se face prin IC-ul `BQ25180` conectat la portul USB-C. Tensiunea bateriei este monitorizată precis de un fuel gauge `MAX17048` via I2C, iar un convertor buck-boost `RT6160` asigură o tensiune stabilă de 3.3V pentru restul sistemului.
* **Interfața cu Utilizatorul:** Afișajul este un ecran E-Paper de 1.54" (comandat prin SPI), ales pentru vizibilitate excelentă și consum minim (necesită curent doar la update). Navigarea se face prin 3 butoane push, iar feedback-ul haptic este generat de un motor de vibrații controlat de driverul `DRV2605`.
* **Senzori:** Un accelerometru `BMA421` (I2C) este folosit pentru monitorizarea mișcării (ex: numărarea pașilor, funcția "raise to wake").

---

## 4. Alocarea Pinilor (Pinout nRF52840)
Următorii pini ai microcontroller-ului nRF52840 au fost utilizați pentru a interfața perifericele:

| Modul / Componentă | Pini nRF52840 | Interfață / Descriere |
| :--- | :--- | :--- |
| **Magistrala I2C** | `SDA`, `SCL` | Comunicare cu BMA421 (Senzor), MAX17048 (Baterie), DRV2605 (Haptic) și BQ25180. Pull-up-uri incluse pe placă. |
| **Display E-Paper**| `MOSI`, `SCK`, `CS` | Interfață SPI (Master) pentru transmiterea imaginii către ecran. |
| **Display Control**| `DC`, `RES`, `BUSY` | Pini de control: Data/Command, Reset hardware și citirea stării de "Busy" a ecranului. |
| **Butoane (x3)** | `UP`, `ENTER`, `DOWN` | Intrări digitale configurate cu pull-up intern, active pe LOW. |
| **Haptic Motor** | `EN / PWM` | Pin digital pentru activarea/controlul driver-ului DRV2605. |
| **Programare** | `SWDIO`, `SWDCLK`, `RESET`| Interfață de debug/programare (rutată la pad-uri de test / conector). |

---

## 5. BOM (Bill of Materials) - Componente Principale

| Componentă | Descriere | Link JLC Parts | Datasheet |
| :--- | :--- | :--- | :--- |
| **nRF52840** | Bluetooth 5 / MCU | [JLC nRF52840](https://jlcpcb.com/parts) | [Nordic Semiconductor](#) |
| **E-Paper 1.54"** | Display E-Ink V2 | - | [Waveshare WSH-12561](#) |
| **Baterie LiPo** | AKYGA LP502030 (250mAh)| - | [AKYGA Datasheet](#) |
| **BQ25180** | Battery Charger IC | [JLC BQ25180](https://jlcpcb.com/parts) | [Texas Instruments](#) |
| **MAX17048** | Fuel Gauge (I2C) | [JLC MAX17048](https://jlcpcb.com/parts) | [Maxim Integrated](#) |
| **BMA421** | Accelerometru (IMU) | [JLC BMA421](https://jlcpcb.com/parts) | [Bosch Sensortec](#) |
| **RT6160** | DC/DC Buck-Boost 3.3V | [JLC RT6160](https://jlcpcb.com/parts) | [Richtek](#) |
| **DRV2605** | Haptic Motor Driver | [JLC DRV2605](https://jlcpcb.com/parts) | [Texas Instruments](#) |

*Lista completă a componentelor pasive (rezistențe, condensatoare) și coordonatele de asamblare se află în folderul `/Manufacturing/BOM.bom` și `PickAndPlace.cpl`.*

---

## 6. Design Log & Constrângeri Respectate
Pentru a asigura fiabilitatea produsului și respectarea specificațiilor EVT, s-au aplicat următoarele reguli de proiectare:
1.  **Reguli de Amprentă (Footprints):** Toate rezistențele SMD sunt în capsulă **0201**. Condensatoarele de decuplare (<=100nF) sunt **0201** și au fost plasate cât mai aproape de pinii de alimentare ai IC-urilor. Condensatoarele mai mari (>100nF) sunt **0402**.
2.  **Grosimea Traseelor:** S-a utilizat o lățime (width) de **0.3mm** pentru toate traseele de putere (VBUS, VSYS, 3V3, VBAT), cu subțieri justificate doar la intrarea în capsulele BGA. Traseele de date au minim **0.15mm**. Nu există trasee cu unghiuri drepte (90°).
3.  **Performanță RF (Antena):** Antena de 2.4GHz a fost plasată exclusiv pe marginea plăcii. S-a realizat decupaj complet al planurilor de masă (Top și Bottom) pe toată suprafața de sub antenă, fiind interzisă rutarea oricărui semnal în acea zonă.
4.  **Planuri de Masă (GND):** S-a turnat cupru legat la GND atât pe layer-ul TOP, cât și pe BOTTOM. S-a aplicat *Via Stitching* generos între cele două planuri, cu precădere în jurul circuitului radio (nRF52840), pentru a asigura o cale de retur scurtă și ecranare electromagnetică.
5.  **Layout Mecanic:** Toate componentele sunt amplasate exclusiv pe layer-ul TOP. Conectorul de display, butoanele și mufa USB-C au fost aliniate cu designul carcasei.
6.  **Bateria:** Bateria AKYGA a fost modelată 3D (32.5x21x5.5mm). Din rațiuni de spațiu, s-a renunțat la mufa JST, conexiunea planificându-se prin lipire directă pe Test Pad-urile de VBAT și GND.
7.  **DRC & ERC:** Designul trece fără erori de fișierul de reguli DRC (Design Rules Check) impus. Erorile de "Overlap" la test pad-uri și erorile acceptate "Only INPUT pins on NET" au fost verificate manual și aprobate.

---

## 7. Licență
Acest proiect este lansat sub licența **Apache License 2.0**. (Vedeți fișierul `LICENSE` pentru detalii).
