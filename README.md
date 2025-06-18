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

 c





Code for LCD ALONE

#include <xc.h>
#define _XTAL_FREQ 4000000

#pragma config FOSC = HS       
#pragma config WDTE = OFF      
#pragma config PWRTE = ON      
#pragma config BOREN = ON      
#pragma config LVP = OFF       
#pragma config CPD = OFF       
#pragma config WRT = OFF       
#pragma config CP = OFF        

// LCD Pin definitions
#define RS PORTCbits.RC1
#define EN PORTCbits.RC2
#define D4 PORTCbits.RC3
#define D5 PORTCbits.RC4
#define D6 PORTCbits.RC5
#define D7 PORTCbits.RC6

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
    lcd_cmd(0x02);    // 4-bit mode
    lcd_cmd(0x28);    // 2 lines, 5x8 font
    lcd_cmd(0x0C);    // Display ON, Cursor OFF
    lcd_cmd(0x06);    // Entry mode
    lcd_cmd(0x01);    // Clear display
}

void lcd_print(const char* str) {
    while (*str) lcd_data(*str++);
}

void lcd_set_cursor(unsigned char row, unsigned char col) {
    unsigned char pos = (row == 1) ? 0x80 : 0xC0;
    pos += col;
    lcd_cmd(pos);
}

