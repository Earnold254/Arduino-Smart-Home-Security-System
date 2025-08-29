# Arduino-Smart-Home-Security-System
This project introduces children to electronics and programming using Arduino. It combines an RFID sensor, keypad, distance sensor, LCD screen, buzzer, servo motor, and a traffic light module to create a simple home security system. The project teaches logic, problem solving, and hardware interaction in a fun and safe way.



Components Required

Arduino Uno	1	Main controller board

RFID Module (RC522)	1	Reads RFID cards

RFID Keycards	2+	Used as access keys

20x4 LCD (16-pin)	1	Displays messages

10k Potentiometer	1	Controls LCD contrast

4x4 Keypad	1	Enters password

Ultrasonic Sensor (HC-SR04)	1	Detects nearby person

Servo Motor	1	Acts as the door lock

Traffic Light Module (Red, Yellow, Green LEDs)	1	System status indication

Buzzer	1	Gives sound feedback

Resistors (220Ω)	3	For LEDs and backlight

Breadboard + Jumper Wires	1 set	Connections




Wiring Connections


LCD	RS	12

LCD	E	11

LCD	D4	5

LCD	D5	4

LCD	D6	3

LCD	D7	2

LCD	VSS	GND

LCD	VDD	5V

LCD	V0	Potentiometer

LCD	RW	GND

LCD	A (Backlight +)	5V via 220Ω

LCD	K (Backlight -)	GND

Servo Motor	Signal	6

Buzzer	Signal	7

Ultrasonic Sensor	TRIG	8

Ultrasonic Sensor	ECHO	9

Traffic Light	Red	10

Traffic Light	Yellow	A0

Traffic Light	Green	A1

RFID	SDA	A3

RFID	RST	A2

Keypad	Rows/Cols	A4-A11




Arduino Code


Below is the Arduino sketch for the Smart Home System. It uses multiple libraries: LiquidCrystal, Servo, Keypad, SPI, and MFRC522. The system detects when a person is near, prompts them to scan their RFID card or enter a PIN, and then grants or denies access accordingly.




#include <LiquidCrystal.h>
#include <Servo.h>
#include <Keypad.h>
#include <SPI.h>
#include <MFRC522.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
#define SERVO_PIN 6
#define BUZZER_PIN 7
#define TRIG_PIN 8
#define ECHO_PIN 9
#define RED_LED 10
#define YELLOW_LED A0
#define GREEN_LED A1
#define RST_PIN A2
#define SS_PIN  A3
MFRC522 mfrc522(SS_PIN, RST_PIN);
Servo doorServo;

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {A4, A5, A6, A7}; 
byte colPins[COLS] = {A8, A9, A10, A11};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

String correctPIN = "1234"; 
String inputPIN = "";
long duration;
int distance;

void setup() {
  Serial.begin(9600);
  SPI.begin();
  mfrc522.PCD_Init();
  lcd.begin(20, 4);
  lcd.print("System Initializing");
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(RED_LED, OUTPUT);
  pinMode(YELLOW_LED, OUTPUT);
  pinMode(GREEN_LED, OUTPUT);
  digitalWrite(YELLOW_LED, HIGH);
  doorServo.attach(SERVO_PIN);
  doorServo.write(0);
}

void loop() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  duration = pulseIn(ECHO_PIN, HIGH);
  distance = duration * 0.034 / 2;

  if (distance < 20) {
    lcd.clear();
    lcd.print("Scan Card / Enter PIN");
    if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
      tone(BUZZER_PIN, 1000, 200);
      digitalWrite(GREEN_LED, HIGH);
      lcd.setCursor(0,1); lcd.print("Card Accepted!");
      doorServo.write(90);
      delay(3000);
      doorServo.write(0);
      digitalWrite(GREEN_LED, LOW);
      mfrc522.PICC_HaltA();
    }
    char key = keypad.getKey();
    if (key) {
      if (key == '#') {
        if (inputPIN == correctPIN) {
          tone(BUZZER_PIN, 1000, 200); 
          digitalWrite(GREEN_LED, HIGH);
          lcd.setCursor(0,1); lcd.print("PIN Accepted!");
          doorServo.write(90);
          delay(3000);
          doorServo.write(0);
          digitalWrite(GREEN_LED, LOW);
        } else {
          for (int i=0; i<3; i++) {
            tone(BUZZER_PIN, 2000, 100);
            delay(200);
          }
          digitalWrite(RED_LED, HIGH);
          lcd.setCursor(0,1); lcd.print("Wrong PIN!");
          delay(2000);
          digitalWrite(RED_LED, LOW);
        }
        inputPIN = "";
      } else {
        inputPIN += key;
        lcd.setCursor(0,2); lcd.print(inputPIN);
      }
    }
  }
}


