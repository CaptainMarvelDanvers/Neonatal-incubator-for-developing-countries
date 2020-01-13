## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/CaptainMarvelDanvers/Refactored-invention/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Neonatal kuvös till lågt pris för utvecklingsländer
Gjort Av Kristiina Niit, Isabelle Lind, Patrick Carrera Jeri
HL 2030
Skolan för Kemi, Bioteknologi och hälsa (CBH)

Sammanfattning                                           
Prematura barn är underutvecklade och behöver växa i en kontrollerad miljö vilket är svårt att åstadkomma utan rätt teknik. Vid rätt luftfuktighet och temperatur kan barnen fortsätta utvecklas och växa i rätt miljö som ska efterlikna livmodern.
Med hjälp av en kuvös som innesluter barnet i en isolerad miljö med rätt luftfuktighet och värme kan barnet växa och utvecklas i säkerhet.
Detta ska lösas med luftfuktighet- och temperatursensor samt fläktsystem som kontrollerar och reglerar fukt och temperatur till en jämn nivå med digital övervakning. Allt detta monteras i en låda med ett skikt av vatten som värms upp med hjälp av resistorer, ett skikt med elektrisk krets och huvuddelen som påverkas av de övriga skikten som ska hålla en jämn nivå av luftfuktighet och temperatur. Detta ska även fungera i u-länder med hjälp av relativt billiga och enkla komponenter.


Introduktion

Genom att bygga en kuvös ska värmen och luftfuktigheten regleras. Kuvösens mjukvara och hårdvara ska tillämpas så att den ska vara enkel, billig samt fungerande i U-länder. Kuvöser är viktiga för infantila barn och är en av de viktigaste komponenterna för barnets överlevnad. En kuvös ska hålla den rätta temperaturen som är runt 37 grader celcius och den rätta luftfuktigheten som är 50-80% (upp till 18 dagar)(Stephen baumgart md faap nephrology and fluid/electrolyte Physiology (Third edition) 2019). En metod är att reglera ventilationen och luftfuktigheten, vilket har gjorts. En annan metod är att reglera ventilationen, luftfuktigheten, infant reflux och bullret. I detta dokumentet presenteras en metod att bygga en kuvös vars applikationer är bluetooth-styrda detta genom att hålla ned kostnaderna. Men även ha en kontrollerad miljö i kuvösen och jämföra med ett  tidigare arbete inom lågkostnadskuvöser “Project Omoverhi : low-cost, neonatal incubator. Richard Fong, Guillermo Gallardo, William Jeffery, Danny Maeda,Gabriel Romero". 
Design

Bild 1. Visar ett  flödesschema över kuvösen
Vad som har hänt vid start är att allt har initieras, elementet har satts på för att sedan börjat med loopen, temperatur har övervakats och därefter har temperaturfläktens hastighet samt luftfuktigheten reglerats. Efter detta har temperaturen och fuktighetsvärdet skickats via bluetooth till datorn för att presentera data i realtid.

Bild 2. Visar en skiss av kuvösens komponenter 
I kuvösen har det använts en DHT11 SENSOR för att ta fram temperaturen och fuktigheten. Arduino valdes på grund av att den har flera PWM-signaler medan PIC enbart har en PWM-output som kan användas. En sensor har valts för att hålla kostnaden och komplexiteten nere. Kokande vatten har tillsatts ner i en behållare och har använts som källa för värme och fuktighet. Vattnet har hållit värmen med hjälp av ett värmeljus under behållaren, vilket i sin tur har ökat fuktigheten i kuvösen.
Resultat

Bild 3. Hur fläkten påverkade fuktigheten 
Flera tester gjordes för att klargöra vilka fuktighetsvärden som kunde nås men även hur snabbt det har gått. Snabbheten var ett krav på grund av att PID-systemet har blivit instabilt om hastigheten har förändrats. Systemets fuktighetsproduktion har varit tillräckligt bra för att producera 30-40%(se ref [4] [6])  nästa steg var att se om 50% var något systemet kunde producera och som man kan se på bild 3 var det möjligt. Genomsnittlig fuktighet var 49.7% med fel på +-5%. Bättre PID-värden har kunnat öka prestandan på systemet och det har inte varit någon påverkan på temperaturen. 50% var minimivärdet enligt “Nyföddhet: Temperatur och fukt i kuvös (Barn- och ungdomskliniken i Linköping/Motala)”
Slutsats
Vi presenterar en enkel modell av en kuvös med få inställningar, det vill säga temperatur- och fuktighetskontroll. Vår produkt kommer reducera kostnaden massivt men har även en kontrollerad miljö som är grunden i en kuvös.
Systemet som presenterades kan reglera fuktigheten däremot har den ingen påverkan på temperaturen. En lösning som ej kunde testas var att använda Glödtråd som läggs över en fläkt som i sin tur producerar värde som fläkten kan blåsa ut. 
Diskussion

När fläkten har varit avstängd läcker värme och fukt ur så att värdena blir mindre stabila.

Det har varit svårt att kontrollera både värme och luftfuktighet så att de blir konstanta och inom rätt intervall man har önskat.

Det har varit svårt att hålla vattnet med vår första idé med att ha resistorer med 10 ohm, men för att det ska fungera bra måste effekten vara högre än den vi tänkte från början. Därför har det till slut blivit en lösning med värmekälla i form av värmeljus. Denna värmekälla har varit bättre med svår att kontrollera.
Fläkten har pipigt när den har fått för låg spänning, detta är i praktiken inte optimalt då det kan vara skadligt för barn med höga ljud. Detta problem har vi dock löst genom att lägga en offsetspänning på utsignalen.


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
Data behövdes hämtas från Bluetooth till datorn för att sedan presentera det i real tid. Detta korta skriptet användas för just det. Notera att Arduino har en tendens att skicka ut allt sin data (bluetooth) i form av text och inget annat. För att kringå detta problem används str2num eller string to number. Detta funkar enbart om vi vet att Arudino inte skickar två saker samtidigt exempelvis om vi får in 1010 när det faktiskt är 10 10 då kommer matlab koden utan tvekan tolka det som 1010 när konverteringen sker. 
![Skärmklipp](https://user-images.githubusercontent.com/46792060/72286982-8bd7ae80-3646-11ea-97ae-5af537caaea5.PNG)
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