void main() {
    // Set LCD Pins as Output
    TRISC = 0x00;
    PORTC = 0x00;

    lcd_init();
    lcd_set_cursor(1, 2);
    lcd_print("Hello World!");

    while (1);
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







Full Complete Code



full code


#include <xc.h>
#define _XTAL_FREQ 4000000

// CONFIG
#pragma config FOSC = HS
#pragma config WDTE = OFF
#pragma config PWRTE = ON
#pragma config BOREN = ON
#pragma config LVP = OFF
#pragma config CPD = OFF
#pragma config WRT = OFF
#pragma config CP = OFF

// ====== LCD Pins on PORTC ======
#define RS PORTCbits.RC1
#define EN PORTCbits.RC2
#define D4 PORTCbits.RC3
#define D5 PORTCbits.RC4
#define D6 PORTCbits.RC5
#define D7 PORTCbits.RC6

// ====== LED and PIR ======
#define LED PORTCbits.RC0
#define PIR_SENSOR PORTBbits.RB0

// ====== Keypad Pins ======
#define ROW1 PORTDbits.RD0
#define ROW2 PORTDbits.RD1
#define ROW3 PORTDbits.RD2
#define ROW4 PORTDbits.RD3

#define COL1 PORTDbits.RD4
#define COL2 PORTDbits.RD5
#define COL3 PORTDbits.RD6

// ====== Function Prototypes ======
void init();
void lcd_init();
void lcd_command(unsigned char);
void lcd_data(unsigned char);
void lcd_clear();
void lcd_set_cursor(unsigned char, unsigned char);
void lcd_print(const char*);
void lcd_pulse();
void lcd_send_nibble(unsigned char);
char scan_keypad();
void turnLightOn();
void turnLightOff();
void display_mode(const char* mode);

// ====== Global Variables ======
char mode = 0;       // 0 = PIR, 1 = Keypad
char ledState = 0;   // 0 = OFF, 1 = ON

// ====== MAIN ======
void main() {
    init();
    char key;

    lcd_clear();
    display_mode("PIR Mode");
    __delay_ms(3000);
    lcd_clear();

    while (1) {
        key = scan_keypad();

        if (key == '3') {
            mode = !mode;
            lcd_clear();
            display_mode(mode ? "Keypad Mode" : "PIR Mode");
            __delay_ms(3000);
            lcd_clear();
        }

        if (mode == 0) {
            if (PIR_SENSOR == 1 && ledState == 0)
                turnLightOn();
            else if (PIR_SENSOR == 0 && ledState == 1)
                turnLightOff();
        } else {
            if (key == '1' && ledState == 0)
                turnLightOn();
            else if (key == '2' && ledState == 1)
                turnLightOff();
        }

        __delay_ms(100);
    }
}

// ====== Initialization ======
void init() {
    // LED
    TRISCbits.TRISC0 = 0; LED = 0;

    // LCD Pins
    TRISCbits.TRISC1 = 0;
    TRISCbits.TRISC2 = 0;
    TRISCbits.TRISC3 = 0;
    TRISCbits.TRISC4 = 0;
    TRISCbits.TRISC5 = 0;
    TRISCbits.TRISC6 = 0;

    // PIR
    TRISBbits.TRISB0 = 1;

    // Keypad: Rows = Input, Cols = Output
    TRISDbits.TRISD0 = 1;
    TRISDbits.TRISD1 = 1;
    TRISDbits.TRISD2 = 1;
    TRISDbits.TRISD3 = 1;
    TRISDbits.TRISD4 = 0;
    TRISDbits.TRISD5 = 0;
    TRISDbits.TRISD6 = 0;

    lcd_init();
}

// ====== Keypad ======
char scan_keypad() {
    const char map[4][3] = {
        {'1','2','3'},
        {'4','5','6'},
        {'7','8','9'},
        {'*','0','#'}
    };

    for (int col = 0; col < 3; col++) {
        PORTD |= 0b01110000;

        if (col == 0) COL1 = 0;
        if (col == 1) COL2 = 0;
        if (col == 2) COL3 = 0;

        if (!ROW1) return map[0][col];
        if (!ROW2) return map[1][col];
        if (!ROW3) return map[2][col];
        if (!ROW4) return map[3][col];

        __delay_ms(20);
    }

    return '\0';
}

// ====== LCD ======
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

void lcd_command(unsigned char cmd) {
    RS = 0;
    lcd_send_nibble(cmd >> 4);
    lcd_send_nibble(cmd & 0x0F);
    __delay_ms(10);
}

void lcd_data(unsigned char data) {
    RS = 1;
    lcd_send_nibble(data >> 4);
    lcd_send_nibble(data & 0x0F);
    __delay_ms(2);
}

void lcd_init() {
    __delay_ms(50);
    lcd_command(0x02);
    lcd_command(0x28);
    lcd_command(0x0C);
    lcd_command(0x06);
    lcd_clear();
}

void lcd_clear() {
    lcd_command(0x01);
    __delay_ms(20);
}

void lcd_set_cursor(unsigned char row, unsigned char col) {
    lcd_command((row == 1 ? 0x80 : 0xC0) + col);
}

void lcd_print(const char* str) {
    while (*str) lcd_data(*str++);
}

// ====== Light Control ======
void turnLightOn() {
    LED = 1;
    ledState = 1;
    lcd_clear(); __delay_ms(20);
    lcd_set_cursor(1, 4); // center: 16-char line
    lcd_print("Light ON");
}

void turnLightOff() {
    LED = 0;
    ledState = 0;
    lcd_clear(); __delay_ms(20);
    lcd_set_cursor(1, 3);
    lcd_print("Light OFF");
}

void display_mode(const char* modeText) {
    lcd_clear(); __delay_ms(20);
    lcd_set_cursor(1, 2);
    lcd_print(modeText);
}



-----------------------------------------------------------------------------------------------

New code without the KEYPAD, + adding a buzzer connected to PIR sensor


#include <xc.h>
#define _XTAL_FREQ 20000000

// CONFIG
#pragma config FOSC = HS       // Use external crystal
#pragma config WDTE = OFF      // Disable watchdog
#pragma config PWRTE = ON      // Enable power-up timer
#pragma config BOREN = ON      // Brown-out reset enabled
#pragma config LVP = OFF       // Low voltage programming off
#pragma config CPD = OFF       
#pragma config WRT = OFF       
#pragma config CP = OFF     

#define RS RD0
#define EN RD2

#define PIR3 RD5
#define MAIN_LIGHT RB3
#define BUZZER RB2

#define MODE_BUTTON RA0       // Button to change mode
#define LIGHT_BUTTON RA1      // Button to toggle light in manual mode

unsigned char mode = 1; // 0 = PIR, 1 = Button Mode (start with button mode)
unsigned char ledState = 0;
unsigned char lastModeBtnState = 1;
unsigned char lastLightBtnState = 1;

void lcd_send_nibble(unsigned char nibble) {
    PORTC = (PORTC & 0x0F) | ((nibble & 0x0F) << 4);
    EN = 1; __delay_us(1); EN = 0;
    __delay_us(100);
}

void lcd_data(unsigned char data) {
    RS = 1;
    lcd_send_nibble(data >> 4);
    lcd_send_nibble(data);
    __delay_us(100);
}

void lcd_command(unsigned char command) {
    RS = 0;
    lcd_send_nibble(command >> 4);
    lcd_send_nibble(command);
    if (command == 0x01 || command == 0x02) __delay_ms(2);
}

void lcd_string(const char *string) {
    while (*string) lcd_data(*string++);
}

void lcd_initialize() {
    __delay_ms(20);
    RS = 0;
    lcd_send_nibble(0x03); __delay_ms(5);
    lcd_send_nibble(0x03); __delay_us(100);
    lcd_send_nibble(0x03); __delay_us(100);
    lcd_send_nibble(0x02);
    lcd_command(0x28);
    lcd_command(0x0C);
    lcd_command(0x06);
    lcd_command(0x01);
    __delay_ms(2);
}

void update_lcd_display() {
    lcd_command(0x80); // Line 1
    lcd_string("Mode: ");
    lcd_string(mode ? "Button " : "PIR    ");

    lcd_command(0xC0); // Line 2
    lcd_string("Light: ");
    lcd_string(ledState ? "ON " : "OFF");
    lcd_string("       "); // Padding
}

void turnLightOn() {
    MAIN_LIGHT = 1;
    ledState = 1;
    update_lcd_display();

    if (mode == 0) // PIR mode only
        BUZZER = 1;
    else
        BUZZER = 0;
}

void turnLightOff() {
    MAIN_LIGHT = 0;
    ledState = 0;
    update_lcd_display();
    BUZZER = 0;
}

void main() {
    TRISC = 0x00;
    TRISD = 0x3A;  // RD0/RD2 output, RD5 input
    TRISB = 0x00;
    TRISA = 0xFF;  // RA0 and RA1 as inputs

    PORTC = PORTD = PORTB = PORTA = 0x00;

    lcd_initialize();
    update_lcd_display();

    while (1) {
        // Button 2: Change mode
        if (MODE_BUTTON == 0 && lastModeBtnState == 1) {
            mode = !mode;
            update_lcd_display();
            __delay_ms(300);
        }
        lastModeBtnState = MODE_BUTTON;

        if (mode == 0) {
            // PIR Mode
            if (PIR3 && !ledState)
                turnLightOn();
            else if (!PIR3 && ledState)
                turnLightOff();
        } else {
            // Button Mode
            if (LIGHT_BUTTON == 0 && lastLightBtnState == 1) {
                if (ledState)
                    turnLightOff();
                else
                    turnLightOn();
                __delay_ms(300);
            }
            lastLightBtnState = LIGHT_BUTTON;
        }

        __delay_ms(50);
    }
}
