# include <p32xxxx.h>
# include <xc.h>
# include <sys/attribs.h>
# include "config_bits.h" 
//# include "pwm.h"
//# include "lcd.h"
//# include "bluetooth.h"
//# include "motorcontrol.h"
//# include "ultrasound.h"
//# include "led.h"

int flag = 0, count = 0;
void __ISR (19, IPL7SRS) INT4Interrupt(void) //interrupt for button to toggle between perimeter mode and area mode
{
   flag ^= 1;
   count++;
   IFS0bits.INT4IF=0;
}

void delay(int a)
{
   int i,j;
   for (i=0; i<a;i++)
   {
      for (j=0; j<10000;j++);
   }
}
void initLCD(void){
   PMCON=0x83BF; // enable PMP, long wait
   PMMODE=0x03FF; //master mode 
   PMAEN=0x0001; //PMA0 enabled
   T1CON= 0x8070; //TMR1 sets to 256 Tpb (i.e. 8 us))
   TMR1=0; while (TMR1<5000); //wait for ~35 ms 

   PMADDR = LCDCMD; // command register (ADDR = 0) 
   PMDATA = 0x38; // set: 8-bit interface, 2 lines, 5x7 
   TMR1=0; while (TMR1<10); //set ~50 us second waiting

   PMDATA = 0x0c; // ON, no cursor, no blink 
   TMR1 = 0; while( TMR1<10); // 7 x 7 us = 49 us 
   PMDATA = 0x01; // clear display 
   TMR1 = 0; while( TMR1<1000); // 260x 7 us ~ 1.8 ms 
   PMDATA = 0x06; // increment cursor, no shift 
   TMR1 = 0; while( TMR1<1000); // 260 x 7 us = 1.8 ms 
}

void putsLCD( char *s){ 
   while( *s) 
   putLCD( *s++); 
 }
 
void initServo(void){
   TRISBbits.TRISB8 = 0; // set RPB8R as output 
   TRISAbits.TRISA15 = 0; // set RPA15R as output
   ANSELBbits.ANSB8 = 0; // turn off analog for RPB8R

   RPB8R = 11; // set RPB8R as OC5
   RPA15R = 11; // set RPA15R as OC4

   PR2 = 50000;
   OC4CON = 0x8005;
   OC5CON = 0x8005;
   T2CON = 0x8020;
}

void servoSpeed(float speed){
   int a = (int)13500+(speed*500); // counter-clockwise rotation
   int b = (int)13500-(speed*575); // clockwise rotation
 
 if(speed == 0)
   T2CON = 0X0020;
 else 
 {
   T2CON = 0X8020;
   OC4R = 0; // right wheel - connected to servo 1
   OC5R = 0; // left wheel - connected to servo 0
   OC4RS = b; // right wheel/ b to go forward, 13500 to stop, a to go backward
   OC5RS = a; // left wheel/ a to go forward, 13500 to stop, b to go backward
 }
}
void turn45(void){
   int a = 18500; // counter-clockwise rotation
   servoSpeed(0); 
   T2CON = 0X8020;
   OC4R = 0; // left wheel
   OC5R = 0; // right wheel
   OC4RS = a; // left wheel/ a to go forward, 13500 to stop, b to go backward
   OC5RS = a; // right wheel/ b to go forward, 13500 to stop, a to go backward
   delay(70); // this delay allows vehicle to complete a 45 degree turn
   servoSpeed(0);
}

void servoDrive(void){
   int a = 14500; // counter-clockwise rotation
   int b = 12397; // clockwise rotation at same speed as clockwise rotation
   T2CON = 0X8020;
   OC4R = 0; // right wheel - connected to servo 1
   OC5R = 0; // left wheel - connected to servo 0
   OC4RS = b; // b to go forward, 13500 to stop, a to go backward
   OC5RS = a; // left wheel/ a to go forward, 13500 to stop, b to go backward
}

void servoReverse(void){
   int a = 14500; // counter-clockwise rotation
   int b = 12397; // clockwise rotation at same speed as clockwise rotation
   T2CON = 0X8020;
   OC4R = 0; // right wheel - connected to servo 1
   OC5R = 0; // left wheel - connected to servo 0
   OC4RS = a; // right wheel/ b to go forward, 13500 to stop, a to go backward
   OC5RS = b; // left wheel/ a to go forward, 13500 to stop, b to go backward
}

