Написать микропрогу, которая управляет яркостью светодиода с помощью пина в аналоговом режиме.
Диапазон значений: 0 - 255.
```C
int RED_PIN = 9;
int SLEEP_MS = 25;

double dt = 0.1;
double time = 0;

void setup() {
	pinMode(RED_PIN, OUTPUT);

	// for reading analog signal thru ADC
	pinMode(READING_PIN, INPUT);
	// for printing value to the serial port
	// and sending it to the host PC
	Serial.begin(9600);
}

unsigned char getAnalogPinValue() {
	time += dt;
	return char(50 + sin(time) * 50);
}

void showCounterState(unsigned char state) {
	analogWrite(RED_PIN, state);
}

void loop() {
	int state = getAnalogPinValue();
	showCounterState(state);

	// extra logic to use ADC
	read_voltage = analogRead(READING_PIN);
	// raw adc value
	// Serial.println(read_voltage);
	Serial.println((5./1023.) * read_voltage);

	delay(SLEEP_MS);
}
```

Сигнал не аналоговый. Ардуино самостоятельно посылает ШИМ-сигнал, который приводит к среднему значению, соответствующему указанному значению.

Можно поставить конденсатор, который физически усреднит сигнал и почти точно выдаст нужное значение.

АЦП 10ти разрядный, поэтому, чтобы получить вольты, можно домножить считанное значение на `5/1023`
