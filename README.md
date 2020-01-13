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
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/CaptainMarvelDanvers/Refactored-invention/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
