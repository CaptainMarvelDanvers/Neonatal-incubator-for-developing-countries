### Made by

Kristiina Niit, Isabelle Lind, Patrick Carrera Jeri 

Medical Engineering, Project Course

### Neonatal Incubator 

Our project was to make an medical incubator for developing countries with a focus on controlling the humidity. We used a bluetooth module to get humidity data from a DHT11 sensor and controlled it with an arduino.

### TLDR.... 

Low cost medical incubator. Uses boiling water with candle to generate humidity. 12V fan was chosen to bring down the humidity so we can have a stable humidty around 50%. The speed of the fan is determine with a PID system. PID system takes in a input and generates an output based on the current input. This case it's humidity in and PWM value out, the PWM value is used to handle the speed of the fan.

### What components are we going to use? 

DHT11 SENSOR - Find Current Temperature and humidity

Arduino - Controll everything 

2 12V FANS - To handle the temperature and humidity

METAL container - Hold the Water 

CANDLE - maintaine the tempeture in the water 

RESISTORS 

MOSFET - More output voltage from Arduino
### There was a plan
This is the final plan

![alt text](https://user-images.githubusercontent.com/46792060/72289837-3900f580-364c-11ea-8a92-1d2f13b578c2.png)

The current plan is to have a stable humidity at 50% 

### Controll of the Humidity? 
This is a two step problem. 

1. Generate Humidty: A way to generate humidty is to use boiling water. A problem with this is that the water will naturly lose temperature and lower the humidity production. Solution to this is a candle under the boiling water to keep it at the same temperature
2. Lower the Humidity: A fan is used to filter out the air

### Controll of the fan?

The speed of the fan is determine with a PID-system. The input for the PID system will be the current humidity and the Output of the PID system will be PWM signal to the Fan. TLDR: High Humidty = High Speed, Low Humidity = Low Speed. 

### Link to Short paper writen about this

https://docs.google.com/document/d/1ZC7YEh4ozLTuGMorGt0SCKr8AUzATNuX0OGmZ97gEUg/edit

### Give Arduino the ability to generate 12V from 5V
We need a way to generate 12V so we used a MOSFET with 12V Drain 

![alt text](https://user-images.githubusercontent.com/46792060/72335584-79e62200-36bf-11ea-8306-94b2d2aa4d73.png)

### Everything togther How did it look? 

![alt text](https://user-images.githubusercontent.com/46792060/72289573-b2e4af00-364b-11ea-8719-26426a0036b8.jpeg)

### Arduino code
Few things to consider before: 

1. DHT11 SENSOR will need atleast 1s to update it's values. 
2. PID system can be impruven on with more testing around the constants Kp, ki and kd 

```markdown
#include <dht.h>
#include <LiquidCrystal.h>
#include <AutoPID.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

dht DHT;
//Use PIN 9 and 6, those have PWM option....

int Hum_fan = 6;
int Temp_Fan = 9; 


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
PID gives the opotion to have a Propotional, Integral, Derivative. 

1. Propotional: This in its simplest form is error * Kp. In other words it's more error equals more output. Problem with this is that we will always overshot and undershot the set point (the value we are after) 
2. Integral: Ki * ∫(error(t)): This will take the sum of all errors over some time T and multiplies it by Ki: The output will be U=I+P. More overshot will increase the I and less overshot will reduce it. This part is used to reduce the overshot part this combined with P can generate a stable humidity in the incubator. 
3. Derivative: Kd * f'(e)/dt: takes the velocity of the error and multiplies it with kd. Faster rate of error will go down with higher D. This part is added when we need the system to react faster.

Final part: U=I+P+D. All these togther lets us have a chanse to get a stable output around the value we are after. 

### Find PID constants? 
In this case we used a "trial and error" attack. Start with I = D = 0 and adjust Kp until you have a oscillating behavior. Oscillating behavior have undershot and overshot and this is then corrected with the integral part. 

How you should start: 

step 1: Make a guess for kP (we started with 0.5) 

step 2: double the kP value 

step 3: oscillating behavior?

stpe 4: if yes... go to step 6 

step 5: if no go to step 2

step 6: Make a guess for kI(we started with 0.1) 

step 7: make slow adjustments, add 0.5. until you are happy with the error 

### Showing Data in Real Time with Matlab Code

This short matlab code was done to pressent data in real time. Arduino likes to send data in a form of a text file and this can cause some problems and the main problem is the following 

1. Assumes that arduino always sends data once! If for some reason we are given the number "1010" the code will convert it to 1010 and not 10 & 10. This is something to consider when doing this, a way to solve this is to hard code it in to the arduino that it will never update the sending value if it's the same value all the time. 

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

### Future Work? 

A Medical incubator needs to regulate both humidity and temperature. Our solution regulates humidity but has no inpact on the temperature in the incubator. Possible solutions to this? 

Nichrome. We could use nichrome in small strips over a 12V fan. if we use 1.2Amps with 12V we can generate up to 80W. This will generate alot of temperature in the incubator. This will have some troubling issues...

1. If we dont check nichrome often it can become hot enough to melt, if that happens it can damage the fan enough to break it and stop it from heating the incubator. 
2. To solve 1. We need to monitor the current temperatur of the nichrome and make sure it never reaches the melting point. 
3. if 2 and 1 solved we need to generate appropriate output to the value and it needs to be exact. this requires perfect values almsot because we need it hot enough to generate temperature and not high enough to melt.  

A Simple 60W Lamp can be used to generate some temperature, problem is that most Lamps needs 220V and this needs a relay system that can handle high voltages. A Relay system that handles high voltages cost alot! we are after low cost solutions
