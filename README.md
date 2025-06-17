# Smart-Lighting-System - code 
 Microprocessors Course Project using PIC16F877A


Code for PIR ALONE

#include <xc.h>
#define _XTAL_FREQ 4000000

// === CONFIG BITS ===
#pragma config FOSC = HS       // Use external crystal
#pragma config WDTE = OFF      // Disable watchdog
#pragma config PWRTE = ON      // Enable power-up timer
#pragma config BOREN = ON      // Brown-out reset enabled
#pragma config LVP = OFF       // Low voltage programming off
#pragma config CPD = OFF       
#pragma config WRT = OFF       
#pragma config CP = OFF        

// === PIN DEFINITIONS ===
#define PIR_INPUT PORTBbits.RB0     // PIR sensor input
#define LED_OUTPUT PORTCbits.RC0    // LED output

void main() {
    // === SETUP ===
    TRISBbits.TRISB0 = 1;  // PIR sensor pin as input
    TRISCbits.TRISC0 = 0;  // LED pin as output
    LED_OUTPUT = 0;        // Make sure LED is initially off

    while (1) {
        if (PIR_INPUT == 1) {
            LED_OUTPUT = 1;  // Turn on LED if motion detected
        } else {
            LED_OUTPUT = 0;  // Turn off LED otherwise
        }

        __delay_ms(100);  // Small delay to stabilize reading
    }
}


LCD+ PIR 2


#include <xc.h>
#define _XTAL_FREQ 4000000

// === CONFIG BITS ===
#pragma config FOSC = HS       
#pragma config WDTE = OFF      
#pragma config PWRTE = ON      
#pragma config BOREN = ON      
#pragma config LVP = OFF       
#pragma config CPD = OFF       
#pragma config WRT = OFF       
#pragma config CP = OFF        

// === I/O DEFINITIONS ===
#define PIR_INPUT PORTBbits.RB0     // RB0 = PIR Sensor
#define LED_OUTPUT PORTCbits.RC0    // RC0 = LED

// LCD Pins (4-bit mode)
#define RS PORTCbits.RC1
#define EN PORTCbits.RC2
#define D4 PORTCbits.RC3
#define D5 PORTCbits.RC4
#define D6 PORTCbits.RC5
#define D7 PORTCbits.RC6

// === Function Prototypes ===
void lcd_init();
void lcd_cmd(unsigned char);
void lcd_data(unsigned char);
void lcd_clear();
void lcd_set_cursor(unsigned char row, unsigned char col);
void lcd_print(const char* str);
void lcd_pulse();
void lcd_send_nibble(unsigned char);
void system_init();
void turnLightOn();
void turnLightOff();

void main() {
    system_init();

    lcd_clear();
    lcd_set_cursor(1, 3);
    lcd_print("System Ready");
    __delay_ms(2000);
    lcd_clear();

    while (1) {
        if (PIR_INPUT == 1) {
            turnLightOn();
        } else {
            turnLightOff();
        }
        __delay_ms(100);
    }
}

// === Initialization ===
void system_init() {
    TRISBbits.TRISB0 = 1;     // PIR sensor input
    TRISCbits.TRISC0 = 0;     // LED output
    LED_OUTPUT = 0;

    TRISCbits.TRISC1 = 0;     // RS
    TRISCbits.TRISC2 = 0;     // EN
    TRISCbits.TRISC3 = 0;     // D4
    TRISCbits.TRISC4 = 0;     // D5
    TRISCbits.TRISC5 = 0;     // D6
    TRISCbits.TRISC6 = 0;     // D7

    lcd_init();
}

// === LCD Functions ===
void lcd_pulse() {
    EN = 1; __delay_us(200); EN = 0;
}

void lcd_send_nibble(unsigned char nibble) {
    D4 = (nibble >> 0) & 1;
    D5 = (nibble >> 1) & 1;
    D6 = (nibble >> 2) & 1;
    D7 = (nibble >> 3) & 1;
    lcd_pulse();
}

void lcd_cmd(unsigned char cmd) {
    RS = 0;
    lcd_send_nibble(cmd >> 4);
    lcd_send_nibble(cmd & 0x0F);
    __delay_ms(2);
}

void lcd_data(unsigned char data) {
    RS = 1;
    lcd_send_nibble(data >> 4);
    lcd_send_nibble(data & 0x0F);
    __delay_ms(2);
}

void lcd_init() {
    __delay_ms(20);
    lcd_cmd(0x02);  // 4-bit mode
    lcd_cmd(0x28);  // 2 lines, 5x8 font
    lcd_cmd(0x0C);  // Display ON, cursor OFF
    lcd_cmd(0x06);  // Entry mode
    lcd_clear();
}

void lcd_clear() {
    lcd_cmd(0x01);
    __delay_ms(2);
}

void lcd_set_cursor(unsigned char row, unsigned char col) {
    unsigned char pos = (row == 1) ? 0x80 : 0xC0;
    pos += col;
    lcd_cmd(pos);
}

void lcd_print(const char* str) {
    while (*str) lcd_data(*str++);
}

// === Light Control Logic ===
void turnLightOn() {
    if (LED_OUTPUT == 0) {
        LED_OUTPUT = 1;
        lcd_clear();
        lcd_set_cursor(1, 4);  // Centered: (16 - 8) / 2 = 4
        lcd_print("Light ON");
    }
}

void turnLightOff() {
    if (LED_OUTPUT == 1) {
        LED_OUTPUT = 0;
        lcd_clear();
        lcd_set_cursor(1, 4);
        lcd_print("Light OFF");
    }
}
