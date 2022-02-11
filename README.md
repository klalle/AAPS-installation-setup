# AAPS-installation-setup
AAPS och installationsprocessen är [väldokumenterad](https://androidaps.readthedocs.io/en/latest) på engelska, men jag tycker att den saknar en röd tråd för hur allt hänger ihop, så jag tänkte förtydliga lite med ett exempel på hur jag har satt upp systemet.

## CGM
Alla loop-system är beroende av en stabil och korrekt CGM (koninuerlig glukos-mätare) som levererar ett BG-värde minst var 5e minut. AAPS rekomenderar i nuläget Dexcom G6, då det är den som har tillräckligt  stabil mätning för att kunna lita på. Det går att loopa med [de andra systemen också](https://androidaps.readthedocs.io/en/latest/Configuration/BG-Source.html), men då kommer AAPS inte tillåta SMB (Super micro bolus), vilket vore synd att vara utan - men funkar helt ok ändå, bara inte lika aggresivt. 

## Nightscout
Nightscout (NS) är en moln-baserad tjänst som sparar och visualiserar/tillhandahåller historisk data från ditt loop-system. AAPS skickar upp sina värden och beräkningar till NS var 5e minut och NS tar emot datan och lagrar den i en databas som du själv har satt upp och har full kontroll över (Mongodb i Atlas).

AAPS i sig är inte beroende av någon extern databas/tjänst (NS), men eftersom de absolut flesta användarna vill kunna titta på historisk data (längre tillbaka än de 48h som AAPS håller) och ha möjlighet för t.ex. föräldrar att följa AAPS på distans, så är det i nuläget tvingande att sätta upp NS (Nightscout) att i de första stegen när man installerar AAPS.

Installation av NS är väl beskrivet på och Jonas Hummelstrand har lagt upp en [youtube-tutorial](https://youtu.be/rNIpmIhPCpU) på svenska om hur man sätter upp NS med mongodb på Atlas. OBS! Skippa allt som har med BRIDGE att göra (steg 13 och framåt), du ska INTE sätta upp Dexcom bridge då vi istället ska skicka upp all data direkt från AAPS. 

## Installation CGM
Den oficiella Dexcom-appen är låst till att bara fungera på vissa mobiler och den är också låst till att bara skicka sin data till Dexcom-share. Det finns en patchad (hackad) variant som du själv konfigurerar i ett google-formulär och sedan får en länk att ladda ner appen (apk-filen).
Så dy fyller i



- BG-värde
- basal
- temporär basal (loopen ändrar din basal kontinuerligt)
- ISF (insulinkänslighetsfaktor)
- DIA (duration of insulin active som är )
- IOB (insulin on bord => hur många enheter mer än basalen har du i kroppen)
- COB (carbs on bord => hur många gram kolhydrater har du just nu i kroppen, räknas automatiskt ner utifrån hur din BG-kurva förändras)
- Profiler/profilbyten (AAPS jobbar med profiler som kan ha olika basal/ISF/IC) 
![AAPS_system_overview](./images/AAPS_system_overview.png)