[[Stepper motor]]

Реализация полношагового режима
```C
#include <binary.h>

unsigned int STEP_DELAY = 1000; // actually reciprocal to rotation speed

unsigned char COIL_A1 = 8;
unsigned char COIL_A2 = 9;
unsigned char COIL_B1 = 10;
unsigned char COIL_B2 = 11;

int coilsStateInd = 0;

// written in reverse order to keep the bytes written from A1 through to B2
unsigned char StepperMotorPins[4] = {COIL_B2, COIL_B1, COIL_A2, COIL_A1};

// the bytes
int FullStepBytes[4] = {B1010, B0110, B0101, B1001};
int STEP_CYCLE_LENGTH = 4;

// unsigned char 

void setup() {
  for (int i = 0; i < 4; i++) {
    pinMode(StepperMotorPins[i], OUTPUT);
  }
}

void updateRotorCoilsState() {
  coilsStateInd = (coilsStateInd + 1) % STEP_CYCLE_LENGTH;
}

void applyCoilsState() {
  unsigned char coilState = FullStepBytes[coilsStateInd];
  for (int i = 0; i < STEP_CYCLE_LENGTH; i++) {
    unsigned char coilPin = StepperMotorPins[i];
    digitalWrite(coilPin, coilState & (1 << i));
  }
}

void loop() {
  updateRotorCoilsState();
  applyCoilsState();
  delay(STEP_DELAY);
}
```

В полушаговом режиме логика такая же, но комбинаций напряжения в два раза больше:
```C
// the bytes
int FullStepBytes[8] = {B1000, B1010, B0010, B0110, B0100, B0101, B0001, B1001};
int STEP_CYCLE_LENGTH = 8;
```