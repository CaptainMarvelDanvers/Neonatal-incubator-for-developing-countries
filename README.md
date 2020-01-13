
You can use the [editor on GitHub](https://github.com/CaptainMarvelDanvers/Refactored-invention/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

### Neonatal Incubator 

Our project was to make an medical incubator for developing countries with a focus on controlling the humidity. We used a bluetooth module to get humidity data from a DHT11 sensor and controlled it with an arduino.
### There was a plan
This was the inicial ide 

![alt text](https://user-images.githubusercontent.com/46792060/72289837-3900f580-364c-11ea-8a92-1d2f13b578c2.png)

The plan was to have around 50% humidity in the incubator. 

### Controll of the Humidity? 

This is a two step problem. We first need to generate humidity and we need a way to lower the humidity. A way to generate humidity is to use boiling water. A problem with this solution is that the water will naturly lose temperature and lower the production off humidity a sulution to this was to use a candle. How do we lower the humidity? we use a fan to filter air out. 

### Controll of the fan?

The speed of the fan is determine with a PID-system. The input for the PID system will be the current humidity and the Output of the PID system will be PWM signal to the Fan. TLDR: High Humidty = High Speed, Low Humidity = Low Speed. 

### Everything togther How did it look? 

![alt text](https://user-images.githubusercontent.com/46792060/72289573-b2e4af00-364b-11ea-8719-26426a0036b8.jpeg)

### Arduino code

Some part of the code is not active and the reason behind this was ..... osäker hur vi gör här.... Ta bort manuelt alla komentarer eller säger vi varför det är borta? 

```markdown
#include <dht.h>
#include <LiquidCrystal.h>
#include <AutoPID.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

dht DHT;
//Use PIN 9 and 6, those have PWM option....

int Hum_fan = 6;
int Temp_Fan = 9; 
int Fan_Speed = 0; 

int Heater;
//int HeaterSpeed;
//This Will Be for the PID SYSTEM!
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

AutoPID myPID(&HumRead, &setPoint, &outputHum, OUTPUT_MIN, OUTPUT_MAX, KP, KI, KD);
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
  
  Lower_HUM_FAN();
  PID_TEMP_FAN();
  ShowValues(); // This Will Send Current Data/Values...
  //HeaterControll();
  delay(1250);
}
/*
 * Heater Should Reach Temp Wanted
 * And Then keep it at that point... PID system Here aswell? 
 */
void HeaterControll() {

  //HeaterSpeed = (12/12)*256;
  
  //digitalWrite(Heater,HIGH); 

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
// PID system for HUM fan....... 
// while(Hum < 30) KEEP PID on 
// End with Giving HUM_FAN LOW
void Lower_HUM_FAN() {

  //setPoint = 35;
  //HumRead = DHT.humidity;
  //myPID.run();
  //analogWrite(Hum_fan, outputHum); 
  digitalWrite(Hum_fan,HIGH);
  
  
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


### Showing Data in Real Time with Matlab Code

This short matlab code was done to pressent data in real time. Arduino likes to send data in a form of a text file and this can cause some problems and the main problem is the following 

1. Assumes that arduino always sends data once! If for some reason we are given the number "1010" the code will convert it to 1010 and not 10 & 10.  

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
### Real Time data with PWM signal 
Here we can see how the humidity affects the fan speed. 
![alt text](https://user-images.githubusercontent.com/46792060/72286982-8bd7ae80-3646-11ea-97ae-5af537caaea5.PNG)
