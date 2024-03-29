# AAPS
AAPS (AndroidAPS) är byggt med ett stort säkerhetstänk och du kommer (till skillnad från ios-loop) INTE tillåtas att slå på en loop som är helt självgående och pytsar i insulin från början, utan du lotsas genom ett helt gäng "Mål" som du måste ta dig igenom och sakta men säkert öppna upp fler funktioner som tillslut gör loopen mer och mer självgående och kraftfullare. 
Se till att du läser på  hur appen fungerar så att du kan styra den på ett säkert sätt. [Dokumentationen](https://androidaps.readthedocs.io/en/latest) är på engelska, men lättläst - är det nåt du inte förstår, så ställ en fråga i fb-gruppen [Looped - Sweden](https://www.facebook.com/groups/loopedsweden) eller den internationella [AndroidAPS users](https://www.facebook.com/groups/AndroidAPSUsers). Tips är också att sökfunktionen är bra (dock kan det vara svårt att komma på översättningen då AAPS-appen är på svenska)

<img src="./images/search.png" width=240>

AAPS och installationsprocess är väldokumenterad, men jag tycker att den saknar en röd tråd för hur allt hänger ihop så jag tänkte förtydliga lite med ett exempel på hur jag har satt upp systemet.

# Förklaring av alla ingående komponenter
Här nedan ser du hur vi satt upp vårt system. Har inte mer än testkört NSClient-appen, då användaren i vårt fall är självgående, men tog med den som ett exempel.

![AAPS_system_overview](./images/AAPS_system_overview.png)

- [AAPS](#aaps)
- [Förklaring av alla ingående komponenter](#förklaring-av-alla-ingående-komponenter)
  - [Pumpar](#pumpar)
  - [CGM](#cgm)
  - [Nightscout](#nightscout)
  - [Hur installerar man en app som inte kommer från google](#hur-installerar-man-en-app-som-inte-kommer-från-google)
  - [Dexcom BYODA](#dexcom-byoda)
  - [Installera Nightscout](#installera-nightscout)
    - [Installera på Azure (bara hemsida, eller + databas)](#installera-på-azure-bara-hemsida-eller--databas)
    - [för att sätta upp på Heroku (inte längre aktuellt...)](#för-att-sätta-upp-på-heroku-inte-längre-aktuellt)
  - [Bygg AAPS](#bygg-aaps)
  - [Installationer på telefon](#installationer-på-telefon)
    - [Dexcom](#dexcom)
    - [AAPS](#aaps-1)
      - [Målen i AAPS](#målen-i-aaps)
  - [Klocka](#klocka)
  - [NSClient (för föräldrar/följare)](#nsclient-för-föräldrarföljare)
  - [xDrip+](#xdrip)
  - [OmnipodStash](#omnipodstash)
  - [Gadgets](#gadgets)
    - [M5Stack Nightscout](#m5stack-nightscout)
    - [NightStander](#nightstander)


## Pumpar
AAPS stödjer [flertalet pumpar](https://androidaps.readthedocs.io/en/latest/Hardware/pumps.html) varav några har stöd för blåtand så att de kan prata direkt med telefonen som kör AAPS, andra måste ha en länk mellan telefonen och pumpen (Rileylink, orangelink, emalink - de är samma länk-prylar som används av ios-loopare!):
- Accu-Chek Combo
- Accu-Chek Insight
- DanaR/RS/i
- Diaconn G8
- Omnipod Eros*
- **Omnipod DASH**
- Medtronic pumps*

*Behöver en Rileylink då de inte har bluetooth!

Vi har gått över till Omnipod DASH som har fått fullt stöd nu i AAPS 3.0. Fördelen med DASH över EROS är att den kommunicerar över blåtand och då behövs inte en Rileylink. Sitter du på Eros kan jag tipsa om att direkt ansöka om att få byta till DASH! Vi fick byta på en vecka trots att receptet hade långt kvar på Eros! Mycket bättre kontakt, en färre pryl att ha koll på/ladda och den återansluter jättebra vid tappad kontakt! Notera dock att [inte alla Android-telefoners](https://docs.google.com/spreadsheets/d/1zO-Vf3wv0jji5Gflk6pe48oi348ApF5RvMcI6NG5TnY) blåtand fungerar så bra med DASH's då den är väldigt energisnål"!

## CGM
Alla loop-system är beroende av en stabil och korrekt CGM (koninuerlig glukos-mätare) som levererar ett BG-värde minst var 5e minut. AAPS rekomenderar i nuläget Dexcom G6, då det är den som har tillräckligt stabil mätning och utjämnande algoritm för att kunna aktivera den mer aggresiva typen av loop (SMB). Det går att loopa med [de andra systemen också](https://androidaps.readthedocs.io/en/latest/Configuration/BG-Source.html), men då kommer AAPS inte tillåta användning av SMB (Super micro bolus) fullt ut, vilket vore tråkigt att vara utan - men funkar helt ok ändå, bara inte lika aggresivt. 

Nu för tiden rekomenderar utvecklarna av AAPS att vi använder den så kallade BYODA ("Bygg din egen Dexcom-app"), vissa använder xDrip för att hämta BG-värden från sändaren, men den har inte lika bra back-fill funktionsalitet som BYODA (vid missade värden) så vi har bara använt BYODA! En annan fördel med BYODA är att den fortfarande skickar upp värden till dexcoms servrar och då fortfarande fungerar med Diasend!. 

Jag kan ändå verkligen rekommendera att ni installerar xDrip+appen också då den har fantastiska "smarta" larm som kan konfigueras att agera olika beroende på dagar/tider och om BS är på väg åt rätt håll eller bara larma vid bestående hög mm. xDrip funkar både på loop-telefonen och på följar-telefoner (loop-telefonens xDrip får värden direkt från BYODA, följarnas hämtas via NS)

Vill du byta från ios loop till AAPS, så kan du börja med att köra dubbelt ett tag (upp till mål 5!?), då kör du virtuell pump och ställer in `NSClient BG` som din BG-källa, så laddas de i stället ner från NS. 



## Nightscout
Nightscout (NS) är en moln-baserad tjänst som sparar och visualiserar/tillhandahåller historisk data från ditt loop-system. AAPS skickar upp sina värden och beräkningar till NS var 5e minut och NS tar emot datan och lagrar den i en databas som du själv har satt upp och har full kontroll över (Mongodb i Atlas).

<img src="./images/NS_exempel_2.png" width=700>

AAPS i sig är inte beroende av någon extern databas/tjänst (NS), men eftersom de absolut flesta användarna vill kunna titta på historisk data (längre tillbaka än de 48h som AAPS håller) och ha möjlighet för t.ex. föräldrar att följa AAPS på distans, så är det i nuläget tvingande att ha en NS (Nightscout) i de första målen och när man installerar AAPS. Till skillnad från att logga in på Dexcom och titta på dina BG-värden, kommer du kunna se ALL data i NS så som måltider, bolusar, temporära basaler och profilbyten. 

Det är väldigt viktigt att den som loopar förstår hur loopen tänker, och att man förstår varför den gör som den gör i olika lägen! Det lär man sig absolut bäst genom att titta på historisk data i t.ex. NS-hemsidans rapport-verktyg där det framkommer väldigt tydligt hur AAPS har jobbat.

<img src="./images/NS_rapport_1.png" width=700>

Obs, jag har modifierat koden lite och bl.a. flyttat upp IOB och COB-graferna och ändrat lite text mm, så din NS-rapport kommer inte se ut precis som bilden ovan!

Sidan innheåller också lite övrig statistik:

- hba1c beräkning
- Tid i fluktuation
- etc...

<details>
  <summary><b>lite exempel på hur det kan se ut...</b></summary>

<img src="./images/NS_rapport_2.png" width=700>
<img src="./images/NS_rapport_3.png" width=700>
<img src="./images/NS_rapport_4.png" width=700>

Du kan även se historiken på de profiler du har haft vilket kan vara väldigt bra att se hur man har ändrat sina värden... (även här har jag ändrat i koden, så att alla timmar syns och de ändrade värdena färgläggs.)

<img src="./images/NS_rapport_5.png" width=300>

</details>

Vill du testa min moddade variant av NS, får du läsa i installations-steget hur du gör.

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
        - <img src="./images/AllowInstalation.png" width=200>


    - Dropbox funkar också, men tillåter inte direktinstallation från appen, då måste man först kopiera apk-filen till telefonens internminne (tre punkter/Exportera/Spara till Enhet) och sedan tillåta att t.ex. "Files by google" får installera appar (likt steget för Drive ovan).



## Dexcom BYODA
Den oficiella Dexcom-appen är låst till att bara fungera på vissa mobiler och den är också låst till att bara skicka sin data till Dexcom-share. Du behöver en Dexcom-app som förutom att skicka till Dexcom-share och diasend, också delar med sig av BG-värden till AAPS (och xDrip). För detta finns en patchad (hackad) variant som du själv [konfigurerar/bygger i ett google-formulär](https://docs.google.com/forms/d/e/1FAIpQLScD76G0Y-BlL4tZljaFkjlwuqhT83QlFM5v6ZEfO7gCU98iJQ/viewform?fbzx=2196386787609383750&fbclid=IwAR2aL8Cps1s6W8apUVK-gOqgGpA-McMPJj9Y8emf_P0-_gAsmJs6QwAY-o0) och sedan får en länk att ladda ner appen (apk-filen).

<details>
  <summary><b>När du fyller i detta formulär är det viktigt att du väljer rätt på dessa inställningar...</b></summary>

Tillåt att den installeras på ALLA android-telefoner

<img src="./images/BYODA_devices.png" width=500>

Versionen av dexcom som vi har i Europa:

<img src="./images/BYODA_version.png" width=500>

Skippa INTE de 2h warmup utan värden och tillåt att dexcom skikar värden internt inom telefonen till AAPS och eventuellt xDrip (välj detta även om du inte har xDrip till att börja med)

<img src="./images/BYODA_brodcast.png" width=500>

Resten är ganska själv-förklarande tror jag (använd default-värdena på de du är osäker på)

</details>

<br>

Du får nu ett mejl med en nedladdningslänk inom 5min som du laddar ner och lägger på Drive. OBS! Kolla skräpposten, där hamnade mitt mejl iaf! 

Under tiden så kan du passa på att ladda ner xDrip+ [här](https://xdrip-plus-updates.appspot.com/stable/xdrip-plus-latest.apk) som du också lägger på Drive (den kommer du vilja ha för larm!).

## Installera Nightscout 

### Installera på Azure (bara hemsida, eller + databas)

Obs! Heroku är inte längre gratis så det kan man fortsätta att använda för en ganska billig peng. Om man vill fortsätta använda NS på en gratis-tjänst, så kan man köra Azure. 
Jag installerade enl [den här youtube-tutorialen](https://www.youtube.com/watch?v=EDADrteGBnY&ab_channel=ScottHanselman) som var väldigt bra och gick igenom det mesta. Jag tog screenshots på den svenska versionen av azure för varje steg om nån vill följa: 
OBS! Jag följde guiden och skapade en ny databas på azure - **men struntade i att använda den** då jag istället konfigurerade den att använda vår gamla MONGODB_URI som pekar på Atlas-databasen, så att vi direkt fick tillgång till all historisk data och fortsätter använda den ist!

tips: klicka på bilderna för att öppna dom i ny flik i webläsaren om de är för små att se...

<details>
  <summary><b>Hur jag installerade NS på Azure (svenska screenshots)</b></summary>

För det första är det lite otydligt om det verkligen är gratis på azure-hemsidan, men om man klickar på sin plan, så kan man läsa att de tjänster vi kommer använda är bland de 40 som kommer fortsätta vara gratis även efter prövo-tiden. 

<img src="./images/ns/bild01.png" width=800>

1. Skapa ett [gratis azure-konto här](https://azure.microsoft.com/sv-se/) (grön knapp uppe till höger "kostnadsfritt konto")
   - Du kommer precis som heroku behöva fylla i ett kontokort - men allt kommer vara gratis om du följer instruktionerna
2. Skippa "azure-style databas" stegen om du vill fortsätta använda din Atlas-databas: 

<details> 
    <summary><b>azure-style databas (om du inte redan har en Atlas)</b></summary>

1. skapa en Cosmos DB 
   - Sök efter "Azure Cosmos DB for MongoDB":
  
<img src="./images/ns/bild00.png" width=800> <br>
<img src="./images/ns/bild011.png" width=500>
   
   - skapa ny "Resursgrupp" som du döper till ex "nightscout":
  
<img src="./images/ns/bild012.png" width=800>
   
   - Döp Kontonamn till nåt unikt ex "pellesnightscoutdb" 
   - välj Europa
   - Etablerat dataflöde
   - **Tillämpa fri nivå-rabatt**
   - Kryssa i **Begränsa totala dataflödet**
  
<img src="./images/ns/bild013.png" width=800>

    - Tryck nu blåa knappen **"Granska + skapa"**
  

1. Öppna din resursgrupp -> databasresurs och stäng välkommen-filmen

<img src="./images/ns/bild04.png" width=800><br>
<img src="./images/ns/bild06.png" width=800><br>
<img src="./images/ns/bild07.png" width=800>

3. Skapa en databas (var noga med att det är ifyllt enl nedan så att den är gratis!)

<img src="./images/ns/bild08.png" width=800>
<img src="./images/ns/bild09.png" width=500>

4. i din databas-vy tryck på "Snabbstart"/"Node js" för att få tillgång till din connection-string (MONGODB_URI)
   - Kopiera den nedersta "Node js 3.0" genom att trycka där röda pilen visar
   - spara undan i text-fil på datorn!
   - **VIKTIGT!** ändra maxIdleTImeMS till socketTimeoutMS antingen genom att manuellt skriva över, eller find & replace som jag visar nedan!

<img src="./images/ns/bild10.png" width=800>
<img src="./images/ns/bild11.png" width=800>
<img src="./images/ns/bild12.png" width=500>


</details>

<br>

3. Skapa en web-app
  - Välj din resursgrupp (om du inte har skapat en asure-databas ovan, tryck "Skapa ny" och döp till ex "nightscout")
  - Namnge din app (kommer vara del av sökvägen till NS-hemsidan)
  - Välj "Docker-container" och Linux
  - Välj region "North Europe"
  - Välj "Skapa ny" på Linux-plan och döp till ex "nightscoutplan"
  - Se till att välja "F1" (**den bruna gratis-planen!**)
  - Grå knapp **"Nästa Docker>"**

  
<img src="./images/ns/bild13.png" width=500>
<img src="./images/ns/bild14.png" width=800>
<img src="./images/ns/bild15.png" width=800>

  - Konfa vilken docker-image och källa enl nedan:
  - Enskild container
  - Docker Hub
  - nightscout/cgm-remote-monitor:latest
  - Tryck Blå knapp **"Granska + skapa"**
 
<img src="./images/ns/bild18.png" width=800>

3. granska din web-app å tryck "Skapa"

<img src="./images/ns/bild19.png" width=800>

4. Konfigurera Nightscout-variabler
   - Resursgrupp/nightscout/din web-app


<img src="./images/ns/bild20.png" width=800>

 - Tryck sen på Konfiguration

<img src="./images/ns/bild21.png" width=800>

 - Långsamma varianten: trycka på "Ny programinställning" och lägga till variablerna 1 och 1 (samma variabler som i HEROKU)
 - Snabba varianten: Tryck på Avancerad redigering och klistra in värden som du har ändrat i t.ex. notepad (nedan hittar du hur min ser ut)
<details> 
    <summary><b>Klicka här för att se mina Nightscout-inställningar (AAPS) som du kan modifiera och klistra in i din ruta</b></summary>

Observera att de 4 första var där by default - de lät jag vara kvar!

```
[
    {
        "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
        "value": "",
        "slotSetting": false
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_URL",
        "value": "https://index.docker.io",
        "slotSetting": false
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_USERNAME",
        "value": "",
        "slotSetting": false
    },
    {
        "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
        "value": "false",
        "slotSetting": false
    },
    {
        "name": "API_SECRET",
        "value": "VÄLDIGT HEMLIG API-SECRET SOM DU ANVÄNDER FÖR ATT LOGGA IN I NS",
        "slotSetting": false
    },
    {
        "name": "BASAL_RENDER",
        "value": "default",
        "slotSetting": false
    },
    {
        "name": "BG_HIGH",
        "value": "180",
        "slotSetting": false
    },
    {
        "name": "BG_LOW",
        "value": "45",
        "slotSetting": false
    },
    {
        "name": "BG_TARGET_BOTTOM",
        "value": "63",
        "slotSetting": false
    },
    {
        "name": "BG_TARGET_TOP",
        "value": "180",
        "slotSetting": false
    },
    {
        "name": "BOLUS_RENDER_FORMAT_SMALL",
        "value": "minimal",
        "slotSetting": false
    },
    {
        "name": "DEVICESTATUS_ADVANCED",
        "value": "true",
        "slotSetting": false
    },
    {
        "name": "DISPLAY_UNITS",
        "value": "mmol",
        "slotSetting": false
    },
    {
        "name": "EDIT_MODE",
        "value": "off",
        "slotSetting": false
    },
    {
        "name": "ENABLE",
        "value": "careportal basal dbsize rawbg iob maker cob bwp cage iage sage boluscalc pushover treatmentnotify mmconnect loop pump profile food openaps bage alexa override cors",
        "slotSetting": false
    },
    {
        "name": "MONGO_COLLECTION",
        "value": "entries",
        "slotSetting": false
    },
    {
        "name": "MONGODB_URI",
        "value": "mongodb://VÄLDIGTHEMLIGAORD HÄR KAN DU ANTINGEN ANVÄNDA ADRESSEN TILL ATLAS, ELLER TILL COSMOS HÄR I AZURE!nightscoutdb.mongo.cosmos.azure.com:10255/?ssl=true&retrywrites=false&socketTimeoutMS=120000&appName=@XXXXnightscoutdb@",
        "slotSetting": false
    },
    {
        "name": "OPENAPS_COLOR_PREDICTION_LINES",
        "value": "true",
        "slotSetting": false
    },
    {
        "name": "PUMP_FIELDS",
        "value": "reservoir battery clock",
        "slotSetting": false
    },
    {
        "name": "SCALE_Y",
        "value": "linear",
        "slotSetting": false
    },
    {
        "name": "SHOW_FORECAST",
        "value": "openaps",
        "slotSetting": false
    },
    {
        "name": "SHOW_PLUGINS",
        "value": "iob cob pump openaps iage basal dbsize",
        "slotSetting": false
    },
    {
        "name": "SHOW_RAWBG",
        "value": "never",
        "slotSetting": false
    },
    {
        "name": "THEME",
        "value": "colors",
        "slotSetting": false
    },
    {
        "name": "TIME_FORMAT",
        "value": "24",
        "slotSetting": false
    }
]
```

</details>

<br>
<img src="./images/ns/bild22.png" width=800>

<br>

- Glöm inte att trycka på **Spara** när du är klar!

<img src="./images/ns/bild24.png" width=800>


- Gå till översikten och tryck på URL-länken, så kommer din sida att laddas 
- Första gången så stog den bara och tuggade, då fick jag trycka på "Starta om" som man ser på bilden nedan: 

<img src="./images/ns/bild25.png" width=800>

Nu är det bara att gå in i AAPS/Loop/xdrip(följare)/NS-client/m5Stack mm och peka om till rätt URL (https://din-webb-app.azurewebsites.net) 

</details>

<br>


### för att sätta upp på Heroku (inte längre aktuellt...)
Installation av NS är väl beskrivet i detalj [här](http://nightscout.github.io/nightscout/new_user/), vill du i stället ha det berättat för dig, så har Jonas Hummelstrand gjort en [youtube-tutorial](https://youtu.be/rNIpmIhPCpU) på svenska där han går igenom processen steg för steg. OBS! Skippa allt som har med BRIDGE att göra (steg 13 och framåt), du ska INTE sätta upp Dexcom bridge då vi istället ska skicka upp all data direkt från AAPS. 

Vill du testa mina modifikationer, så får du forka från mitt repo [min fork av nightscout](https://github.com/klalle/cgm-remote-monitor) och deploya branchen "wip/customtest"

För att NS-hemsidan ska visa OpenAPS-prediktionerna och alla plugins vi är intresserade av, har jag laggt till vissa variabler i Heroku.
<details>
  <summary><b>Mina Heroku-inställningar...</b></summary>
Jag har laggt till ett gäng variabler så att NS-hemsidan startar med rätt inställningar från början i alla webläsare (så jag slipper gå in i menyn å klicka fram rätt saker som jag vill se...)

Förutom alla larminställningar (jag kör inte med larm i NS-web) så har jag: 

- API_SECRET = xxxxxxx
- BASAL_RENDER = default
- BG_HIGH = 180     (18*bg för att NS räknar mg/dl ist för mmol/L)
- BG_LOW = 45
- BG_TARGET_BOTTOM = 63
- BG_TARGET_TOP = 180
- BOLUS_RENDER_FORMAT_SMALL = minimal
- BRIDGE_USER_NAME = LÄMNAS BLANK! (eller skippa)
- **DEVICESTATUS_ADVANCED = true**
- DISPLAY_UNITS = mmol
- EDIT_MODE = off
- ENABLE = careportal basal dbsize rawbg iob maker cob bwp cage iage sage boluscalc pushover treatmentnotify mmconnect loop pump profile food **openaps** bage alexa override cors
- MONGO_COLLECTION = entries
- MONGODB_URI = mongodb+srv://dbanvändaren:lösenordet@klusternamn.el6di.mongodb.net/dbnamn?retryWrites=true&w=majority
- **OPENAPS_COLOR_PREDICTION_LINES = true**
- PAPERTRAIL_API_TOKEN = disabled (om du inte vill ha loggnig)
- PUMP_FIELDS = reservoir battery clock
- SCALE_Y = linear
- **SHOW_FORECAST = openaps**
- **SHOW_PLUGINS = iob cob pump openaps iage basal dbsize**
- SHOW_RAWBG = never
- **THEME = colors**
- TIME_FORMAT = 24

</details>

<br>

När du är klar med detta steg så har du fortfarande ingen data att visa, men det ska gå att komma åt NS-hemsidan i en webläsare!

<details>
  <summary><b>Lite tips hur jag använder NS...</b></summary>

Om du har aktiverat samma variabler i Heroku som jag har, så kommer det här redan vara ifyllt korrekt, men här är lite förtydligande: 

- Aktivera OpenAPS predictions (de är inte färglagda från början, kommer strax!)
- <img src="./images/NS_Web_3.png" width=400> <img src="./images/NS_Web_5.png" width=400>
  
  - Om du hovrar musen över OpenAPS-rutan (trycker på mobil) så visas massa bra info om varför AAPS har tagit det beslut det har tagit.
  - notera att min "OpenAPS"-ruta visar mer än standard då jag har moddat koden lite... 
- Öppna menyn (tre streck i högra hörnet)
  - Rapportverktyg är det jag använder absolut mest! (förutom grafen i huvudfönstret)
- <img src="./images/NS_Web_1.png" height=700> <img src="./images/NS_Web_2.png" height=700>
  - "Vård Portal" (särskrivning delux) kommer upp som ett litet kryss bredvid huvudmenyn (där kan man logga saker som faktiskt hamnar i databasen!)
    - Behöver inte användas så mycket då AAPS loggar det mesta själv, men här kan man t.ex. logga profilbyten jag tror att man kan få AAPS att byta till (om man är inloggad med token och har godkänt i AAPS)
  - Här ser du mot slutet också "Färgsätt progrosrader" som gör att du ser skillnad på prognos-graferna (denna finns inte om du inte först aktiverat OpenAPS enl ovan)

</details>

<br>

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

<details>
  <summary><b>Installationsguide...</b></summary>

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

<details>
  <summary><b>Klicka här för att visa screenshots av installationsprocessen</b></summary>

<img src="./images/instal_1_visning.png" width=250> <img src="./images/instal_2_nsclient.png" height=150> <img src="./images/instal_2_nsclient_2.png" width=250> <img src="./images/instal_3_insu.png" width=250> <img src="./images/instal_4_BG.png" width=250> 
<img src="./images/instal_5_dia.png" width=250> <img src="./images/instal_5_target.png" width=250><img src="./images/instal_5_CR.png" width=250><img src="./images/instal_5_done.png" width=250> 
<img src="./images/instal_6_profil.png" width=250> <img src="./images/instal_6_profil2.png" width=250> <img src="./images/instal_7_pump.png" width=250> <img src="./images/instal_8_ama.png" width=250>
<img src="./images/instal_9_loop.png" width=250> <img src="./images/instal_10_oref1.png" width=250>

</details>

</details>

<br>

Du hittar alla viktiga saker i de två menyerna i vardera övre hörnen. Jag kan inte gå igenom allt, du måste själv bekanta dig med var du hittar allt! Se [Preferences](https://androidaps.readthedocs.io/en/latest/Configuration/Preferences.html?highlight=preferences#preferences) för mer info!

Men här är lite kort info:
- "Hamburgermenyn" (de tre strecken uppe i vänstra hörnet) där du (förutom genvägar till vald pump/insu/cgm mm) når:
    - Mål - det här är där du måste klara av målen ([Objectives](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html) på engelska)
    - Underhåll - här exporterar du dina inställningar (Gör det efter varje klarat mål, och lägg på Drive så att du kan återskapa var du var i målen om mobilen dör!)
    - [Konfiugreringsverktyget](https://androidaps.readthedocs.io/en/latest/Configuration/Config-Builder.html)
     inställningar (kugghjul) samt väljer vilka flikar som ska synas (gröna ögat -> tryck i checkboxen så aktiveras en ny flik i huvudfönstret)
- Högermenyn (tre prickar uppe till höger) här nås massor av inställningar 

<img src="./images/overview.png" width=250> <img src="./images/hamburger.png" width=250> <img src="./images/konfigverktyg.png" width=250>

Läs sen in dig på [AndroidAPS screens](https://androidaps.readthedocs.io/en/latest/Getting-Started/Screenshots.html) för att förstå allt som visas i appen.

Nu är det bara att börja jobba dig igenom Målen ett efter ett (läs snälla på om varje mål! finns länkar vid varje mål i appen och du har ju länken till "[Objectives](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html)"). När du är klar kommer du ha en vältrimmad loop som du förhoppningsvis vet hur du ska justera när insulinkänsligheten förändras. 

#### Målen i AAPS
[Målen 1-11](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html) tar dig från genom alla steg för som gör att AAPS får mer och mer mandat att ta egna beslut och ge kraftigare och kraftigare korrigeringsdoser. Mål 3 är det som de flesta fastnar på och har frågor om, så jag tänkte förklara lite: 

<details>
  <summary><b>Klicka här för att expandera Mål-sektionen</b></summary>>

- **Mål 1**
    - Valt din profil
    - Fått BG-data från sensor
    - Har kontakt med NS
    - <img src="./images/obj_1.png" width=250>
- **Mål 2**
    - Testa lite funktionalitet genom att:
        - Sätta temporär profil
        - Koppla från pumpen å åtekloppla efter 1h (obs, detta är ett mål som inte är så logiskt när man kör omnipod, men du behöver ju inte ta av dig pumpen ändå, bara koppla från anslutningen till AAPS...)
        - Skapa ett TT (Temporärt BG-mål)
        - Ändra lite vad som syns i AAPS
- **Mål 3**
    - Bevisa din kunskap - Här är det många som fastnar då vissa frågor är lite konstiga. Jag reder ut de som jag tycker är knasiga:
    - **insulinkänslighetsfaktor (ISF)**
        - *ISF bör anges i dina AndoidAPS-inställningar*... 
            - hint: profilen räknas inte som AndroidAPS-inställningar
    - **Profilbyte**
        - *ISF kommer att bli 10% högre*
            - Jag hävdar att de har räknat fel här... ISF/0.9 blir inte 10 % högre, men det blir åtminståne inte lägre - så svara med en avrunding så blir det rätt!
    - **Brusiga CGM-värden"**
        - *Inaktivera closed loop-läge för att undvika över- eller underdosering*
            - Ja, på pappret, men frågan är hur många som gör det... men visst - tänk JÄTTE-brusiga värden som får loopen att gå bananas.
        - *Kontrollera att din CGM-app ger utjämnande data.*
            - Den här frågan är nog inte utformade för BYODA-användare... om man däremot använder xDrip för att läsa ut BG, så finns valet att använda Dexcoms utjämnande algoritm, och vet man det så blir frågan med logisk
    - **Insticksprogram för känslighet**
        - *Insticksprogram för känslighet ger användaren föreslagna ändringar av basaldoser, KH-kvoter och insulinkänslighetsfaltprer som kan användas för att redigera profilen.*
            - läs: ger autosens förslag på nya värden för IC, ISF och basal?
        - *Vissa av insticksprogrammen har konfigurerbara tidsintervall som kan ställas in av användaren*
            - Inte helt säker på vad de är ute efter här? kanske en rest från AAPS<2.8 ("In versions prior to AAPS 2.7 user had to choose between 8 or 24 hours manually.")?
    - **Aktivt Insulin (IOB)**
        - Hint IOB är allt insulin i kroppen utöver det som är basal!
- **Mål 4**
    - Öppen loop
        - Du får förslag av AAPS som du manuellt får godkänna - du måste agera på **minst 20st** för att bli godkänd och hålla på **minst 7 dagar**.
        - Ställ in ett högre spann som BG-målvärde så slipper du får så många "tips" på temporära basal-ändringar... 
    - Gå igenom alla inställningar så att det ser bra ut
- **Mål 5**
    - Förstå din öppna loop (från mål 4)
        - Du uppmanas titta på data/statistik-sidorna som finns i AAPS
        - tips: kolla på NS-hemsidans report också!
    - **Målet har inget krav** - bara en uppmaning att inte gå vidare förän du förstått och labbat dig fram till bra värden.
- **Mål 6**
    - Öppen loop med "Low Glucose Suspend" i **minst 5 dagar** ("Aktiverad funktion att stänga av vid lågt BG")
        - Så typ en halv stängd loop
        - AAPS får utan manuella åtgärder sänka eller helt stänga av basalen när den förutspår att du kommer bli låg (gå under target) men den får inte öka basalen om du inte har negativ IOB.
        - AAPS får alltså inte ge extra insulin (ökad basal) så länge IOB >= 0
    - se upp för höga värden efter du varit låg, då AAPS kan ha kört på zero-temp (noll basal) när du förutspåts bli låg. 
    - När du loopar om du inte gjort det förut, kommer du inse att du behöver mycket mindre druvsocker när du är låg, eftersom basalen automatiskt har varit avstängd långt innan du blev låg (när den förutspådde det) - så slå en kik på IOB när du är låg å notera hur hög du blir av korrigerings-kh!
    - Tips, i graf 1: Aktivera IOB och COB så ser du dessa kurvor på varandra i lilla grafen under huvudgrafen (tryck på pilen uppe i stora grafens hörn och aktivera graf 1, sen öppna samma pil igen och välj IOB och COB)
- **Mål 7**
    - Hurra nu börjar det roliga med stängd loop!
    - Höj sakta din MaxIOB från 0 som i mål 6, till det du beräknar enl dokumentationen (står under objectives)
    - Tips är att nu ställa in ett målvärde istället för ett spann så kommer aaps konstant att försöka korrigera dig var 5e minut för att nå mot målet.  
      - Gör det genoma att speca spannet som ex: 5,5-5,5...
    - När du känner att du fått koll på dina värden och du har testat i **minst 1 dag** - gå vidare
- **Mål 8**
    - Aktivera Autosens som automatiskt försöker se om din insulinkänslighet ändras över tid och då korrigerar det genom att sätta en %-sats på din basal + ISF 
        - OBS! IC påverkas inte! jag har försökt fråga varför, men inte fått gehör...
    - Kör med autosens i **minst 7 dagar**
- **Mål 9 Borttaget i 3.0!!!**
    - ~~Aktivera AMA "Advanced meal assist" som tillåter att systemet ger snabbare insulintillförsel om du matar in dina kh korrekt~~
    - ~~läs på om AMA och tweeka dina inställningar så att det funkar bra vid måltider~~
    - ~~Kör **minst 28 dagar**~~
- **Mål 10**
    - Aktivera oref1 (SMB) som ger mikrobolusar istället för (och tillsammans med) basaländringar för att snabbare korrigera dina svängningar. 
    - Kör **minst 28 dagar**
- **Mål 11**
    - Automationer
    - Nu har du tillgång till allt!

</details>




## Klocka
Klockan kan förutom att visualisera aktuell status på dina värden, också styra AAPS: 
- TT (temporära target)
- Bolus-kalkylatorn
- eCarbs
- vanliga bolus+carbs
- profilbyte

<img src="./images/watch2.png" height=350>

Smart-klockor måste numera byggas separat på samma sätt som du bygger AAPS (man väljer bara en annan "Module" i byggsteget). läs [här](https://androidaps.readthedocs.io/en/latest/Hardware/Smartwatch.html) om aaps och smartwatch.
Där står bland annat att du måste installera appen med "Wear installer", se [youtube-tutorial](https://www.youtube.com/watch?v=8HsfWPTFGQI). 

<img src="./images/watch.png" height=350>

## NSClient (för föräldrar/följare)
NSClient används för att monitorera och styra AAPS från en följartelefon. Det är en strippad version av AAPS, så den ser ut väldigt mycket som AAPS, men saknar funktionalitet. Om man i AAPS godkänner att NSClient får lov att göra ändringar, så kan man från NSClient-appen göra profil-byten och sätta TT (temporära target = målvärden). 

All övrig styrning går att sköta via [sms-komandon](https://androidaps.readthedocs.io/en/latest/Children/SMS-Commands.html?highlight=sms). Telefonnumret måste vara aktiverat i AAPS och en 2-faktors auktorisering rekomenderas (krävs?). 

Obs! NSclient-appen funkar bara på Android-telefoner, men det går utmärkt att som följare köra iOS och ha nån anna nightscout-klient / [xDrip4iOS](https://xdrip4ios.readthedocs.io/en/latest/connect/follower/) och sköta remote-styrningen via sms! Jag har sett att folk använder 2-faktors-apparna Authy eller 2FAS Auth.

Läs på om [Remote monitoring med AAPS](https://androidaps.readthedocs.io/en/latest/Children/Children.html)

<img src="./images/remote.png">

NSClient är en app som (oftast) inte behöver byggas själv, utan släpps med senaste releasen av AAPS och kan laddas ner från [github](https://github.com/nightscout/AndroidAPS/releases/) där du hittar två byggda apk-filer (för att kunna styra 2 barn med olika AAPS). För över med Drive och installera som de övriga apparna. 

<img src="./images/nsclientapk.png">
 
## xDrip+
Installera xDrip+ från apk-filen du laddat ner i tidigare skede. 
Gå till inställningar (övre vänstra hörnets tre streck/Inställningar) och börja med att välja "Hårdvarudatakälla" (BG-källa).
- Telefonen med BYODA och AAPS: välj `640G / EverSense`
- Alla följartelefoner: Välj `Nightscout Follower` och fyll i adressen till din NS-site (inkl `https://`) under "Nightscout Follow URL" som kommer upp under "Hårdvarukälla" i menyn. 
- Om du vill kan du aktivera "Download Treatments" för att även se insulin och kh i xDrip, men jag har inte det då jag bara använde xDrip för larm.

<img src="./images/xdrip_BYODA.png" width=250><img src="./images/xdrip_NS.png" width=250>

Gå till Inställningar/`Larm och varningar` (näst högst upp) och börja ovanifrån

<img src="./images/xdrip_inst.png" width=250> 
<details>
  <summary><b>Här är lite tips på bra larm du kan aktivera...</b></summary>>

- "Lista över glukosnivåvarningar"
    - Här skapar du larm för låga/höga BG-värden
    - Larmen kan vara olika för olika dagar/tider och du väljer själv melodi osv.
    - OBS! jag hade förut problem med att fylla i ett värde då xdrip inte tillåter mig att skriva "," men vägrar godta "."... skriv då på ett annat ställe i telefonen och kopiera/klistra in värdet (t.ex "3,5") i rutan
    - <img src="./images/xdrip_BG.png" width=250> <img src="./images/xdrip_BG_low.png" width=250>
- "Glukosvarningsinställningar"
    - "Intelligent snooze" & "Intelliganta larm" - Fantastiskt att slippa få larm om värdena ändå går åt rätt håll!
    - <img src="./images/xdrip_glukvarninst.png" width=250> 
- "Varning för missade avläsningar" 
    - Bra att ha för t.ex. följare!
- "Andra varningar" 
    - "Bg falling fast" 
    - "Bg rising fast"
    - <img src="./images/xdrip_BG_fallfast.png" width=250> 
- "Ytterligare varningar (xDrip+)"
    - "Larm vid beständigt högt" - Använde jag som följare för att slippa få larm bara för att det blev högt ett litet tag
    - <img src="./images/xdrip_larmBest.png" width=250> 

</details>

Det finns säker fler larm som är praktiska än de jag visat ovan. 

## OmnipodStash
Jag har byggt en liten applikation som man kan installera bredvid NS och som håller koll på lagret av poddar/sensorer och insulin man har kvar där hemma. När det börjar ta slut så får du ett mejl som påminner om att beställa nya.

 Poddar och sensorer registrerar AAPS till NS vid byten, så de räknas ner automatiskt - insulinflaskor får du registrera manuellt när en tagit slut.

Installationsbeskrivning finns [här](https://github.com/maja-lofgren/omnipod_stash)

<img src="./images/omnipodstash.png"> 



## Gadgets
### M5Stack Nightscout
Vi har kört en [M5Stack Nightscout](https://www.facebook.com/groups/606295776549008) som är en lite rolig pryl som visar data från NS och kan larma (som en sängklocka eller på arbetsbordet)

<img src="./images/m5stack.png" height=200><img src="./images/m5stack_2.png" height=200>

### NightStander
Daniel Mini Johansson i Looped - Sweden har utvecklat [NightStander](https://github.com/MiniJoko/NightStander) - en egen pryl som kan visa NS-data och har lite funktioner som kan prata med åtminstone ios-loop (vet inte hur de lirar med AAPS!?)

<img src="https://github.com/MiniJoko/NightStander/blob/main/resources/img/product.png" height=200>