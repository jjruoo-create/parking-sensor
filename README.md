#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

const int trigPin = 14;
const int echoPin = 27;

const int greenLED = 32;
const int orangeLED = 33;
const int redLED = 25;

const int buzzer = 13;
const int buttonPin = 12;

bool systemOn = false;
bool lastButtonState = HIGH;

long duration;
int distance;

void setup() {

  lcd.begin();
  lcd.backlight();

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(greenLED, OUTPUT);
  pinMode(orangeLED, OUTPUT);
  pinMode(redLED, OUTPUT);

  pinMode(buzzer, OUTPUT);
  pinMode(buttonPin, INPUT_PULLUP);

  Serial.begin(115200);

  lcd.setCursor(0,0);
  lcd.print("PRESS BUTTON");
  lcd.setCursor(0,1);
  lcd.print("TO START");
}

void loop() {

  bool buttonState = digitalRead(buttonPin);

  if(lastButtonState == HIGH && buttonState == LOW){

    if(systemOn && distance < 40){

      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("SAFETY LOCK!");


      delay(1200);

    }
    else{

      systemOn = !systemOn;

      lcd.clear();

      if(systemOn){
        lcd.setCursor(0,0);
        lcd.print("SYSTEM READY");
      }
      else{
        lcd.setCursor(0,0);
        lcd.print("SYSTEM OFF");

        digitalWrite(greenLED, LOW);
        digitalWrite(orangeLED, LOW);
        digitalWrite(redLED, LOW);
        noTone(buzzer);
      }

      delay(800);
    }
  }

  lastButtonState = buttonState;

  if(!systemOn){
    return;
  }

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);

  distance = duration * 0.034 / 2;

  Serial.println(distance);

  lcd.setCursor(0,0);
  lcd.print("DIST: ");
  lcd.print(distance);
  lcd.print(" cm ");

  if(distance < 20){

    digitalWrite(orangeLED, LOW);
    digitalWrite(greenLED, LOW);

    //  ไฟแดงกระพริบ
    digitalWrite(redLED, HIGH);
    delay(120);
    digitalWrite(redLED, LOW);
    delay(120);

    lcd.setCursor(0,1);
    lcd.print("STOP!!! ");

    tone(buzzer,2000);

  }

  else if(distance < 30){

    digitalWrite(redLED, LOW);
    digitalWrite(orangeLED, HIGH);
    digitalWrite(greenLED, LOW);

    lcd.setCursor(0,1);
    lcd.print("STOP!! ");

    tone(buzzer,2000);
    delay(120);
    noTone(buzzer);
    delay(120);

  }

  else if(distance < 40){

    digitalWrite(redLED, LOW);
    digitalWrite(orangeLED, HIGH);
    digitalWrite(greenLED, LOW);

    lcd.setCursor(0,1);
    lcd.print("STOP ");

    tone(buzzer,2000);
    delay(300);
    noTone(buzzer);
    delay(300);

  }

  else{

    digitalWrite(redLED, LOW);
    digitalWrite(orangeLED, LOW);
    digitalWrite(greenLED, HIGH);

    lcd.setCursor(0,1);
    lcd.print("SAFE ");

    noTone(buzzer);
  }

  delay(50);
}
