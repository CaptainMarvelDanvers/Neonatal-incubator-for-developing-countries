### Made by

Kristiina Niit, Isabelle Lind, Patrick Carrera Jeri 

Medical Engineering, Project Course

### Neonatal Incubator 

Our project was to make an medical incubator for developing countries with a focus on controlling the humidity. We used a bluetooth module to get humidity data from a DHT11 sensor and controlled it with an arduino.

### TLDR.... 

Low cost medical incubator. Uses boiling water with candle to generate humidity. 12V fan was chosen to bring down the humidity so we can have a stable humidty around 50%. The speed of the fan is determine with a PID system. PID system takes in a input and generates an output based on the current input. This case it's humidity in and PWM value out, the PWM value is used to handle the speed of the fan.

### What components are we going to use? 

DHT11 SENSOR - Find Current temperature and humidity

Arduino - Controll everything 

2 12V FANS - To handle the temperature and humidity

METAL container - Hold the Water 

CANDLE - maintaine the tempeture in the water 

RESISTORS - part of the relay system 

MOSFET - More output voltage from Arduino
### There was a plan
This is the final plan

![alt text](https://user-images.githubusercontent.com/46792060/72289837-3900f580-364c-11ea-8a92-1d2f13b578c2.png)

The current plan is to have a stable humidity at 50% 

### Controll of the humidity? 
This is a two step problem. 

1. Generate humidity: A way to generate humidity is to use boiling water. A problem with this is that the water will naturly lose temperature and lower the humidity production. Solution to this is a candle under the boiling water to keep it at the same temperature
2. Lower the Humidity: To lower the humidity we use a 12V fan

### Controll of the fan?

The speed of the fan is determine with a PID-system. The input for the PID system will be the current humidity and the Output of the PID system will be PWM signal to the Fan. TLDR: High Humidity = High Speed, Low Humidity = Low Speed. 

### Link to Short paper writen about this

https://docs.google.com/document/d/1ZC7YEh4ozLTuGMorGt0SCKr8AUzATNuX0OGmZ97gEUg/edit

### Give arduino the ability to generate 12V from 5V
We need a way to generate 12V so we used a MOSFET with 12V Drain 

![alt text](https://user-images.githubusercontent.com/46792060/72335584-79e62200-36bf-11ea-8306-94b2d2aa4d73.png)

### Everything togther How did it look? 

![alt text](https://user-images.githubusercontent.com/46792060/72289573-b2e4af00-364b-11ea-8719-26426a0036b8.jpeg)

### Arduino code
DHT11 sensor will need atleast 1s to read values 

```markdown
#include <dht.h>
#include <LiquidCrystal.h>
#include <AutoPID.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

dht DHT;
//Use PIN 9 and 6, those have PWM option....

int Hum_fan = 6;
int Temp_Fan = 9; 

#define OUTPUT_MIN 0
#define OUTPUT_MAX 255
/*
 * How we set this up is -> start with KP.... Make it have Swining Behav 
 * Adjust KI to kill the error.... 
 * Adjust KD to make it react faster 
 */
#define KP 10
#define KI 0.4
#define KD 0

double HumRead, setPoint, outputHum;
double TempRead,SetPoint,OutPutTemp;

// from 100 - 255 PWM.... Needs that for temp.... 
AutoPID my2PID(&TempRead, &SetPoint, &OutPutTemp, OUTPUT_MIN,OUTPUT_MAX,KP,KI,KD);

#define DHT11_PIN 7

void setup() {
  lcd.begin(16, 2);
  pinMode(Temp_Fan,OUTPUT);
  pinMode(Hum_fan, OUTPUT);
  pinMode(Heater, OUTPUT);
  setPwmFrequency(9, 6);
  Serial.begin(9600); 
  Fan_Speed = 0; 
  SetPoint = 50; 
}


void loop()
{
  int chk = DHT.read11(DHT11_PIN);
  lcd.setCursor(0, 0);
  
  lcd.print("Temp: ");
  lcd.print(DHT.temperature);
  lcd.print((char)223);
  lcd.print("C");
  lcd.setCursor(0, 1);
  lcd.print("Humidity: ");
  lcd.print(DHT.humidity);
  lcd.print("%");
  
  PID_TEMP_FAN();
  ShowValues(); // This Will Send Current Data/Values...
 
  delay(1250);
}

void PID_TEMP_FAN(){
  int out = 0;
  int offset = 110; 
  TempRead = DHT.humidity;
  my2PID.run();
  out = 255 - OutPutTemp + offset; 
  if (out > 255)
  {
    out = 255; 
  }
  if (out == offset){
    out = 0; 
  }
  analogWrite(Temp_Fan, out);

}



void ShowValues() {
  int value = DHT.humidity;
  int out = 0;
  int offset = 110; 
  Serial.print((value)); //Send that the Averge Temp for 8 ticks... or 1250 * 8 =
  Serial.print("\n");
  value = DHT.temperature;
  
  out = 255 - OutPutTemp + offset; 
  
  Serial.print((out));
  Serial.print("\n");  
 
}
void setPwmFrequency(int pin, int divisor) {
  byte mode;
  if (pin == 5 || pin == 6 || pin == 9 || pin == 10) {
    switch (divisor) {
      case 1: mode = 0x01; break;
      case 8: mode = 0x02; break;
      case 64: mode = 0x03; break;
      case 256: mode = 0x04; break;
      case 1024: mode = 0x05; break;
      default: return;
    }
    if (pin == 5 || pin == 6) {
      TCCR0B = TCCR0B & 0b11111000 | mode;
    } else {
      TCCR1B = TCCR1B & 0b11111000 | mode;
    }
  } else if (pin == 3 || pin == 11) {
    switch (divisor) {
      case 1: mode = 0x01; break;
      case 8: mode = 0x02; break;
      case 32: mode = 0x03; break;
      case 64: mode = 0x04; break;
      case 128: mode = 0x05; break;
      case 256: mode = 0x06; break;
      case 1024: mode = 0x07; break;
      default: return;
    }
    TCCR2B = TCCR2B & 0b11111000 | mode;
  }
}
```
### Why PID system? 
A propotional sulotion overshot with 10+-%... PID was needed to reduce the error. 

Set Point = value wanted. 

1. Propotional: error * kP. More error = More output. This sulution will always overshot the Set point. 
2. Integral: Ki * ∫(error(t)): PI = P + I. More error = Even more output, use I to reduce the overshot. 
3. Derivative: Kd * f'(e)/dt: PID = P + I + D. High rate of error = Faster response.  

### Showing data in real time with Matlab

```markdown
%% This Code Shows Real Time Data.... 

b = Bluetooth('HC-06',1); %Make The Connection... 
fopen(b); 


amount = 200 ;                    % make This many Calls
x = 0:1:amount; 

for tick = 1:amount
    
    humidity(tick) = str2num(fgetl(b));  %Read Data, Convert, Save 
    plot(x(1:tick),humidity(1:tick),'-r'); %Plot Current Data
    xlabel('Time'); 
    ylabel('Humidity in %, PWM in %'); 
    title('Humidity = Red, PWM = BLUE'); 
    drawnow;
    hold on     %Wait and Plot One more Thing....
    
    PWM(tick) = (str2num(fgetl(b))/256)*100;
    plot(x(1:tick),PWM(1:tick),'-b');
    drawnow; 
    AvergeHum = mean(humidity); 
    AvergeTemp = mean(PWM);
end 
hold off
fclose(b);
```
### Real time data with PWM signal 
Here we can see how the humidity affects the fan speed. 
![alt text](https://user-images.githubusercontent.com/46792060/72286982-8bd7ae80-3646-11ea-97ae-5af537caaea5.PNG)
