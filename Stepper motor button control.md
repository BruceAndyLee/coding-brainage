Идея:
- добавить две кнопки
- крутить шаговый двигатель в направлении, закрепленном за кнопкой

```C
unsigned int LOOP_DELAY_MS = 10;
int initialCooldown = 500;

unsigned char READ_FORWARD_PIN = A0;
unsigned char READ_BACKWARD_PIN = A1;

unsigned char COIL_A1 = 8;
unsigned char COIL_A2 = 9;
unsigned char COIL_B1 = 10;
unsigned char COIL_B2 = 11;

int coilsStateInd = 0;

unsigned char StepperMotorPins[4] = {
  COIL_A1,
  COIL_A2,
  COIL_B1,
  COIL_B2,
};
int MotorStepBytes[4] = {
  B1010,
  B0110,
  B0101,
  B1001,
};
// int MotorStepBytes[4] = { // DOESNT REVERSE SHIT!!!!s
//   B1001,
//   B0101,
//   B0110,
//   B1010,
// };
// int MotorStepBytes[8] = {
//   B1000,
//   B1010,
//   B0010,
//   B0110,
//   B0100,
//   B0101,
//   B0001,
//   B1001,
// };
unsigned int STEP_CYCLE_LENGTH = 8;
unsigned int STEP_WORD_LENGTH = 4;

unsigned char DIR_FORWARD = 0;
unsigned char DIR_BACKWARD = 1;
unsigned char STEPS_PER_ITERATION = 20;

typedef struct {
  unsigned char dir; // forward or backward
  int steps;
  bool moving; // whether a button is currently pressed or not
} MotorState;

MotorState motorState = {
  DIR_FORWARD,
  0,
  false,
};

void setup() {
  Serial.begin(9600);

  // put your setup code here, to run once:
  for (int i = 0; i < 4; i++) {
    pinMode(StepperMotorPins[i], OUTPUT);
  }

  pinMode(READ_FORWARD_PIN, INPUT);
  pinMode(READ_BACKWARD_PIN, INPUT);
}

void registerControlPinSignals() {
  int forwardSignal = digitalRead(READ_FORWARD_PIN);
  int backwardSignal = digitalRead(READ_BACKWARD_PIN);
  
  if (forwardSignal == 0 && backwardSignal == 0) {
    motorState.moving = false;
    return;
  }

  motorState.moving = true;
  motorState.dir = forwardSignal ? DIR_FORWARD : DIR_BACKWARD;
}

void updateMotorState() {
  if (!motorState.moving) {
    return;
  }

  if (motorState.dir == DIR_FORWARD) {
    coilsStateInd = (coilsStateInd + 1) % STEP_CYCLE_LENGTH;
  }
  if (motorState.dir == DIR_BACKWARD) {
    coilsStateInd = (coilsStateInd - 1) % STEP_CYCLE_LENGTH;
  }
}

void writeWord(int word) {
  for (int i = 0; i < STEP_WORD_LENGTH; i++) {
    unsigned char coilPin = StepperMotorPins[i];
    digitalWrite(coilPin, word & (1 << i));
  }
}

void moveMotor() {
  if (!motorState.moving) {
    writeWord(B0000);
    return;
  }
  writeWord(MotorStepBytes[coilsStateInd]);
}

void loop() {
  if (initialCooldown > 0) {
    initialCooldown -= LOOP_DELAY_MS;
    delay(LOOP_DELAY_MS);
    return;
  }
  registerControlPinSignals();
  // update virtual state
  updateMotorState();
  // update physical state
  // (aka send signals to the motor)
  moveMotor();

  delay(LOOP_DELAY_MS);
}

```