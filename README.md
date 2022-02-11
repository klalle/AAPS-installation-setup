# AAPS
AAPS (AndroidAPS) är byggt med ett stort säkerhetstänk och du kommer (till skillnad från ios-loop) INTE tillåtas att slå på en loop som är helt självgående och pytsar i insulin från början, utan du lotsas genom ett helt gäng "Mål" som du måste ta dig igenom och sakta men säkert öppna upp fler funktioner som tillslut gör loopen mer och mer självgående och kraftfullare. 
Se till att du läser på  hur appen fungerar så att du kan styra den på ett säkert sätt. [Dokumentationen](https://androidaps.readthedocs.io/en/latest) är på engelska, men lättläst - är det nåt du inte förstår, så ställ en fråga i fb-gruppen [Looped - Sweden](https://www.facebook.com/groups/loopedsweden) eller den internationella [AndroidAPS users](https://www.facebook.com/groups/AndroidAPSUsers). Tips är också att sökfunktionen är bra (dock kan det vara svårt att komma på översättningen då AAPS-appen är på svenska)

<img src="./images/search.png" height=300>

AAPS och installationsprocess är väldokumenterad, men jag tycker att den saknar en röd tråd för hur allt hänger ihop så jag tänkte förtydliga lite med ett exempel på hur jag har satt upp systemet.

# Förklaring av alla ingående komponenter
Här nedan ser du hur vi satt upp vårt system. Har inte mer än testkört NSClient-appen, då användaren i vårt fall är självgående, men tog med den som ett exempel.
![AAPS_system_overview](./images/AAPS_system_overview.png)


## Pumpar
AAPS stödjer [flertalet pumpar](https://androidaps.readthedocs.io/en/latest/Hardware/pumps.html) och några som har stöd för blåtand så att de kan prata direkt med telefonen som kör AAPS:
- Dana-R pump
- Dana-RS pump
- Accu-Chek Combo pump
- Accu-Chek Insight pump
- Diaconn G8 insulin pump
- Omnipod Eros
- **Omnipod DASH**
- Medtronic pump

Observera att AAPS nu stödjer Omnipod DASH som pratar blåtand och inte behöver en Rileylink. Sitter du på Eros, kan jag tipsa om att direkt ansöka om att få byta till DASH! Vi fick byta på en vecka trots att receptet hade långt kvar på Eros! Mycket bättre kontakt, en färre pryl att ha koll på/ladda och den återansluter jättebra vid tappad kontakt! Notera dock att [inte alla Android-telefoners](https://docs.google.com/spreadsheets/d/1zO-Vf3wv0jji5Gflk6pe48oi348ApF5RvMcI6NG5TnY) blåtand är så bra med DASH!

## CGM
Alla loop-system är beroende av en stabil och korrekt CGM (koninuerlig glukos-mätare) som levererar ett BG-värde minst var 5e minut. AAPS rekomenderar i nuläget Dexcom G6, då det är den som har tillräckligt stabil mätning och utjämnande algoritm för att kunna aktivera den mer aggresiva typen av loop (SMB). Det går att loopa med [de andra systemen också](https://androidaps.readthedocs.io/en/latest/Configuration/BG-Source.html), men då kommer AAPS inte tillåta SMB (Super micro bolus), vilket vore synd att vara utan - men funkar helt ok ändå, bara inte lika aggresivt. 
Nu för tiden rekomenderar utvecklarna för AAPS att vi använder den så kallade BYODA ("Bygg din egen Dexcom-app"), vissa använder xDrip för att hämta BG-värden från sändaren, men det har krånglat på sistone, och BYODA funkar utmärkt (och kan fortsätta skicka till diasend!) Det rekomenderade är att endast använda xDrip som larm-app då den har mycket fler och bättre larm-funktionalitet och funkar både på loop-telefonen och på följar-telefoner. 

Vill du byta från ios loop till AAPS, så kan du börja med att köra dubbelt ett tag (upp till mål 5!?), då kör du virtuell pump och ställer in `NSClient BG` som din BG-källa, så laddas de i stället ner från NS. 



## Nightscout
Nightscout (NS) är en moln-baserad tjänst som sparar och visualiserar/tillhandahåller historisk data från ditt loop-system. AAPS skickar upp sina värden och beräkningar till NS var 5e minut och NS tar emot datan och lagrar den i en databas som du själv har satt upp och har full kontroll över (Mongodb i Atlas).

AAPS i sig är inte beroende av någon extern databas/tjänst (NS), men eftersom de absolut flesta användarna vill kunna titta på historisk data (längre tillbaka än de 48h som AAPS håller) och ha möjlighet för t.ex. föräldrar att följa AAPS på distans, så är det i nuläget tvingande att ha en NS (Nightscout) i de första målen och när man installerar AAPS. Till skillnad från att logga in på Dexcom och titta på dina BG-värden, kommer du kunna se ALL data i NS så som måltider, bolusar, temporära basaler och profilbyten. 

Det är väldigt viktigt att den som loopar förstår hur loopen tänker, och att man förstår varför den gör som den gör i olika lägen! Det lär man sig absolut bäst genom att titta på historisk data i t.ex. NS-hemsidans rapport-verktyg där det framkommer väldigt tydligt hur AAPS har jobbat.

![BYODA-version](./images/NS_exempel.png)

## Hur installerar man en app som inte kommer från google
1.  Stäng av "Play protect"
    - Play protect ska behållas avslagen då den annars kommer försöka stänga av alla dina AAPS-appar varje natt!
    - Inställningar/Säkerhet/Appsäkerhet/Kugghjulet uppe till höger
        - Stäng av både "Sök igenom appar" och "Förbättra appidentifiering"
    - Använd Inställningarnas sökfönster och sök efter "Appsäkerhet" eller "protect" om du inte hittar inställningen!
2. Tillåt installation från annan källa
    - Flera av systemen som kommer bygger på att du installerar en app på telefonen som inte finns på google-play. För att kunna göra det måste du gå in på inställningar och berätta för telefonen att förslagsvis "Drive" (google drive). 
    
    - På min telefon (Google pixel 4a med android 12) hittas denna inställning: 
        - Inställningar/Appar/Särskild appåtkomst/Installera okända appar
        - Använd Inställningarnas sökfönster och sök efter "okända" eller "installera" om du inte hittar inställningen!
        - I den listan väljer "Drive" och tillåter att den får installera andra appar.
        - <img src="./images/AllowInstalation.png" height="400">


    - Dropbox funkar också, men tillåter inte direktinstallation från appen, då måste man först kopiera apk-filen till telefonens internminne (tre punkter/Exportera/Spara till Enhet) och sedan tillåta att t.ex. "Files by google" får installera appar (likt steget för Drive ovan).



## Dexcom BYODA
Den oficiella Dexcom-appen är låst till att bara fungera på vissa mobiler och den är också låst till att bara skicka sin data till Dexcom-share. Du behöver en Dexcom-app som förutom att skicka till Dexcom-share och diasend, också delar med sig av BG-värden till AAPS (och xDrip). För detta finns en patchad (hackad) variant som du själv [konfigurerar/bygger i ett google-formulär](https://docs.google.com/forms/d/e/1FAIpQLScD76G0Y-BlL4tZljaFkjlwuqhT83QlFM5v6ZEfO7gCU98iJQ/viewform?fbzx=2196386787609383750&fbclid=IwAR2aL8Cps1s6W8apUVK-gOqgGpA-McMPJj9Y8emf_P0-_gAsmJs6QwAY-o0) och sedan får en länk att ladda ner appen (apk-filen).
När du fyller i detta formulär är det viktigt att du väljer rätt på dessa inställningar: 

Tillåt att den installeras på ALLA android-telefoner

<img src="./images/BYODA_devices.png" height="100">

Versionen av dexcom som vi har i Europa:

<img src="./images/BYODA_version.png" height="250">

Skippa INTE de 2h warmup utan värden och tillåt att dexcom skikar värden internt inom telefonen till AAPS och eventuellt xDrip (välj detta även om du inte har xDrip till att börja med)

<img src="./images/BYODA_brodcast.png" height="400">

Resten är ganska själv-förklarande tror jag (använd default-värdena på de du är osäker på)

Du får nu ett mejl med en nedladdningslänk inom 5min som du laddar ner och lägger på Drive. OBS! Kolla skräpposten, där hamnade mitt mejl iaf! 

Under tiden så kan du passa på att ladda ner xDrip+ [här](https://xdrip-plus-updates.appspot.com/stable/xdrip-plus-latest.apk) som du också lägger på Drive (den kommer du vilja ha för larm!).

## Installera Nightscout

Installation av NS är väl beskrivet i detalj [här](http://nightscout.github.io/nightscout/new_user/), vill du i stället ha det berättat för dig, så har Jonas Hummelstrand gjort en [youtube-tutorial](https://youtu.be/rNIpmIhPCpU) på svenska där han går igenom processen steg för steg. OBS! Skippa allt som har med BRIDGE att göra (steg 13 och framåt), du ska INTE sätta upp Dexcom bridge då vi istället ska skicka upp all data direkt från AAPS. 

När du är klar med detta steg så har du fortfarande ingen data att visa, men det ska gå att komma åt NS-hemsidan i en webläsare!

## Bygg AAPS
För att verkligen vara tydlig med att detta är ett DIY-system och att du tar fullt ansvar för alla följder som kan tänkas bli, så måste du själv ladda ner koden från Github och bygga appen. Jag tänkte inte gå igenom alla steg då de är väl beskrivna [här](https://androidaps.readthedocs.io/en/latest/Installing-AndroidAPS/Building-APK.html), men som det ser ut nu, så är ett av de sista stegen i `Build the app` fel, och gör att många fastnar... 

I steget när du har byggt appen i AndroidStudio och öppnar "Event log" så kommer det inte alls stå att du byggt "1 variant" med en länk. Det du egentligen ska kolla efter är att det INTE står nåt i stil med: 

`Generate Signed APK: Errors while building APK. You can find the errors in the 'Messages' view.`

Om allt har gått bra så står det bara:

`Generate Signed APK: APK(s) generated successfully for module 'AndroidAPS.app' with 0 build variants:` 

Och nu måste du själv leta reda på apk-filen som kommer ligga under mappen som du laddade ner all kod till (inte androidStudio, utan AAPS-koden) och där i har det skapats en mapp "app" där du hittar:
`app/full/app-full-release.apk`. 

Ta nu och ladda upp `app-full-release.apk` till Drive och du kan från och med nu fokusera helt på mobiltelefonen.

## Installationer på telefon
Nu kommer det roliga.
Förutsättning är att du har Drive på telefonen så att du ser dina uppladdade appar (.apk-filer)

### Dexcom
Läs mer [här](https://androidaps.readthedocs.io/en/latest/Hardware/DexcomG6.html#if-using-g6-with-build-your-own-dexcom-app) om du vill se vad dokumentationen säger...
- Om du har kör original-dexcom-appen och har en aktiv sensor+sändare så kan du byta till BYODA utan att förlora någon av dom!
    - Gå in i original-Dexcom-appen och skriv noga ner de sensor-koden och sändar-serienummer som du har för de aktiva sensorn och sändaren. 
    - Avinstallera dexcom-appen
    - Installera den nya BYODA-dexcom-appen som du lagt på Drive genom att trycka på den.
    - Logga in som vanligt och aktivera sensor-kod och sändar-serienummer.
    - Sen står det såhär (kommer inte ihåg om det var nåt jag gjorde) 
        - In phone settings go to apps > Dexcom G6 > permissions > additional permissions and press ‘Access Dexcom app’.
    - Efter ett tag så kommer appen hitta sändaren och börja logga värden precis som förut. Enda skillnaden är att den också tillåter AAPS/xdrip att få tillgång till datan. 

### AAPS
Installera nu AAPS genom att trycka på `app-full-release.apk`-filen i Drive
Tror att du automatiskt kommer till "Installationsguiden" (hittas annars i menyn 3-punkter övre högra hörnet/installationsguide)
#### Installationsguide:
- Alla inställningar kommer kunna ändras senare, så ingen panik att det måste bli rätt från början. Du kommer INTE tillåtas att slå på en loop som är helt självgående och pytsar i insulin från början, utan du lotsas genom ett helt gäng "Mål" som du måste ta dig igenom och sakta men säkert öppna upp fler funktioner som tillslut gör loopen mer och mer självgående och kraftfullare. 
- Se lite screenshots nedanför denna lista: 
- Godkänn allt som AAPS vill ha, behövigheter/platsåtkomst mm
- Välj ett "Huvudlösenord" som du skriver upp (går att återskapa...)
- Det ploppar upp notiser om att "ingen profil vald" - det är ok
- Visningsinställningar har inget med loopen att göra, visar bara grönt område på grafen!
- NSClient - tryck på "Aktivera NSClient" och fyll i din NS-adress (inkl `https://`) och verifiera att du får kontakt (Status: `Authenticated (RWT)`)
- Välj insulintyp (notera här att DIA inte kommer vara det du är van vid!)
- Välj BG-källa (dexcom) och fyll i att du vill `Ladda upp BG-data till NS`
    - OBS! vanligt att inte få in BG-värdena i AAPS i detta steg! Då väljer du en annan BG-källa, och sedan direkt tillbaka till dexcom, för att trigga en liten popup som frågar om tillstånd...
- Profil, AAPS jobbar med profiler man kan byta mellan som håller koll på inställningarna:
    - IC (insulin till kh-kvot, hur många gram kh tar en enhet insulin hand om)
    - ISF (insulinkänslighetsfaktor, hur många mmol/l i BG sjunker du på en enhet insulin)
    - basal
    - Mål (BG-målvärde/område)
    - OBS! loopen tar för givet att dessa är korrekta och får därför svårt att jobba om du inte testar dessa med jämna mellanrum. Ofta varierar de dessutom över dygnet, så du kan speca olika värden för olika timmar i alla utom Mål-värdet. 
- Profilbyte (appen vill att du gör ett första profilbyte, så att den väljer din nyskapade profil, ändra inga värden, tryck bara "Genomför profilbyte"/"OK"!)
- Pump - välj "virtuel" pump först för att ha nåt att jobba med i första målet (du behöver ingen riktig pump förän mål 5, men antar att du vill ansluta den så snart du kan) Du kan också kryssa i att ladda status upp till NS.
- APS - vilken algoritm ska AAPS jobba efter - i senare skede kommer du gå över till SMB, men det får du inte tillgång till förän ett långt senare skede, så börja med AMA som det står där.
- APS-läge - Börja med öppen loop (du måste manuellt genomföra alla ändringar. Du har ändå ingen rättighet att sätta på closed loop än...)
- Känslighetsavkänning - sätt oref1. 

<img src="./images/instal_1_visning.png" height="400"> <img src="./images/instal_2_nsclient.png" height="150"> <img src="./images/instal_2_nsclient_2.png" height="400"> <img src="./images/instal_3_insu.png" height="400"> <img src="./images/instal_4_BG.png" height="400"> 
<img src="./images/instal_5_dia.png" height="400"> <img src="./images/instal_5_target.png" height="400"><img src="./images/instal_5_CR.png" height="400"><img src="./images/instal_5_done.png" height="400"> 
<img src="./images/instal_6_profil.png" height="400"> <img src="./images/instal_6_profil2.png" height="400"> <img src="./images/instal_7_pump.png" height="400"> <img src="./images/instal_8_ama.png" height="400">
<img src="./images/instal_9_loop.png" height="400"> <img src="./images/instal_10_oref1.png" height="400">

Du hittar alla viktiga saker i de två menyerna i vardera övre hörnen. Jag kan inte gå igenom allt, du måste själv bekanta dig med var du hittar allt! Se [AndroidAPS screens](https://androidaps.readthedocs.io/en/latest/Getting-Started/Screenshots.html) för mer info!
Men här är lite kort info:
- "Hamburgermenyn" (de tre strecken uppe i vänstra hörnet) där du (förutom genvägar till vald pump/insu/cgm mm) når:
    - Mål - det här är där du måste klara av målen ([Objectives](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html) på engelska)
    - Underhåll - här exporterar du dina inställningar (Gör det efter varje klarat mål, och lägg på Drive så att du kan återskapa var du var i målen om mobilen dör!)
    - Konfiugreringsverktyget
     inställningar (kugghjul) och väljer vilka flikar som ska synas (gröna ögat -> tryck i checkboxen så aktiveras en ny flik i huvudfönstret)
- Högermenyn (tre prickar uppe till höger) här nås massor av inställningar 

<img src="./images/overview.png" height="400"> <img src="./images/hamburger.png" height="400"> <img src="./images/konfigverktyg.png" height="400">

Nu är det bara att börja jobba dig igenom Målen ett efter ett (läs snälla på om varje mål! finns länkar vid varje mål i appen och du har ju länken till "[Objectives](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html)"). När du är klar kommer du ha en vältrimmad loop som du förhoppningsvis vet hur du ska justera när insulinkänsligheten förändras. 


## Klocka
Du kan både se BG/IOB/COB i klockan och lägga till måltider och få bolus för dessa. 

<img src="./images/watch2.png" height="400">

Smart-klockor måste numera byggas separat på samma sätt som du bygger AAPS (man väljer bara en annan "Module" i byggsteget). läs [här](https://androidaps.readthedocs.io/en/latest/Hardware/Smartwatch.html) om aaps och smartwatch

<img src="./images/watch.png" height="400">

## NSClient (för föräldrar/följare)
Läs på om [Remote monitoring med AAPS](https://androidaps.readthedocs.io/en/latest/Children/Children.html)

<img src="./images/remote.png">

NSClient är en app som (oftast) inte behöver byggas själv, utan släpps med senaste releasen av AAPS och kan laddas ner från [github](https://github.com/nightscout/AndroidAPS/releases/) där du hittar två byggda apk-filer (för att kunna styra 2 barn med olika AAPS). För över med Drive och installera som de övriga apparna. 

<img src="./images/nsclientapk.png">