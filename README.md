## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/CaptainMarvelDanvers/Refactored-invention/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Our project was to make an medical incubator for developing countries with a focus on controlling the humidity. We used a bluetooth module to get humidity data from a DHT11 sensor and controlled it with an arduino.
### The Plan

The plan was to have around 50% humidity in the incubator. 

### How are we going to controll the Humidity? 

To Generate humidity we are going to use boiling water a big problem with this is that the waters tempeture will slowly go down and that will reduce the humidity production. To solve this we used a candle under the water to keep it boiling. Now we are going to a way to lower the humidity and this is where a fan is going to be used to filter out the high humidity

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
Data was taken 
Data behövdes hämtas från Bluetooth till datorn för att sedan presentera det i real tid. Detta korta skriptet användas för just det. Notera att Arduino har en tendens att skicka ut allt sin data (bluetooth) i form av text och inget annat. För att kringå detta problem används str2num eller string to number. Detta funkar enbart om vi vet att Arudino inte skickar två saker samtidigt exempelvis om vi får in 1010 när det faktiskt är 10 10 då kommer matlab koden utan tvekan tolka det som 1010 när konverteringen sker. 


![alt text](https://user-images.githubusercontent.com/46792060/72286982-8bd7ae80-3646-11ea-97ae-5af537caaea5.PNG)

Här kan man se hur fuktigheten påverkar hastigheten av fläkten. 

###
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
