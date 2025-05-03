```C
int bit0 = 2;
int bit1 = 3;
int bit2 = 4;
int bit3 = 5;

int bitPins[4] = {bit0, bit1, bit2, bit3};

char state = 0b0;
int sleepMs = 1000;

void setup() {
// put your setup code here, to run once:
	for (int i = 0; i < 4; i++) {
		pinMode(bitPins[i], OUTPUT);
	}
}

void updateCounterState() {
	if (state == 0b00001111) {
		state = 0b0;
	}
	state += 1;
}

void showCounterState() {
	for (int i = 3; i >= 0; i--) {
		digitalWrite(bitPins[i], (state >> i) & 0b1);
	}
}

void loop() {
	updateCounterState();
	showCounterState();
	delay(sleepMs);
}
```