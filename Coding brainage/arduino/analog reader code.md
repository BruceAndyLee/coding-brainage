```C
unsigned char READ_ANALOG_PIN = A0;
unsigned int DELAY_MS = 100;

void setup() {
  Serial.begin(9600);
  pinMode(READ_ANALOG_PIN, INPUT);
}

void loop() {
  Serial.println((5. / 1023.) * analogRead(READ_ANALOG_PIN));
  delay(DELAY_MS);
}

```