void initUltrasound(void){
   TRISCbits.TRISC2=0; // set RC2 as the output for the trigger
   LATCbits.LATC2=0;
   TRISCbits.TRISC1-1; // set RC1 as the echo for the ultrasound sensor 
}

void initTracker(void){
   TRISCbits.TRISC3=1; // set RC3 (PMODA pin 7) as input from the LS1
   LATCbits.LATC3=1;
   ANSELGbits.ANSG7 = 0; //turn off analog on PMOD pin8
   LATGbits.LATG7=1; // set RG7 (PMODA pin 8) as input from the LS1
   LATGbits.LATG7=1; 
}
void greenOn(void){
   LATDbits.LATD2 = 0; // turn off red LED
   TRISDbits.TRISD12 = 0; // set green LED to output
   LATDbits.LATD12 = 1; // turn on green LED 
}

void redOn(void){
   LATDbits.LATD12 = 0; // turn off green LED
   TRISDbits.TRISD2 = 0; // set red LED to output
   ANSELDbits.ANSD2 = 0;
   LATDbits.LATD2 = 1; // turn on red LED
}

float measureDistance(void){
   int a;
   float b;
   LATCbits.LATC2 = 1; // turn on trigger signal
   TMR3 = 0;
   
   while(TMR3 < 360); /// hold trigger on for 10 microseconds per the spec of the ultrasound sensor
   
   LATCbits.LATC2=0;
   
   while(!PORTCbits.RC1 && TMR3<50000); // while the echo signal is low, wait. While the timer is less than 50,000, wait.
   // ^^^^^ Break while loop when high echo signal returns or if timer reaches 50,000 
   
   TMR3 = 0; // reset timer when high echo signal comes back - this will count how long the echo signal is high
  
   while(PORTCbits.RC1); //wait while timer counts how long high signal is being returned
   
   a = TMR3; // pass the timer value into variable. value of timer corresponds to how far the object is
   b = 0.00074366*a; // multiple of a needed to convert timer value into distance (inches)
   return b; 
 }
 
void followPerimeter(void){
   int a = 14500; // counter-clockwise rotation
   int b = 12397; // clockwise rotation at same speed as clockwise rotation

   greenOn(); // turn on green LED 
   cmdLCD(0x80 | 0x00); 
   putsLCD(" Scanning ");
   cmdLCD(0x80 | 0x40); 
   putsLCD(" Perimeter... ");
   while((PORTCbits.RC3 || PORTGbits.RG7) && flag == 1) // RC3 = 1st pin, 2nd row of PMOD & connected to LD1/right side; RG7 = 2nd pin, 2nd row of PMODA & connected to LD2/left side.
      servoDrive(); 
   
   // once perimeter is reached, turn 90 degrees
   OC4R = 0; // left wheel
   OC5R = 0; // right wheel
   OC4RS = 13500; // right wheel 
   OC5RS = a; // left wheel 
   delay(300); 
   while (flag==1){ 
   float distance; 
   
   while(PORTCbits.RC3 && PORTGbits.RG7 && flag == 1) // while both line trackers are high turn right until on perimeter
   {
     T2CON = 0X8020;
     OC4R = 0; // left wheel
     OC5R = 0; // right wheel
     OC4RS = 13500; // right wheel/servo 1 - needs to == b to go forward, 13500 to stop, a to go backward 
     OC5RS = a; // left wheel/servo 0 - needs to == a to go forward, 13500 to stop, b to go backward
   }

   distance = measureDistance();
 
   if(distance > 6 && flag == 1){
       while(!PORTCbits.RC3 && PORTGbits.RG7&& flag == 1) //while right line tracker is low and left line tracker is high, turn right toward perimeter
       {
           greenOn(); // turn on green LED 
           distance = measureDistance();

           T2CON = 0X8020;
           OC4R = 0; // left wheel
           OC5R = 0; // right wheel

           OC4RS = 13500; // right wheel/servo 1 - needs to = b to go forward, 13500 to stop, a to go backward 
           OC5RS = a; // left wheel/servo 0 - needs to = a to go forward, 13500 to stop, b to go backward

           while (distance < 6 && flag == 1 && distance != 0.0){
             distance = measureDistance();
             servoSpeed(0);
             redOn(); // turn on red LED
             cmdLCD(0x80 | 0x00); 
             putsLCD("INTRUDER ALERT!");
             cmdLCD(0x80 | 0x40); 
             putsLCD(" ");
           }
       }

     while(PORTCbits.RC3 && !PORTGbits.RG7 && flag == 1) //while left line tracker is low and right is high, turn left toward perimeter
     {
       greenOn(); // turn on green LED 
       distance = measureDistance();
       T2CON = 0X8020;
       OC4R = 0; // left wheel
       OC5R = 0; // right wheel
       OC4RS = b; // right wheel/servo 1 - needs to == b to go forward, 13500 to stop, a to go backward 
       OC5RS = 13500; // left wheel/servo 0 - needs to == a to go forward, 13500 to stop, b to go backward
       while (distance < 6 && flag == 1 && distance != 0.0){
         distance = measureDistance();
         servoSpeed(0);
         redOn(); // turn on red LED
         cmdLCD(0x80 | 0x00); 
         putsLCD("INTRUDER ALERT!");
         cmdLCD(0x80 | 0x40); 
         putsLCD(" ");
       }
     }

     while(!PORTCbits.RC3 && !PORTGbits.RG7 && flag == 1) //while both line trackers are low, drive straight
     {
       greenOn(); // turn on green LED
       servoDrive();
       distance = measureDistance();
       while (distance < 6 && flag == 1 && distance != 0.0){
         distance = measureDistance();
         servoSpeed(0);
         redOn(); // turn on red LED
         cmdLCD(0x80 | 0x00); 
         putsLCD("INTRUDER ALERT!");
         cmdLCD(0x80 | 0x40); 
         putsLCD(" ");
       }
      }
    }
    else{
     while(distance<6 && flag == 1){
       redOn(); // turn on red LED
       cmdLCD(0x80 | 0x00); 
       putsLCD("INTRUDER ALERT!");
       cmdLCD(0x80 | 0x40); 
       putsLCD(" ");
       servoSpeed(0);
     }
    } 
   } 
}


