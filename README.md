# SMARCLE_Farm
<Source_Code>
'''arduino
#define soilWet 500 
#define soilDry 750

#define sensorPower 7
#define sensorPin A0

#include <dht.h>
#define outPin 8  // dht11
dht DHT;

#include <Servo.h>
int servoPin = 9;
Servo servo;
int angle = 0;  
int sw = 13;

int AA = 10;  // AA를 연결한 디지털 핀(워터펌프)
int AB = 6;   // AB를 연결한 디지털 핀

int fan1 = 2;
int fan2 = 3;

int waterheight = A5;

int led = 4;

void setup() {
  pinMode(sensorPower, OUTPUT);
  
  // Initially keep the sensor OFF
  digitalWrite(sensorPower, LOW);
  
  Serial.begin(9600);

  servo.attach(servoPin);

  pinMode(sw, INPUT);

  pinMode(fan1,OUTPUT);
  pinMode(fan2,OUTPUT);

  pinMode(led, OUTPUT);
  
  pinMode(AA, OUTPUT);  // AA를 출력 핀으로 설정,워터펌프
  pinMode(AB, OUTPUT);  // AB를 출력 핀으로 설정,워터펌프
}

void soilwet_func();
void water_jet();

void water_jet()    //워터펌프
{                  
  digitalWrite(AA, HIGH);  // 정방향으로 모터 회전
  digitalWrite(AB, LOW);
  delay(2500); 
 
  digitalWrite(AA, LOW); 
  digitalWrite(AB, LOW);
 
 
}
// function
void soilwet_func(){      // 토양수분센서
  int moisture = readSensor();
  Serial.println(moisture);
  if (moisture < soilWet) {
    Serial.println("Status: Soil is too wet");
  } 
  else if (moisture >= soilWet && moisture < soilDry) {
    Serial.println("Status: Soil moisture is perfect");
  } 
  else {
    Serial.println("Status: Soil is too dry - time to water!");
    water_jet();
  }
  
  Serial.println();
}

void fan_func()   //환기팬
{
  digitalWrite(fan1,HIGH);
  digitalWrite(fan2,HIGH);
  delay(8000);
  digitalWrite(fan1,LOW);
  digitalWrite(fan2,LOW);
  
}

void dht_func()    // 온습도 측정센서
{
  int readData = DHT.read11(8);
  float t = DHT.temperature;  
  float h = DHT.humidity;  

  Serial.print("Temperature = ");
  Serial.print(t);
  Serial.print("°C | ");
  Serial.print((t*9.0)/5.0+32.0); 
  Serial.println("°F ");
  Serial.print("Humidity = ");
  Serial.print(h);
  Serial.println("% ");
  Serial.println("");
  int realhum = h - 100;
  if( realhum >= 50){
    fan_func();
  }
 
}

void servo_sw()    // 스위치, 편광필름
{
  if(digitalRead(sw) == HIGH){
    Serial.println("sw High");
    int i = 90;
    while(true){
      servo.write(i);
      if (i <= 0)
        break;
      i--;
      delay(15);
    }
    delay(3000);
    i = 0;
    while(true){
      servo.write(i);
      if (i >= 90)
        break;
      i++;
      delay(15); 
    }
  }
  else{
    Serial.println("sw Low");  
  }
}


void loop()  // 수분수위센서
{
  Serial.print("Analog Output: ");

  int val = analogRead(A5);
  Serial.print("waterheight = ");
  Serial.println(val);
  if (val <= 400){
    Serial.println("led on");
    digitalWrite(4, HIGH);
  }
  else{
    digitalWrite(4, LOW);
  }
  
  soilwet_func();
  dht_func();
  servo_sw();
  delay(1000);
 
  
}

int readSensor() {
  digitalWrite(sensorPower, HIGH);  
  delay(10);            
  int val = analogRead(sensorPin);  
  digitalWrite(sensorPower, LOW);   
  return val;            
}
'''
