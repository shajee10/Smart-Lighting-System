# Smart-Lighting-System
 Microprocessors Course Project using PIC16F877A


PIR ALONE

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