void findIntruder(void){
   while((PORTCbits.RC3 || PORTGbits.RG7)&& flag == 0) // if either line tracker is high, the boundary has not yet been crossed. Keep driving
      servoDrive();
   if (!PORTCbits.RC3 && !PORTGbits.RG7 && flag == 0) /// when both line trackers are low,boundary has been reached. Stop
   { 
      servoSpeed(0);
      delay(50);
   }  
}


void main(void){
 float a; 
 //configure external interrupt
 TRISA=0xff00;
 PORTA=0;
 TRISFbits.TRISF0=1;
 INT4R=4; 
 IFS0bits.INT4IF=0; // flag
 IPC4bits.INT4IP=7; // priority
 IEC0bits.INT4IE=1; // enable
 INTCON=0x1004;
 asm("ei"); // end of configure external interrupt
 
 initLCD();
 initServo(); 
 initADC();
 initUltrasound(); 
 initTracker(); 
 T3CON = 0x8020; // timer 3 turned on at 1:4 ratio
 
 while(1){ 
   char s[120]; 
   float distance;

   while (flag == 1)
   followPerimeter(); 

   while(flag == 0 && count == 0){

     distance = measureDistance();

     if (distance < 15.0 && distance > 5.0)
     {
       redOn(); // turn on red LED
       cmdLCD(0x80 | 0x00); 
       putsLCD("INTRUDER ALERT!");
       cmdLCD(0x80 | 0x40); 
       putsLCD("Removing Threat ");
       findIntruder();
       cmdLCD(0x80 | 0x00); 
       putsLCD(" Returning ");
       cmdLCD(0x80 | 0x40); 
       putsLCD(" to base ");
       servoReverse(); // travel back to base
       delay(552); // delay required to reverse 16inches 
       servoSpeed(0);
       greenOn(); // turn on green LED
     }
     else if (distance <= 5.0 || distance >= 15.0)
     {
       greenOn(); // turn on green LED
       cmdLCD(0x80 | 0x00); 
       putsLCD(" Scanning for ");
       cmdLCD(0x80 | 0x40); 
       putsLCD(" intruders ");
       turn45(); 
       delay(50);
     }

     }
     if(flag == 0 && count > 0){
       turn45();
       turn45();
       servoDrive();
       delay(550);
       count = 0;
     }
     delay(50); 
   }
}
