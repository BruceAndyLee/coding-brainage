[[Serial2Parallel shift register]]

Выдача одного из двух заданных заранее байтов
```C
unsigned int DELAY = 500;

unsigned int CLOCK_PIN = 9;
unsigned int LATCH_PIN = 10;
unsigned int SERIAL_DATA_PIN = 11;

#define LEDS_LENGTH 2

unsigned char LEDs[LEDS_LENGTH] = {
  0b01010101,
  0b10101010,
};
unsigned char currentHexInd = 0;

void setup() {
  pinMode(CLOCK_PIN, OUTPUT);
  pinMode(LATCH_PIN, OUTPUT);
  pinMode(SERIAL_DATA_PIN, OUTPUT);

  digitalWrite(LATCH_PIN, LOW);
  shiftOut(SERIAL_DATA_PIN, CLOCK_PIN, LSBFIRST, LEDs);
  digitalWrite(LATCH_PIN, HIGH);
}

void loop() {

  unsigned char curLEDs = LEDs[(currentHexInd++) % LEDS_LENGTH];

  digitalWrite(LATCH_PIN, LOW);
  shiftOut(SERIAL_DATA_PIN, CLOCK_PIN, LSBFIRST, curLEDs);
  digitalWrite(LATCH_PIN, HIGH);

  delay(DELAY);
}

```

Счетчик числа в бинарном представлении
