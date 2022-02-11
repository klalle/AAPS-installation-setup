# AAPS-installation-setup
AAPS och installationsprocessen är [väldokumenterad](https://androidaps.readthedocs.io/en/latest) på engelska, men jag tycker att den saknar en röd tråd för hur allt hänger ihop, så jag tänkte förtydliga lite med ett exempel på hur jag har satt upp systemet.

# Förklaring av alla ingående komponenter
![AAPS_system_overview](./images/AAPS_system_overview.png)
Observera att AAPS nu stödjer Omnipod DASH som pratar blåtand och inte behöver en Rileylink. Sitter du på Eros, ansök direkt om att få byta till DASH! Vi fick byta på en vecka trots att receptet hade långt kvar på Eros! Mycket bättre kontakt och den återansluter jättebra vid tappad kontakt! Notera dock att [inte alla Android-telefoners](https://docs.google.com/spreadsheets/d/1zO-Vf3wv0jji5Gflk6pe48oi348ApF5RvMcI6NG5TnY) blåtand är så bra med DASH!

## CGM
Alla loop-system är beroende av en stabil och korrekt CGM (koninuerlig glukos-mätare) som levererar ett BG-värde minst var 5e minut. AAPS rekomenderar i nuläget Dexcom G6, då det är den som har tillräckligt  stabil mätning för att kunna lita på. Det går att loopa med [de andra systemen också](https://androidaps.readthedocs.io/en/latest/Configuration/BG-Source.html), men då kommer AAPS inte tillåta SMB (Super micro bolus), vilket vore synd att vara utan - men funkar helt ok ändå, bara inte lika aggresivt. 
Nu för tiden rekomenderar utvecklarna för AAPS att vi använder den så kallade BYODA ("Bygg din egen Dexcom-app"), vissa använder dock istället xDrip för att hämta BG-värden från sändaren, men det är nog mest av gammal vana! Det rekomenderade är att endast använda xDrip som larm-app då den har mycket fler och bättre larm-funktionalitet och funkar både på loop-telefonen och på följar-telefoner. 

Se [Installation Nightscout](#Installation-Nightscout)

## Nightscout
Nightscout (NS) är en moln-baserad tjänst som sparar och visualiserar/tillhandahåller historisk data från ditt loop-system. AAPS skickar upp sina värden och beräkningar till NS var 5e minut och NS tar emot datan och lagrar den i en databas som du själv har satt upp och har full kontroll över (Mongodb i Atlas).

AAPS i sig är inte beroende av någon extern databas/tjänst (NS), men eftersom de absolut flesta användarna vill kunna titta på historisk data (längre tillbaka än de 48h som AAPS håller) och ha möjlighet för t.ex. föräldrar att följa AAPS på distans, så är det i nuläget tvingande att sätta upp NS (Nightscout) att i de första stegen när man installerar AAPS. Till skillnad från att logga in på Dexcom och titta på dina BG-värden där, kommer du kunna se ALL data i NS så som måltider, bolusar, temporära basaler och profilbyten. 

Det är väldigt viktigt att den som loopar förstår hur loopen tänker, och att man förstår varför den gör som den gör i olika lägen! Det lär man sig absolut bäst genom att titta på historisk data i t.ex. NS-hemsidans rapport-verktyg där det framkommer väldigt tydligt hur AAPS har jobbat. Här t.ex. ser ISF ut att behöva höjas lite, och kanske även CR (I:C)

![BYODA-version](./images/NS_exempel.png)

## Hur installerar man en app som inte kommer från google
1.  Stäng av "Play protect"

    - Play protect ska behållas avslagen då den annars kommer försöka stänga av alla dina AAPS-appar varje natt!
    - Inställningar/Säkerhet/Appsäkerhet/Kugghjulet uppe till höger
     - Stäng av både "Sök igenom appar" och "Förbättra appidentifiering"
2. Tillåt installation från annan källa
    - Flera av systemen som kommer bygger på att du installerar en app på telefonen som inte finns på google-play. För att kunna göra det måste du gå in på inställningar och berätta för telefonen att förslagsvis "Drive" (google drive). 
    
    - På min telefon (Google pixel 4a med android 12) hittas denna inställning: 
        - Inställningar/Appar/Särskild appåtkomst/Installera okända appar
        - Lättast är nog om du öppnar inställningar och söker i sökrutan efter "okända" eller "installera" så hittar du nog snabbt till rätt ställe.
I den listan väljer "Drive" och tillåter att den får installera andra appar.

<img src="./images/AllowInstalation.png" height="400">


Dropbox funkar också, men tillåter inte direktinstallation från appen, då måste man först kopiera apk-filen till telefonens internminne (tre punkter/Exportera/Spara till Enhet) och sedan tillåta att t.ex. "Files by google" får installera appar (likt steget för Drive ovan).



## Installera CGM
Den oficiella Dexcom-appen är låst till att bara fungera på vissa mobiler och den är också låst till att bara skicka sin data till Dexcom-share. Det finns en patchad (hackad) variant som du själv konfigurerar/bygger i ett google-formulär och sedan får en länk att ladda ner appen (apk-filen).
Så när du fyller i detta formulär är det viktigt att du väljer rätt på dessa inställningar: 

Tillåt att den installeras på ALLA android-telefoner

<img src="./images/BYODA_devices.png" height="100">

Versionen av dexcom som vi har i Europa:

<img src="./images/BYODA_version.png" height="250">

Skippa INTE de 2h warmup utan värden och tillåt att dexcom skikar värden internt inom telefonen till AAPS och eventuellt xDrip (välj detta även om du inte har xDrip till att börja med)

<img src="./images/BYODA_brodcast.png" height="400">

Resten är ganska själv-förklarande tror jag (använd default-värdena på de du är osäker på)

Du får nu ett mejl med en nedladdningslänk inom 5min.

## Installera Nightscout

Installation av NS är väl beskrivet i detalj [här](http://nightscout.github.io/nightscout/new_user/), men det kan kännas lite väl tekniskt, så lättast kanske är att följa en [youtube-tutorial](https://youtu.be/rNIpmIhPCpU) på svenska som Jonas Hummelstrand har lagt upp om hur man sätter upp NS med mongodb på Atlas. OBS! Skippa allt som har med BRIDGE att göra (steg 13 och framåt), du ska INTE sätta upp Dexcom bridge då vi istället ska skicka upp all data direkt från AAPS. 

När du är klar med detta steg så har du fortfarande ingen data att visa, men det ska gå att komma åt NS-hemsidan i en webläsare!

## Bygg AAPS
För att verkligen vara tydlig med att detta är ett DIY-system och att du tar fullt ansvar för alla följder som kan tänkas bli, så måste du själv ladda ner koden från Github och bygga appen. Jag tänkte inte gå igenom alla steg då de är väl beskrivna [här](https://androidaps.readthedocs.io/en/latest/Installing-AndroidAPS/Building-APK.html), men som det ser ut nu, så är ett av de sista stegen i `Build the app` fel, och gör att många fastnar... 

I steget när du har byggt appen i AndroidStudio och öppnar "Event log" så kommer det inte alls stå att du byggt "1 variant" med en länk. Det du egentligen ska kolla efter är att det INTE står nåt i stil med: 

`Generate Signed APK: Errors while building APK. You can find the errors in the 'Messages' view.`

Om allt har gått bra så står det bara:

`Generate Signed APK: APK(s) generated successfully for module 'AndroidAPS.app' with 0 build variants:` 

Och nu måste du själv leta reda på apk-filen som kommer ligga under mappen som du laddade ner all kod till (inte androidStudio, utan AAPS-koden) och där i har det skapats en mapp "app" där du hittar:
`app/full/app-full-release.apk`. 

Ta nu och ladda upp `app-full-release.apk` till t.ex. Gdrive och 

d

- BG-värde
- basal
- temporär basal (loopen ändrar din basal kontinuerligt)
- ISF (insulinkänslighetsfaktor)
- DIA (duration of insulin active som är )
- IOB (insulin on bord => hur många enheter mer än basalen har du i kroppen)
- COB (carbs on bord => hur många gram kolhydrater har du just nu i kroppen, räknas automatiskt ner utifrån hur din BG-kurva förändras)
- Profiler/profilbyten (AAPS jobbar med profiler som kan ha olika basal/ISF/IC) 
