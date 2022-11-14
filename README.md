#Introduction
Bought an ATOMSTACK A5 Pro Plus 40W from AliExpress
https://www.aliexpress.com/item/1005003652763657.html

It was OK, but the firmware kinda sucks.
Let's crack it open and see what the board is.

----------------

# BOARD

![Board Overview](https://github.com/cram42/ATOMSTACK_Laser_32-A5/raw/main/Images/BoardOverview.jpg)

**"ATOMSTACK Laser 32-A5 V1.0."**

At a glance, unused features:
- WiFi
- SD Card
- HDMI (External Controller)
- Limit Switches
- Dual Y Motors
- Reset (E-Stop?)
- I2C Header
- Probe Servos

----------------

# PROCESSOR

![ESP32](https://github.com/cram42/ATOMSTACK_Laser_32-A5/raw/main/Images/ESP32.jpg)

**"ESP32-WROOM-32U"**

```
> esptool --port COM5 chip_id
    Chip is ESP32-D0WD (revision 1)
    Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
    Crystal is 40MHz
> esptool --port COM5 flash_id
    Manufacturer: 20
    Device: 4017
    Detected flash size: 8MB
```

OK. Let's make a backup.
`> esptool --port COM5 read_flash 0 0x800000 .\Laser_32-A5_v1.0_Factory.bin`

----------------

# DRIVERS

![4988ET](https://github.com/cram42/ATOMSTACK_Laser_32-A5/raw/main/Images/4988ET.jpg)

**"4988ET"**

----------------

# OTHER CHIPS

![74HC595 & HC125](https://github.com/cram42/ATOMSTACK_Laser_32-A5/raw/main/Images/74HC595_HC125.jpg)

## 74HC595

Texas Instruments SNx4HC595
8-bit, serial-in, parallel-out shift register

```
       /-----------\
 QB ---|1 O      16|--- Vcc
 QC ---|2        15|--- QA
 QD ---|3        14|--- Serial Input
 QE ---|4        13|--- Output Enable
 QF ---|5        12|--- Register Clock
 QG ---|6        11|--- Storage Clock
 QH ---|7        10|--- Storage Clear
GND ---|8         9|--- QH'
       \-----------/
```

|  # | Function          | Destination                                      |
| -: | :-                | :-                                               |
|  1 | QB                | 4988 (X) Pin 16 (Step)                           |
|  2 | QC                | 4988 (X) Pin 19 (Dir)                            |
|  3 | QD                | ???                                              |
|  4 | QE                | ???                                              |
|  5 | QF                | 4988 (Y) Pin 16 (Step)                           |
|  6 | QG                | 4988 (Y) Pin 19 (Dir)                            |
|  7 | QH                | HDMI 13                                          |
|  8 | GND               | GND                                              |
|  9 | QH'               | ???                                              |
| 10 | Storage Clear     | ???                                              |
| 11 | Storage Clock     | ESP32 Pin 25 (GPIO16, RXD_2)                     |
| 12 | Register Clock    | ESP32 Pin 27 (GPIO17, TXD_2)                     |
| 13 | Output Enable     | ???                                              |
| 14 | Serial Data Input | ESP32 Pin 42 (GPIO21, VSPI_HD, SDA)              |
| 15 | QA                | 4988 (Both) Pin 2 (Enable) - Pulled up to 3.3V   |
| 16 | Vcc               | 3.3V                                             |

## HC125

Texas Instruments SNx4HC125
Quadruple Buffers with 3-State Outputs

```
                            /---------------\
Channel 1, Output Enable ---|1 O          14|--- Vcc
      Channel 1, Input A ---|2            13|--- Channel 4, Output Enable
     Channel 1, Output Y ---|3            12|--- Channel 4, Input A
Channel 2, Output Enable ---|4            11|--- Channel 4, Output Y
      Channel 2, Input A ---|5            10|--- Channel 3, Output Enable
     Channel 2, Output Y ---|6             9|--- Channel 3, Input A
                     GND ---|7             8|--- Channel 3, Output Y
                            \---------------/
```

|  # | Function                 | Destination                                           |
| -: | :-                       | :-                                                    |
|  1 | Channel 1, Output Enable | GND                                                   |
|  2 | Channel 1, Input A       | ESP32 Pin 34 (GPIO5, V_SPI_CS0, SS)                   |
|  3 | Channel 1, Output Y      | ? 3 mid
|  4 | Channel 2, Output Enable | GND                                                   |
|  5 | Channel 2, Input A       | ESP32 Pin 14 (GPIO25, ADC2_8, RTC GPIO6, DAC 1)       |
|  6 | Channel 2, Output Y      | HDMI 12                                               |
|  7 | GND                      | GND                                                   |
|  8 | Channel 3, Output Y      | HDMI 6                                                |
|  9 | Channel 3, Input A       | ESP32 Pin 25 (GPIO16, RXD_2) [74HC595 Storage Clock]  |
| 10 | Channel 3, Output Enable | GND                                                   |
| 11 | Channel 4, Output Y      | HDMI 3                                                |
| 12 | Channel 4, Input A       | ESP32 Pin 27 (GPIO17, TXD_2) [74HC595 Register Clock] |
| 13 | Channel 4, Output Enable | GND                                                   |
| 14 | Vcc                      | 3.3V                                                  |

## TA5

Texas Instruments SN74LVC1T45
Single-bit noninverting bus transceiver

![TA5 1](https://github.com/cram42/ATOMSTACK_Laser_32-A5/raw/main/Images/TA5_1.jpg)
![TA5 2](https://github.com/cram42/ATOMSTACK_Laser_32-A5/raw/main/Images/TA5_2.jpg)

|  # | Function | Destination                                   |
| -: | :-       | :-                                            |
|  1 | Vcca     | 5V                                            |
|  2 | GND      | GND                                           |
|  3 | A        | TTL Header +                                  |
|  4 | B        | ESP32 Pin 39 (GPIO22, V_SPI_WP, SCL, RTS 0)   |
|  5 | DIR      | ???                                           |
|  6 | Vccb     | 3.3V                                          |

----------------

# BOARD PINS

| Location | Function           |  # | Notes | Destination                                                                  |
| :-       | :-                 | -: | :-    | :-                                                                           |
| Probe Header (Left)  |        |  S | Pull Up/Down 3.3V/GND | ESP32 Pin 22 (GPIO2, ADC2_2, HSPI_WP0, Touch2, RTC GPIO12)   |
| Probe Header (Right) |        |  S |       | ESP32 Pin 10 (GPIO34, ADC1_6, RTC GPIO4, Input Only)                         |
| Blue Y Header | Y Endstop     |  S |       | ESP32 Pin 11 (GPIO35, ADC1_7, RTC GPIO5, Input Only)                         |
| Red X Header  | X Endstop     |  S |       | ESP32 Pin 5 (GPIO36, ADC1_0, SensVP, RTC GPIO0, Input Only)                  |
| I2C Header    |               |  1 | (Top) | ESP32 Pin 24 (GPIO4, ADC2_0, HSPI_HD, Touch0, RTC GPIO10)                    |
| I2C Header    |               |  2 |       | ESP32 Pin 23 (GPIO0, ADC2_1, CLK1, Touch1, RTC GPIO11)                       |
| I2C Header    |               |  3 |       | GND                                                                          |
| I2C Header    |               |  4 |       | 3.3V                                                                         |
| TTL Header    | Laser Control |  + |       | TA5 Bus Tranceiver Pin 3 (A) @ 5V                                            |
| SD            | Card Detect   |    |       | LED                                                                          |
| SD            | DAT1          |  8 |       | NC                                                                           |
| SD            | DAT0          |  7 |       | ESP32 Pin 18 (GPIO12, ADC2_5, HSPI_Q, Touch5, RTC GPIO15)                    |
| SD            | VSS           |  6 |       | GND                                                                          |
| SD            | CLK           |  5 |       | ESP32 Pin 17 (GPIO14, ADC2_6, HSPI_CLK, Touch6, RTC GPIO16)                  |
| SD            | VDD           |  4 |       | 3.3V                                                                         |
| SD            | CMD           |  3 |       | ESP32 Pin 20 (GPIO13, ADC2_4, HSPI_ID, Touch4, RTC GPIO14)                   |
| SD            | CD            |  2 |       | ESP32 Pin 21 (GPIO15, ADC2_3, HSPI_CS0, Touch3, RTC GPIO13)                  |
| SD            | DAT2          |  1 |       | NC                                                                           |
| Reset Header  |               |  L |       | ESP32 Pin 9 (EN)                                                             |
| Reset Header  |               |  R |       | GND                                                                          |
| HDMI          |               |  1 | (Bottom) | ???
| HDMI          |               |  2 |       | ???
| HDMI          |               |  3 |       | HC125 11 (Output 4)
| HDMI          |               |  4 |       | ???
| HDMI          |               |  5 |       | ???
| HDMI          |               |  6 |       | HC125 8 (Output 3)
| HDMI          |               |  7 |       | ???
| HDMI          |               |  8 |       | ???
| HDMI          |               |  9 |       | ???
| HDMI          |               | 10 |       | ???
| HDMI          |               | 11 |       | ???
| HDMI          |               | 12 |       | HC125 6 (Output 2)
| HDMI          |               | 13 |       | 74HC595 7 (QH)
| HDMI          |               | 14 |       | ???
| HDMI          |               | 15 |       | ???
| HDMI          |               | 16 |       | 3.3V
| HDMI          |               | 17 |       | GND
| HDMI          |               | 18 |       | 5V
| HDMI          |               | 19 |       | ???

----------------

# Random Notes

- Step pin on Y driver goes to 74HC595D. So, it appears the board needs to use "I2S Stepper Stream".
- SD Card is running in SPI mode

----------------
