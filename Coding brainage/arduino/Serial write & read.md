```C
void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
}

void loop() {
  // put your main code here, to run repeatedly:
  Serial.println("Yo dawg! What's your message?");

  while (Serial.available() == 0) {
    continue;
  }

  String answer = Serial.readString();
  Serial.println(answer.c_str());
}

```