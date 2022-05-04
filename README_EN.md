# AAPS
AAPS (AndroidAPS) has a large focus on safety and will NOT let you enable an automated algorithm with mandate to inject insulin in your body from day one (in contrast with iOS-loop/FAX). You will instead start a jurny that takes you through a couple of objectives that you have to complete one by one, that enables more and more advanced functionallity and in the end gives AAPS full access to give you insulin when it predicts you need it. 

Make sure you read the [documentation](https://androidaps.readthedocs.io/en/latest) thorougly so you fully understand how the app works and can adjust the parameters in a safe manar. 
When you get stuck and need help with something, post a question in the [AndroidAPS users](https://www.facebook.com/groups/AndroidAPSUsers) facebook group. 

An advice is to use the search function of the documentation, that will usually find what you are looking for. 

<img src="./images/search.png" width=240>

AAPS and the installation process are well documented, but in the beguining I had a hard time understanding how everything was connected. This is an overview of my setup as an example that might inspire you and hopefully make things a little less confusing.


# Explenation of the setup
Here you see a chart of our setup, I haven't mort than tested the NSClient-app since the looping person in our case is independent. 

![AAPS_system_overview](./images/AAPS_system_overview.png)

- [AAPS](#aaps)
- [Explenation of the setup](#explenation-of-the-setup)
  - [Pumps](#pumps)
  - [CGM](#cgm)
  - [Nightscout](#nightscout)
  - [How to install an app that's not on google play](#how-to-install-an-app-thats-not-on-google-play)
  - [Dexcom BYODA](#dexcom-byoda)
  - [Install Nightscout](#install-nightscout)
  - [Build AAPS](#build-aaps)
  - [Installations on phone](#installations-on-phone)
    - [Dexcom](#dexcom)
    - [AAPS](#aaps-1)
      - [Objectives in AAPS](#objectives-in-aaps)
  - [Watch](#watch)
  - [NSClient-app (for parents/followers)](#nsclient-app-for-parentsfollowers)
  - [xDrip+](#xdrip)
  - [OmnipodStash](#omnipodstash)
  - [Gadgets](#gadgets)
    - [M5Stack Nightscout](#m5stack-nightscout)


## Pumps
AAPS supports [several different pump models](https://androidaps.readthedocs.io/en/latest/Hardware/pumps.html) where some of them has bluetooth support and can communicate straight to the phone running AAPS. Other pumps need a link device (Rileylink, orangelink, emalink) in between the pump and the phone. The link devices are same for iOS-loop and AAPS!

Supported pumps:
- Accu-Chek Combo
- Accu-Chek Insight
- DanaR/RS/i
- Diaconn G8
- Omnipod Eros*
- **Omnipod DASH**
- Medtronic pumps*

*Needs a link-device!

We started using Omnipod DASH in dec 2021 in dev-mode, which is now since v3.0 fully supported in AAPS. The benefit over Eros is the bluetooth comunication which makes for a much mor stable connection than with a link-device! If you plan on moving over to DASH - make sure your phone is not one of the red-listed ones in the [DASH-phone spreadsheet](https://docs.google.com/spreadsheets/d/1zO-Vf3wv0jji5Gflk6pe48oi348ApF5RvMcI6NG5TnY) (some phones BT has issues)


## CGM
All loop systems are dependent of a s table and correct CGM (continous glucose meter) that delivers a BG-value at least every 5th minute. AAPS is recomending the Dexcom G6 since it has the smoothest BG-curve-algorithm and is very reliable. AAPS supports [other CGM's](https://androidaps.readthedocs.io/en/latest/Configuration/BG-Source.html), as well but then it won't allow the most aggressive insulin delivery algoritms (SMB always).

You cannot use the stock Dexcom-app, so there are two alternatives. Recomended app to use is BYODA (BuildYourOwnDexcomApp) which is a hacked version of the original app, but you can also use the most resent version of xDrip+ (but then you wont have the back-smoothing). Another argument for using BYODA is if your dia-team is used to evaluating data from the dexcom database (or diasend) which will still work with BYODA but not xDrip.

Even when using BYODA as a BG-source, I still recomend installing the xDrip+-app as an alarm app since it has a lot of nice alarms that AAPS/BYODA lacks. It is also very good to have on the follower phones. xDrip on loop-phone will get its data directly from BYODA, xdrip on follower phone will collect it from Nightscout.

If you are currently looping on iOS-loop and want to try out AAPS, then you can start using AAPS simontaniously up to objective 5. Then you select `NSClient BG` as a BG-source and selects the virtual pump to download the BG-values from Nightscout instead of getting them from BYODA.


## Nightscout
Nightscout (NS) is a cloud-based service that saves and visualizes/serves historical data from your loop system. AAPS is sending up its values and calculations to NS every 5 minutes and NS receives the data and stores it in a database that you have set up your selves on a different cloud (Mongodb on Atlas).

<img src="./images/NS_exempel_2.png" width=700>

AAPS is actually not dependent on any external service/database like NS, but since most users wants to view ther historical data (more than the 48h that AAPS holds) and to also have the ability for a follower to view the data live (like NSClient or NS-website), it is mandatory to set up NS to pass the first objective in AAPS. The benifit of NS compared to viewing data on Dexcom, is that you'll see all the data in NS (meals, boluses, temporary basals/targets and profile switches etc.).

It is very important that you understand how the loop works, and that you understand why it takes its desissions in different scenarios! One of the best way of learning this is to analyse historical data - and for that the NS-website has a superb tool called the repport where you see everything of imprtance. 

<img src="./images/NS_rapport_1.png" width=700>

Obs, I have modified the NS-code a bit, i.e. moved the IOB, COB-graphs and changed texts etc. so your NS-repport wont look exactly like the picture above. 

The NS-repport site also contains more interesting information such as:

- hba1c calculation
- Time in fluctuation
- etc...

<details>
  <summary><b>Example screenshots of the NS-report site...</b></summary>

<img src="./images/NS_rapport_2.png" width=700>
<img src="./images/NS_rapport_3.png" width=700>
<img src="./images/NS_rapport_4.png" width=700>

You can also see the history of all the profile changes that you have done under the "Profiles"-tab. I find this very helpful when evaluating former changes and outcomes. Note that I have changed the apperarence of the profiles-table as well (highlighted altered values and show all hours etc.)

<img src="./images/NS_rapport_5.png" width=300>

</details>

If you want to try my modifications to the NS-site, there are instructions under the installation step further down.

## How to install an app that's not on google play
1.  Turn off "Play protect"
    - Play protect should be kept off, since it will try to turn off your apps every night.
    - Settings/Security/App safety/Cog wheel (top right)
        - Turn off both "Search apps" and "enhance app identification"
    - Use settings search-function to find Play protect if your phone has this setting on a different location.
2. Allow installation from different source
    - Several of the apps we need will not be installed from the official google play. To be able to install these apk-files, you need to give permission for on of your existing apps to install unoficiall apps. In this exampl I'll give this permission to "Drive" (google drive). since it is easy to build the app on a computer, and just upload it to Drive, and in the Drive-app on the phone just install the file.
    
    - On my phone (Google pixel 4a with android 12) this setting is located: 
        - Settings/Apps/Specific access/Install unknown apps
        - Use settings search-function if you cant find it (or google)
        - Choose "Drive" in the list and give it permission to install other apps.
        - <img src="./images/AllowInstalation.png" width=200>


    - Dropbox also works, but doesn't allow direct installation of apk-files so you'd have to first copy the apk-file to your phones internal storage (three dots on file/Export/save to unit) then allow i.e. "Files by google" to install unknown apps (like the steps above for Drive)


## Dexcom BYODA
The oficial Dexcom-app is locked to a fiew phone models, and it is also only brodcasting its values to the Dexcm-share. What we need is a Dexcom app that also brodcasts its BG-readings to AAPS (and xDrip). For this, we have the BYODA which is a hacked version that you have to download by filling in [this form](https://docs.google.com/forms/d/e/1FAIpQLScD76G0Y-BlL4tZljaFkjlwuqhT83QlFM5v6ZEfO7gCU98iJQ/viewform?fbzx=2196386787609383750&fbclid=IwAR2aL8Cps1s6W8apUVK-gOqgGpA-McMPJj9Y8emf_P0-_gAsmJs6QwAY-o0) that will give you a link do download the app (apk-file) within a couple of minutes.

Note that the email is often cought in trash mail!

While waiting for the email - you can download latest stable xDrip+ from [here](https://xdrip-plus-updates.appspot.com/stable/xdrip-plus-latest.apk). Upload the apk-file to your Drive as well!

## Install Nightscout
Installation of NS is described in detail [here](http://nightscout.github.io/nightscout/new_user/), OBS! Do not enable Bridge since we will get the BG-values from AAPS instead of dexcom share!

If you want to try my modified version of NS, you'll have to fork from my repo instead of nightscout-github: fork [my fork of nightscout](https://github.com/klalle/cgm-remote-monitor) and choose to deploy the branch named "wip/customtest" on Heroku.

To automate the visual setting/plugins I want to show on Heroku, I have added a fiew variables in Heroku.
<details>
  <summary><b>My Heroku-settings...</b></summary>
The variables shown in bold bellow are there to automatically enable/show plugins associated with AAPS so I don't have to enable them manually every time I open NS on a different device.

I'm not using the alarm functionallity, so I wont show them here, but these are my variables: 

- API_SECRET = xxxxxxx
- BASAL_RENDER = default
- BG_HIGH = 180     (18*bg to convert to mmol/L)
- BG_LOW = 45
- BG_TARGET_BOTTOM = 63
- BG_TARGET_TOP = 180
- BOLUS_RENDER_FORMAT_SMALL = minimal
- BRIDGE_USER_NAME = LEAVE BLANK! (or skip)
- **DEVICESTATUS_ADVANCED = true**
- DISPLAY_UNITS = mmol
- EDIT_MODE = off
- ENABLE = careportal basal dbsize rawbg iob maker cob bwp cage iage sage boluscalc pushover treatmentnotify mmconnect loop pump profile food **openaps** bage alexa override cors
- MONGO_COLLECTION = entries
- MONGODB_URI = mongodb+srv://dbuser:password@klusternamn.el6di.mongodb.net/dbname?retryWrites=true&w=majority
- **OPENAPS_COLOR_PREDICTION_LINES = true**
- PAPERTRAIL_API_TOKEN = disabled (if you dont want logs)
- PUMP_FIELDS = reservoir battery clock
- SCALE_Y = linear
- **SHOW_FORECAST = openaps**
- **SHOW_PLUGINS = iob cob pump openaps iage basal dbsize**
- SHOW_RAWBG = never
- **THEME = colors**
- TIME_FORMAT = 24

</details>

<br>

When you're done with this step, you still haven't got any data displayed on your NS-site, but you will be able to brows to the site to see an empty graph!

<details>
  <summary><b>What I use NS-website for...</b></summary>

If you have activated the same variables as I have listed above, these settings will already be filld in, but here is an explination of the manual settings anyway:

- Activate OpenAPS predictions:
- <img src="./images/NS_Web_3.png" width=400> <img src="./images/NS_Web_5.png" width=400>
  
  - If you houver the mouse over OpenAPS-box (press it on mobile device) you will get a popup with lots of good info of why AAPS has taking the current desission. 
  - notice that my OpenAPS-box shows more information than standard code sice I altered the code a bit... 
- Open the menu (three lines, upper right quorner)
  - "Report" is the tool I use the most! (exept virwing the main graph)
- <img src="./images/NS_Web_1.png" height=700> <img src="./images/NS_Web_2.png" height=700>
  - "Care portal" if selected it adds a "+" sign in top right quorner (where you can post events to the actual NS-database!)
    - I dont use it at all, but you can use it to add profile switches etc.
</details>

<br>

## Build AAPS
To really be clear about this app being DIY and your responsibility, you have to download the code from github and build it on your own computer. I'm not going to cover all build steps since there is a nice step by step manual [here](https://androidaps.readthedocs.io/en/latest/Installing-AndroidAPS/Building-APK.html), but a lot of people get stuck on one of the last steps `Build the app`, so here is an explanation: 

In the step where you have built the app in AndroidStudio and looks in the "Event log", it will NOT say that you have built "1 variant" with a link... What you want to check for is that it's not saying: 

`Generate Signed APK: Errors while building APK. You can find the errors in the 'Messages' view.`

a successful build would output:

`Generate Signed APK: APK(s) generated successfully for module 'AndroidAPS.app' with 0 build variants:` 

Now you have to localize the apk-file that will be located in the folder that you downloaded the code to (from github). In the code folder, you now find a folder called "app" where you find:
`app/full/app-full-release.apk`. 

Upload `app-full-release.apk` to Drive and now you can from here on focus on the telefon installation process.

## Installations on phone
Now the fun part!
Prerequirements: you have Drive on the phone (or at least access to the apk-files somehow).

### Dexcom
Link to documentation [here](https://androidaps.readthedocs.io/en/latest/Hardware/DexcomG6.html#if-using-g6-with-build-your-own-dexcom-app).
- If you are running the original dexcom-app and has an active session with sensor+transmitter, you can simply switch to BYODA without loosing any of them!
    - Open the original Dexcom-app and write down the current sensor-code and transmitter serial number.
    - Deinstall dexcom-app
    - Install the new BYODA by clicking the apk-file.
    - log in as usual and fill in your transceiver serial (if I remember correctly you wont need the sensor code)
    - Then the documentation sais (not sure I did this step)
        - In phone settings go to apps > Dexcom G6 > permissions > additional permissions and press ‘Access Dexcom app’.
    - After a while, the app will find the sensor and start logging values exactly like before. Only difference is that it also brodcasts them to AAPS/xdrip. 

### AAPS
Install AAPS by clicking `app-full-release.apk`-file in Drive
I think you'll start the "Installation guide" automatically (or else you'll find it in the  3-dot-menu in top right quorner)

<details>
  <summary><b>Installation guide...</b></summary>

- All settings can be changed at a later stage, so do not panic if you dont know if you select correct or not from the start. You will NOT be able to automatically inject insulin with AAPS before you have come a fair bit into your objectives even if you selected SMB in the settings for example.
- Have a look at the screenshots below (swedish...)
- Accept all the permissions that AAPS asks for.
- Choose a master password that you write down!
- You will get notified that "no profile selected" - just ignore it!
- Visualization settings has nothing to do with the loop algoritm - its just for the green area on the graph!
- NSClient - click "Aktivera NSClient" and alter your NS-address (inkl `https://`) and verify that you have access (Status: `Authenticated (RWT)`)
- Select insulin-type (notice that your DIA wont be as short as you are used to!)
- Select BG-source (dexcom) and  `upload BG-data to NS`
    - Note! common to not receive BG-values in AAPS now, if so: select a different BG-source, and then re-select dexcom to trigger a popup asking for promision.
- Profile, AAPS works with profiles that you can swap between that stores: 
    - IC (insulin to carb ratio, how many gram of carbs does one unit of insulin take care of)
    - ISF (insulin snsitivity factor, how much will your BG drop from one unit of insulin)
    - basal
    - Target (BG-Target(range))
    - Note! the loop is expecting these values to be correct and will base all calculations on them, so make sure you test them on a regular basis or when you notice strange behaviour from the loop. The values are rarly constant for all hours of the day...
- Profile switch (AAPS wants you to make a first profile switch so that your newly created profile gets activated. Just press the button to make the profile switch / "OK"
- Pump - You can start by choosing "virtual" pump first, and switch to you actual pump in a later stage. You wont need a real pump connected until Object 6, but it's totaly fine to connect your pump as soon as you want (you wont need the pdm any more when AAPS is in control)
- APS - What algoritm AAPS will use - In a later stage you will use SMB, but you wont be able to use it until a much later stage, so start with AMA as the text states.
- APS-status - start with open loop (You manually have to okay the proposed treatments by AAPS. You wont be able to enable closed loop anyway...)

<details>
  <summary><b>Expand to show screenshots of the installation (in swedish...)</b></summary>

<img src="./images/instal_1_visning.png" width=250> <img src="./images/instal_2_nsclient.png" height=150> <img src="./images/instal_2_nsclient_2.png" width=250> <img src="./images/instal_3_insu.png" width=250> <img src="./images/instal_4_BG.png" width=250> 
<img src="./images/instal_5_dia.png" width=250> <img src="./images/instal_5_target.png" width=250><img src="./images/instal_5_CR.png" width=250><img src="./images/instal_5_done.png" width=250> 
<img src="./images/instal_6_profil.png" width=250> <img src="./images/instal_6_profil2.png" width=250> <img src="./images/instal_7_pump.png" width=250> <img src="./images/instal_8_ama.png" width=250>
<img src="./images/instal_9_loop.png" width=250> <img src="./images/instal_10_oref1.png" width=250>

</details>

</details>

<br>

You'll find all the important settings in the two menus in each of the top quorner. I wont go through all of the settings, you will have to get familiar with them by clicking around your self. See [Preferences](https://androidaps.readthedocs.io/en/latest/Configuration/Preferences.html?highlight=preferences#preferences) for more info. 

Short: 
- "Hamburger-menu" (three lines, top left quorner) is where you (except shortcuts to selected pump/insu/cgm etc) accesses:
    - [Objectives](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html) - that you have to complete one by one.
    - Maintenance - here you can export your settings (do this after each completed objective and copy it to i.e. Drive - so you can restart where you were if the telefone dies!)
    - [Config builder](https://androidaps.readthedocs.io/en/latest/Configuration/Config-Builder.html)
        - Access settings (cog wheel) and select which plugins you want to show as tabs in main window (green eye-checkbox)
- [Preferences](https://androidaps.readthedocs.io/en/latest/Configuration/Preferences.html?highlight=preferences#preferences) (three dots, top right) Lots of settings you have to go through!

<img src="./images/overview.png" width=250> <img src="./images/hamburger.png" width=250> <img src="./images/konfigverktyg.png" width=250>


Then you should check out  [AndroidAPS screens](https://androidaps.readthedocs.io/en/latest/Getting-Started/Screenshots.html) for explanation of all the visual elemnts on the front screen.

Now is the time to start working your way through the objectives. Please read the documentation specified under each [objective](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html. When you are done, you will have a well tuned loop that you hopefully will understand how to alter when your sensitivity changes.

#### Objectives in AAPS
[The Objectives 1-11](https://androidaps.readthedocs.io/en/latest/Usage/Objectives.html) will guide you through the steps that gradualy will give AAPS access to more tools and powerful adjustments the furter you get. 

I will go through the objectives in the expandable section bellow and objecitve 3 (is like a small exam), and is where most of us get stuck, so I will try and clearify some of the questions here (without answering it for you!)

<details>
  <summary><b>Click here to expand the Objecives section...</b></summary>>

- **Objective 1**
    - Completed profile switch
    - Received BG from selected source
    - NS-connection
    - etc...
    - <img src="./images/obj_1.png" width=250>
- **Objective 2**
    - Try out some of the functionality by:
        - Setting a temp profile
        - Disconnect the pump and reconnect after 1h (this can seem odd for omnipod users, but you're not supposed to remove the active pod, just disconnect the AAPS-loop temporarly in AAPS!)
        - Set a TT (Temporary BG-target)
        - Change some visual settings in AAPS
- **Objective 3**
    - Proove your knowlage - This is where most of us get stuck since some of the questions are a bit hard to interpret. I will try and explain some of them that I find a bit odd: 
    - **Insulin sensitivity factor (ISF)**
        - *ISF should be entered in your AndroidAPS Preferences.*... 
            - hint: a profile setting does not count as "AndroidAPS Preferences"
    - **Profile Switching**
        - *ISF will be 10% higher.*
            - I think this is a bit wrong... ISF/0.9 does not increase ISF by 10 %, but at least it wont decrese - so answer the question with a rounding, and you're good!
    - **Noisy CGM Readings"**
        - *Disable the closed loop to avoid possible over or underdosing.*
            - Yes, on paper! But how many of you actualy does it...? think REALLY noisy values that will make AAPS act really strange!
        - *Verify that your CGM app provides smoothed data.*
            - This question is not ment for BYODA-users... If you'd be using xDrip to get values from your CGM, there is an option to use the Dexcom Native smoothing algoritm or not - with that knowladge it might be clearer...
    - **Sensitivity plugins**
        - *Sensitivity plugins provide the user with suggested changes to basal rates, I:C ratios and ISF that can be used to edit profile..*
            - read: does autosens give direct pruposals for new values for basal/ISF/IC?
        - *Some of the plugin options have configurable time ranges that can be set by the user.*
            - Not sure what they're after, byt maybe a left over from AAPS <2.8? ("In versions prior to AAPS 2.7 user had to choose between 8 or 24 hours manually.")
- **Objective 4**
    - Open loop
        - You'll get preposals (notifications) of AAPS that you manually have to act on. You have to act on **at least 20** tratments and stay on this objective for **at least 7 days** to pass.
        - Modify your BG-target to a larger range to not get as many preposals of treatments. 
    - Inspect all of your settings to see that it looks good!
- **Objective 5**
    - Understand your open loop.
        - You are deviced to have a look in the data/statistics pages in AAPS.
        - Have a look in the Nightscout/report website as well is my sugestion.
    - **Objective has no rquirement** to pass - It's just a good place to really be certain of your settings before giving more power to AAPS. And to stop here if you are on virtual pump!
- **Objective 6**
    - Open loop with "Low Glucose Suspend" for **at least 5 days**
      - Kind of a half-loop
      - AAPS is allowed to stop/reduce the basal when it predicts you going low, but it is not allowed to increase the basal if you have IOB>0.
    - You might get high after low since AAPS have had you off basal (zero-temo) a good while before you actually got low.
    - If you're new to looping, you will find that you'll need much less sugar to stop you from becoming low since AAPS has stopped the basal before you actually got low. Have a look on the IOB when you are low and notice how much carbs you need to correct!
    - Hint: In Graph 1: Activate IOB and COB  so that you see these graphs in the small graph bellow the main one (click small arrow and select graph 1, then repet and select IOB and COB)
- **Objective 7**
    - Now comes the fun part!
    - Increase (in steps) your MaxIOB from 0 to the calculated according to the text in the objective.
    - Hint: It's now fine to set a BG-target instead of a target-range - to get AAPS to constantly adjust basal every 5 min.
      - Do this by specifying your target range like: 5.5-5.5...
    - When you are satisfied with the looping and have done this for **at least 1 day** this object is done!
- **Objective 8**
    - Activate Autosens, which automatically will try to see if your insulin sensitivity is changing and try and correct that by adjusting basal and ISF with a percentage. 
      - Note that IC is not changed with autosens! I have asked why, but haven't got an satisfying answer....
    - Run with Autosens for **at least 7 days**
- **Objective 9 Removed in 3.0!!!**
    - ~~(AMA - advanced meal assist)~~
- **Objective 10**
    - Activate oref1 (SMB) that will give you small micro boluses instead (and together with) basal changes, to give more power to the algooritm.
    - Run for **at least 28 days**
- **Objective 11**
    - Automations
    - Now you have unlocked everything!

</details>




## Watch
A smart watch can besides visualizing current status on your values, also control AAPS: 
- TT (Temporary BG-targets)
- Bolus-calcylator
- eCarbs
- the usual bolus+carbs
- profile swithces

<img src="./images/watch2.png" height=350>

The warable version of AAPS now has to be built the same way you built the AAPS ("app-full-release.apk"), but in the build step you choose a different "Module". Read more [here]](https://androidaps.readthedocs.io/en/latest/Hardware/Smartwatch.html) about aaps and smartwatches.
You'll need to use the "Wear installer", se [youtube-tutorial](https://www.youtube.com/watch?v=8HsfWPTFGQI) to sideload the app.

<img src="./images/watch.png" height=350>

## NSClient-app (for parents/followers)
NSClient is used to monitor and controll AAPS from a follower phone. It is a stripped version of AAPS, so it looks very much the same as AAPS but lacks some of the functionality. If you on the AAPS-phone approve remote controll by NSClient, the NSClient can do profile-switches and enable TT (temporary targets).

If you want more controll (bolus/carbs etc) you have to enable and use the [SMS-commands](https://androidaps.readthedocs.io/en/latest/Children/SMS-Commands.html?highlight=sms). You'll have to add the telephone number in AAPS, and use a 2-factor authentication app. 

Note! NSclient-app only works on Android-phones, but you as a follower can choose to run iOS and controll AAPS with SMS the same way if you'd like. Then you'd want a different nightscout-client like [xDrip4iOS](https://xdrip4ios.readthedocs.io/en/latest/connect/follower/) and use SMS + 2 factor auth (I've seen people use Authy eller 2FAS Auth)

Read more about [remote monitoring with AAPS](https://androidaps.readthedocs.io/en/latest/Children/Children.html)

<img src="./images/remote.png">

NSClient app is possible to build in AndroidStudio just like AAPS, but you usually don't need to since you can just download it from [github](https://github.com/nightscout/AndroidAPS/releases/). You see 2 versions, since you might want to follow 2 different AAPS (parent with 2 t1d kids). Copy to Drive and install just like the rest of the apps.

<img src="./images/nsclientapk.png">
 
## xDrip+
Install xDrip+ from the apk-filen you've downloaded earlier. 
This is directly translated from swedish, so the actual names might differ in the xdrip app...

Open settings (three lines, top left/settings) and choose "hardware-source" (BG-source).
- Telephone with BYODA and AAPS: select `640G / EverSense`
- followers: select `Nightscout Follower` and supply your NS-site address (inkl `https://`) under "Nightscout Follow URL" that will show up under "hardware source" in the settings menu. 
- Optionally activate "Download Treatments" to see insulin and carbs in xDrip, but I have not since we only use xDrip for alarms.

<img src="./images/xdrip_BYODA.png" width=250><img src="./images/xdrip_NS.png" width=250>

Open settings/`Alarms and warnings`  and start from the top

<img src="./images/xdrip_inst.png" width=250> 
<details>
  <summary><b>some alarms I use...</b></summary>>

- "List of glucose level warnings"
    - High/low BG-alarms
    - Very flexible with different alarms on different days/time of day
    - If your phone uses "," instead of "." I had a problem typing the alarm levels... workaround is copy paste into the textboxes
    - <img src="./images/xdrip_BG.png" width=250> <img src="./images/xdrip_BG_low.png" width=250>
- "Glukos warnings settingsr"
    - "Intelligent snooze" & "Intelligant alarm" - Nice to not get an alarm if BG is heading in the right direction!
    - <img src="./images/xdrip_glukvarninst.png" width=250> 
- "Warning for missed readings" 
    - If you loose contact with CGM!
- "Other Warnings" 
    - "Bg falling fast" 
    - "Bg rising fast"
    - <img src="./images/xdrip_BG_fallfast.png" width=250> 
- "additional warnings (xDrip+)"
    - "Alarm on permanent high " - I as a follower use this to get notified only on permanent highs
    - <img src="./images/xdrip_larmBest.png" width=250> 

</details>

Im sure there are lots of other nice alarms you can use in xdrip.  

## OmnipodStash
I have created a little application that you can install next to NS that keeps track of you stash of podds/sensors and insulin. When the stash runs low, you'll get an notification email telling you to order more. 

 Podd- and sensor-changes are registered by AAPS in NS, so the app will automatically reduce the number - insulin cartages you'll have to register manually when one runs out.

Installation instructions [here](https://github.com/maja-lofgren/omnipod_stash) 

<img src="./images/omnipodstash.png"> 
since the writeup of this manual, i have made a fork of NS that includes omnipodstash and a third application called kitescout. See instructions [here](https://github.com/klalle/cgm-remote-monitor/tree/wip/kitescout#customization-of-nightscout)


## Gadgets
### M5Stack Nightscout
We have an [M5Stack Nightscout](https://www.facebook.com/groups/606295776549008) which is a small little gadget with a screen that displays data from NS and has alarm functionality (use next to bed or on your work-desk)

<img src="./images/m5stack.png" height=200><img src="./images/m5stack_2.png" height=200>

