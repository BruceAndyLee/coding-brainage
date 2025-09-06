Сервопривод - это элемент, который может ставить в нужное положение вращающуюся часть мотора в зависимости от напряжения, подаваемого на управляющий контакт.

На сервоприводе есть три контакта:
- земля/минус
- плюс (5 вольт)
- контроллер

Arduino не всегда может запитать большие сервоприводы, нужно подключать внешний источник питания. При этом управлять ими можно и с arduino.
Самые простые и дешевые сервоприводы не могут двигаться больше, чем на 160-200 градусов.


```C
unsigned int DELAY_MS = 500;
unsigned char SERVO_CONTROL_PIN = 3;

unsigned int servoPosition = 0; // degrees
unsigned int servoStep = 5;

void setup() {
  pinMode(SERVO_CONTROL_PIN, OUTPUT);
}

void loop() {
  servoPosition += servoStep;
  analogWrite(SERVO_CONTROL_PIN, servoPosition);
  delay(DELAY_MS);
}
```

То же самое с дополнительной оберткой
```C
#include <Servo.h>
unsigned int DELAY_MS = 500;
unsigned char SERVO_CONTROL_PIN = 3;
unsigned int servoPosition = 165; // degrees?
unsigned int servoStep = 5;
Servo servo;

void setup() {
	pinMode(SERVO_CONTROL_PIN, OUTPUT);
	servo.attach(SERVO_CONTROL_PIN);
}

void loop() {
	servo.write(servoPosition);
	delay(DELAY_MS);
}
```