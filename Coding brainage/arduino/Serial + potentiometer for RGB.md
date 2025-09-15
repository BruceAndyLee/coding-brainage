```C
unsigned int RED_PIN = 6;
unsigned int GREEN_PIN = 5;
unsigned int BLUE_PIN = 3;
unsigned int DELAY_MS = 100;

unsigned int HIGH_VOLTAGE_PIN = 8;
unsigned int READ_POTENTIOMETER_PIN = A0;
unsigned int READ_PHOTORESISTOR_PIN = A1;

unsigned int maxIntensity = 50;
String lastRequestedColor = "white";

void setup() {
  Serial.begin(9600);

  // put your setup code here, to run once:
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);

  pinMode(READ_POTENTIOMETER_PIN, INPUT);
  pinMode(READ_PHOTORESISTOR_PIN, INPUT);
  pinMode(HIGH_VOLTAGE_PIN, OUTPUT);

  analogWrite(RED_PIN, maxIntensity);
  analogWrite(GREEN_PIN, maxIntensity);
  analogWrite(BLUE_PIN, maxIntensity);
  digitalWrite(HIGH_VOLTAGE_PIN, HIGH);
}

void applyColorSetup(String requestedColor, unsigned int maxIntensity) {
  if (requestedColor == String("white")) {
    Serial.println("white");
    analogWrite(RED_PIN, maxIntensity);
    analogWrite(GREEN_PIN, maxIntensity);
    analogWrite(BLUE_PIN, maxIntensity);
    return;
  }
  if (requestedColor == String("red")) {
    Serial.println("red");
    analogWrite(RED_PIN, maxIntensity);
    analogWrite(GREEN_PIN, 0);
    analogWrite(BLUE_PIN, 0);
    return;
  }
  if (requestedColor == String("green")) {
    Serial.println("green");
    analogWrite(RED_PIN, 0);
    analogWrite(GREEN_PIN, maxIntensity);
    analogWrite(BLUE_PIN, 0);
    return;
  }
  if (requestedColor == String("blue")) {
    Serial.println("blue");
    analogWrite(RED_PIN, 0);
    analogWrite(GREEN_PIN, 0);
    analogWrite(BLUE_PIN, maxIntensity);
    return;
  }
}

void loop() {
  // put your main code here, to run repeatedly:

  int potentiometerIntensity = int((255. / 1023.) * analogRead(READ_POTENTIOMETER_PIN));
  if (maxIntensity != potentiometerIntensity) {
    applyColorSetup(lastRequestedColor.c_str(), potentiometerIntensity);
  }
  maxIntensity = potentiometerIntensity;

  if (Serial.available() != 0) {
    String requestedColor = Serial.readStringUntil('\n');
    requestedColor.trim();
    applyColorSetup(requestedColor, maxIntensity);
    lastRequestedColor = requestedColor;
    Serial.print(requestedColor);
  }
  delay(DELAY_MS);
}

```