[[pushbutton]]

Сырой вариант без препода
```C
unsigned int initialCooldown = 400;
unsigned int LOOP_DELAY_MS = 100;

unsigned char READ_BUTTON_PIN = A0;
unsigned char LIGHT_PIN = 2;

unsigned int BUTTON_CLICK_COOLDOWN_MS = 100;

bool lightOn = false;
bool cooldownInEffect = false;
unsigned int cooldown = 0;

void setup() {
  Serial.begin(9600);
  pinMode(READ_BUTTON_PIN, INPUT);
  pinMode(LIGHT_PIN, OUTPUT);
}

void loop() {

  // Куллдаун в начале работы платы.
  // Видимо, из-за начального дребезга на читающем пине
  // лампа загоралась сама по себе.
  if (initialCooldown) {
    initialCooldown -= 10;
    delay(LOOP_DELAY_MS);
    return;
  }

  bool pullupButtonPressed = digitalRead(READ_BUTTON_PIN) == 0;

  // не вычитаем из куллдауна, если кнопка все еще не нажата
  // чтобы длинное нажатие считалось как одно нажатие 
  if (cooldownInEffect && !pullupButtonPressed) {
    cooldown -= LOOP_DELAY_MS;
    if (cooldown <= 0) {
      cooldownInEffect = false;
    }
  }

  if (pullupButtonPressed && !cooldownInEffect) {
    lightOn = !lightOn;
    digitalWrite(LIGHT_PIN, lightOn ? 1 : 0);
    cooldownInEffect = true;
    cooldown = BUTTON_CLICK_COOLDOWN_MS;
  }
  
  delay(LOOP_DELAY_MS);
}

```
