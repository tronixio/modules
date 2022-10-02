# 74HC595 LED.

## PIC-AS - Chase LED.

```as
; Configuration Registers.
CONFIG FOSC=INTOSC
CONFIG WDTE=OFF
CONFIG PWRTE=OFF
CONFIG MCLRE=ON
CONFIG CP=OFF
CONFIG BOREN=OFF
CONFIG CLKOUTEN=OFF
CONFIG IESO=OFF
CONFIG FCMEN=OFF
CONFIG WRT=OFF
CONFIG PPS1WAY=ON
CONFIG ZCD=OFF
CONFIG PLLEN=OFF
CONFIG STVREN=ON
CONFIG BORV=LO
CONFIG LPBOR=OFF
CONFIG LVP=ON

#include <xc.inc>
; PIC16F1778 - Compile with PIC-AS(v2.40).
; PIC16F1778 - @8MHz Internal Oscillator.
; -preset_vec=0000h, -pstringtext=3FC0h.
; Instruction ~500ns @8MHz.

; HC595 with SPI.

; GPR BANK0.
PSECT cstackBANK0,class=BANK0,space=1,delta=1
u8delay:    DS	1
u8index:    DS	1

; MCU Definitions.
; BANKS.
#define	BANK0   0x0
#define	BANK1   0x1
#define	BANK2   0x2
#define	BANK3   0x3
#define	BANK4   0x4
#define	BANK5   0x5
#define	BANK6   0x6
#define	BANK7   0x7
#define	BANK8   0x8
#define	BANK9   0x9
#define	BANK10  0xA
#define	BANK11  0xB
#define	BANK12  0xC
#define	BANK13  0xD
#define	BANK14  0xE
#define	BANK15  0xF
#define	BANK16  0x10
#define	BANK17  0x11
#define	BANK18  0x12
#define	BANK19  0x13
#define	BANK20  0x14
#define	BANK21  0x15
#define	BANK22  0x16
#define	BANK23  0x17
#define	BANK24  0x18
#define	BANK25  0x19
#define	BANK26  0x1A
#define	BANK27  0x1B
#define	BANK28  0x1C
#define	BANK29  0x1D
#define	BANK30  0x1E
#define	BANK31  0x1F
; SFR STATUS Bits.
#define	C	0x0
#define	Z	0x2
; SFR PPS - MSSP Bits
#define	SSP1CLK	0x21
#define	SSP1DAT	0x23

; User Definitions.
#define CHASE_INDEX_MAX	14
#define	HC595_nOE   LATC, 0x1
#define	HC595_STCP  LATC, 0x2

; Reset Vector.
PSECT reset_vec,class=CODE,space=0,delta=2
resetVector:
    GOTO    main

; Main.
PSECT cinit,class=CODE,space=0,delta=2
main:
    ; MCU Initialization.
    ; Internal Oscillator Settings.
    MOVLB   BANK1
    MOVLW   0b00000000
    MOVWF   OSCTUNE
    MOVLW   0x70
    MOVWF   OSCCON
    BTFSS   HFIOFR
    BRA	    $-1
    ; Ports Settings.
    ; PORT Data Register.
    MOVLB   BANK0
    MOVLW   0b00000000
    MOVWF   PORTA
    MOVLW   0b00000000
    MOVWF   PORTB
    MOVLW   0b00000000
    MOVWF   PORTC
    MOVLW   0b00000000
    MOVWF   PORTE
    ; TRIS Data Direction.
    MOVLB   BANK1
    MOVLW   0b00000000
    MOVWF   TRISA
    MOVLW   0b00000000
    MOVWF   TRISB
    MOVLW   0b00000000
    MOVWF   TRISC
    MOVLW   0b00000000
    MOVWF   TRISE
    ; LATCH Outputs.
    MOVLB   BANK2
    MOVLW   0b00000000
    MOVWF   LATA
    MOVLW   0b00000000
    MOVWF   LATB
    MOVLW   0b00000000
    MOVWF   LATC
    ; ANSEL Analog.
    MOVLB   BANK3
    MOVLW   0b00000000
    MOVWF   ANSELA
    MOVLW   0b00000000
    MOVWF   ANSELB
    MOVLW   0b00000000
    MOVWF   ANSELC
    ; WPU Weak Pull-up.
    MOVLB   BANK4
    MOVLW   0b00000000
    MOVWF   WPUA
    MOVLW   0b00000000
    MOVWF   WPUB
    MOVLW   0b00000000
    MOVWF   WPUC
    MOVLW   0b00000000
    MOVWF   WPUE
    ; ODCON Open-drain.
    MOVLB   BANK5
    MOVLW   0b00000000
    MOVWF   ODCONA
    MOVLW   0b00000000
    MOVWF   ODCONB
    MOVLW   0b00000000
    MOVWF   ODCONC
    ; SRLCON Slew Rate.
    MOVLB   BANK6
    MOVLW   0b11111111
    MOVWF   SLRCONA
    MOVLW   0b11111111
    MOVWF   SLRCONB
    MOVLW   0b11111111
    MOVWF   SLRCONC
    ; INLVL Input Level.
    MOVLB   BANK7
    MOVLW   0b00000000
    MOVWF   INLVLA
    MOVLW   0b00000000
    MOVWF   INLVLB
    MOVLW   0b00000000
    MOVWF   INLVLC
    ; HIDRVB High Drive.
    MOVLB   BANK8
    MOVLW   0b00000000
    MOVWF   HIDRVB
    ; PPS Settings.
    ; PPS Write Enable.
    MOVLB   BANK28
    MOVLW   0x55
    MOVWF   PPSLOCK
    MOVLW   0xAA
    MOVWF   PPSLOCK
    BCF	    PPSLOCK, 0x0
    ; PPS Inputs.
    ; PPS Outputs.
    MOVLB   BANK29
    MOVLW   SSP1CLK
    MOVWF   RC3PPS
    MOVLW   SSP1DAT
    MOVWF   RC4PPS
    ; PPS Settings.
    ; PPS Write Enable.
    MOVLB   BANK28
    MOVLW   0x55
    MOVWF   PPSLOCK
    MOVLW   0xAA
    MOVWF   PPSLOCK
    BSF	    PPSLOCK, 0x0

    ; MSSP Settings.
    ; MSSP1 SPI Master - Mode 0.
    ; Fsck = Fosc/4 - 2MHz @8MHz.
    MOVLB   BANK4
    CLRF    SSP1BUF
    CLRF    SSP1ADD
    CLRF    SSP1MSK
    MOVLW   0b01000000
    MOVWF   SSP1STAT
    CLRF    SSP1CON1
    CLRF    SSP1CON2
    CLRF    SSP1CON3
    ; MSSP1 Enable.
    BSF	    SSPEN

    ; Initialize Variables.
    MOVLB   BANK0
    CLRF    u8index

loop:
    MOVF    u8index, W
    MOVLW   HIGH a8chase + 0x80
    MOVWF   FSR0H
    MOVLW   LOW	a8chase
    ADDWF   u8index, W
    MOVWF   FSR0L
    MOVIW   FSR0++

    CALL    _hc595_write
    CALL    _delay

    MOVLB   BANK0
    INCF    u8index, F
    MOVLW   CHASE_INDEX_MAX
    SUBWF   u8index, W
    BTFSS   STATUS, Z
    BRA	    loop
    CLRF    u8index

    BRA	    loop

; Functions.
; delay = 1 ~390us @8MHz.
; delay = 255 ~98ms @8MHz.
_delay:
    MOVLB   BANK0
    MOVLW   255
    MOVWF   u8delay
    MOVLW   255
    DECFSZ  WREG, F
    BRA	    $-1
    DECFSZ  u8delay, F
    BRA	    $-3
    RETURN

_hc595_write:
    MOVLB   BANK2
    BCF	    HC595_nOE
    MOVLB   BANK4
    MOVWF   SSP1BUF
    BTFSS   BF
    BRA	    $-1
    MOVLB   BANK2
    BSF	    HC595_STCP
    BCF	    HC595_STCP
    RETURN

; PFM Strings.
PSECT	stringtext,class=STRCODE,space=0,delta=2
a8chase:
    DB	0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, \
	0x40, 0x20, 0x10, 0x08, 0x04, 0x02

    END	    resetVector
```

---
DISCLAIMER: THIS CODE IS PROVIDED WITHOUT ANY WARRANTY OR GUARANTEES.
USERS MAY USE THIS CODE FOR DEVELOPMENT AND EXAMPLE PURPOSES ONLY.
AUTHORS ARE NOT RESPONSIBLE FOR ANY ERRORS, OMISSIONS, OR DAMAGES THAT COULD
RESULT FROM USING THIS FIRMWARE IN WHOLE OR IN PART.
