/*
 * 
 */

#include <Servo.h>
Servo servo;

//-------------INITIALIZE GLOBAL VARIABLES
// Ultrasonic Module Pins
const int echo = 2;
const int trig = 3;

// Servo motor
const int servoPin = 4;
int servoAngles[6] = {0, 25, 50, 110, 145, 180};    // Scanning angles for ultrasonic module
int motor_left[] = {9, 6};        // Left motor pins
int motor_right[] = {11, 10};     // Right motor pins

// Other values
long duration;
float forwarddistance;
float distancefront;
float distanceArr[6];
int thresholddistance = 30;

//-------------FUNCTIONS FOR HANDLING MOTION
void brake(){
  digitalWrite(motor_left[0], LOW);
  digitalWrite(motor_left[1], LOW);
  digitalWrite(motor_right[0], LOW);
  digitalWrite(motor_right[1], LOW);
}

void drive_forward(){
  digitalWrite(motor_left[0], LOW);
  delayMicroseconds(400);
  digitalWrite(motor_left[1], HIGH);
  delayMicroseconds(400);
  digitalWrite(motor_right[0], LOW);
  delayMicroseconds(400);
  digitalWrite(motor_right[1], HIGH);
  delayMicroseconds(400);
}

void turn_around(){
  digitalWrite(motor_left[0], HIGH);
  digitalWrite(motor_left[1], LOW);
  digitalWrite(motor_right[0], HIGH);
  digitalWrite(motor_right[1], LOW);
  delay(600);
}

float distance(int angle){
  servo.write(angle);
  delay(500);
  digitalWrite(trig, LOW);
  delayMicroseconds(20);
  digitalWrite(trig, HIGH);
  delayMicroseconds(20);
  digitalWrite(trig, LOW);

  duration = pulseIn(echo, HIGH);
  forwarddistance = duration / 58;
  return forwarddistance;
}

//-------------Sensor module
void scan_area(){
  for (int i = 0; i < 6; i++){
    servo.write(servoAngles[i]);
    delay(100);
    digitalWrite(trig, LOW);
    delayMicroseconds(20);
    digitalWrite(trig, HIGH);
    delayMicroseconds(20);
    digitalWrite(trig, LOW);
    duration = pulseIn(echo, HIGH);
    distanceArr[i] = duration / 58;
    Serial.print("distanceArr[");
    Serial.print(i);
    Serial.print("] = ");
    Serial.println(distanceArr[i]);
    delay(100);
  }
}

//-------------SETUP FUNCTION TO BE RAN WHEN POWERED ON
void setup(){
  Serial.begin(9600);
  servo.attach(servoPin);

  // Setup motors
  for (int i = 0; i < 2; i++){
    pinMode(motor_left[i], OUTPUT);
    pinMode(motor_right[i], OUTPUT);
  }
  
  // Setup sensor
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
}

//-------------MAIN LOOP
void loop(){
  servo.write(75);
  distancefront = distance(75);
  Serial.println(distancefront);

  if ((distancefront > thresholddistance) && (distancefront < 1000)){
    drive_forward();
    delay(1000);
    Serial.println("1");
  }
  else {
    brake();
    scan_area();
    if ((distanceArr[1] > 1000) && (distanceArr[2] > 1000) && (distanceArr[3] > 1000) && (distanceArr[4] > 1000) && (distanceArr[5] > 1000) && (distanceArr[6] > 1000)){
       return;
       Serial.println("Restarted the Loop");
       delay(100);
    }
    
    else {
      turn_around();
      brake();
      delay(1000);
      drive_forward();
      delay(1000);
      Serial.println("6");
    }
  }
}
