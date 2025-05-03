Добавляем потенциометр и управляем скоростью мигания светодиода с его помощью.

```C
int CONTROL_PIN = 10;
int DIODE_PIN = 9;
int SLEEP_MS = 25; 
int READING_PIN = A0;
int oscillation_frequency = 1; // hz

double dt = 0.025;
double time = 0;

void setup() {
  // put your setup code here, to run once:
  pinMode(READING_PIN, INPUT);
  pinMode(DIODE_PIN, OUTPUT);
  pinMode(CONTROL_PIN, OUTPUT);

  digitalWrite(CONTROL_PIN, 1);
  Serial.begin(9600);
}

unsigned char getAnalogPinValue() {
  time += dt;
  return char(50 + sin(oscillation_frequency * time) * 50);
}

void applyAnalogValue(unsigned char value) {
  analogWrite(DIODE_PIN, value);
}

void loop() {
  int state = getAnalogPinValue();
  applyAnalogValue(state);
  oscillation_frequency = int(0.5 + (50. / 1023.) * analogRead(READING_PIN));
  
  delay(SLEEP_MS);
}
```