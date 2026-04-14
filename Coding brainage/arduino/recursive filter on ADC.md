Простейшее решение с одной переменной для накопления данных
```C
// hardware setup
int INPUT_PIN = A0;

// recursive filter setup
float RESPONSE = 0.1;
float DC_normalizer = 5. / 1023.;

// state
float filtered_adc_read = 0.0;

void apply_filter(int read_value) {
	float cur_read = read_value * DC_normalizer;
	
	filtered_adc_read += (cur_read - filtered_adc_read) * RESPONSE;
}

// infra
void setup() {
	pinMode(INPUT_PIN, INPUT);
}

void loop() {
	apply_filter(analogRead(INPUT_PIN));
	
	// send filtered_adc_read to serial or sth
	delay(10);
}
```