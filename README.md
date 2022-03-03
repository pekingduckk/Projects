# Sumo Bot
#include <Servo.h>
 
Servo servoLeft;
Servo servoRight;
 
void setup() // Built-in initialization block
{
 pinMode(10, INPUT); pinMode(9, OUTPUT); // Left IR LED & Receiver
 pinMode(3, INPUT); pinMode(2, OUTPUT); // Right IR LED & Receiver
 pinMode(8, OUTPUT); pinMode(7, OUTPUT); // LED indicators -> output
 tone(4, 3000, 1000); // Play tone for 1 second
 delay(1000); // Delay to finish tone
 servoLeft.attach(13); // Attach left signal to pin 13
 servoRight.attach(12); // Attach right signal to pin 12
}
 
void loop() // Main loop auto-repeats
{
 // Sniff
 int irLeft = digitalRead(10); // Check for IR on left
 int irRight = digitalRead(3); // Check for IR on ri
 
 if((irLeft == 0) || (irRight == 0))
 {
   servoLeft.writeMicroseconds(1500);
   servoRight.writeMicroseconds(1500);
 for(int i = 0; i < 5; i++) // Repeat 5 times
 {
 digitalWrite(7, HIGH); // Turn indicator LEDs on
 digitalWrite(8, HIGH);
 tone(4, 4000, 10); // Sound alarm tone
 delay(20); // 10 ms tone, 10 between tones
 digitalWrite(7, LOW); // Turn indicator LEDs off
 digitalWrite(8, LOW);
 }
 delay(100);
 }
 
 // Roam
 irLeft = irDetect(9, 10, 38000); // Check for object on left
 irRight = irDetect(2, 3, 38000); // Check for object on right
 if((irLeft == 0) && (irRight == 0)) // If both sides detect
 {
 maneuver(-200, -200, 20); // Backward 20 milliseconds
 }
 else if(irLeft == 0) // If only left side detects
 {
 maneuver(200, -200, 20); // Right for 20 ms
 }
 else if(irRight == 0) // If only right side detects
 {
 maneuver(-200, 200, 20); // Left for 20 ms
 }
 else // Otherwise, no IR detects
 {
 maneuver(200, 200, 20); // Forward 20 ms
 }
}
 
int irDetect(int irLedPin, int irReceiverPin, long frequency)
{
 tone(irLedPin, frequency, 8); // IRLED 38 kHz for at least 1 ms
 delay(1); // Wait 1 ms
 int ir = digitalRead(irReceiverPin); // IR receiver -> ir variable
 delay(1); // Down time before recheck
 return ir; // Return 1 no detect, 0 detect
}
void maneuver(int speedLeft, int speedRight, int msTime)
{
 // speedLeft, speedRight ranges: Backward Linear Stop Linear Forward
 // -200 -100....0....100 200
 servoLeft.writeMicroseconds(1500 + speedLeft); // Left servo speed
 servoRight.writeMicroseconds(1500 - speedRight); // Right servo speed
 if(msTime==-1) // If msTime = -1
 {
 servoLeft.detach(); // Stop servo signals
 servoRight.detach();
 }
 delay(msTime); // Delay for msTime
}

