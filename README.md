# InternetArchiveDS
this is internetarchiveDS! it is a remake of a 10 year old nintendo DS homebrew project, (youtubeDS, by GERICOM) that streamed videos from youtube. since 10 years have passed, youtube no longer provides an api for streaming videos, hence breaking youtubeDS. Gericom, unfortunatly gave up on youtubeds! and it has been sitting on the shelf ever since, collecting dust... until now! after aproxximatly 3 months of work, i present to you... INTERNETARCHIVEDS! 

# what is it 🧐
as previously explained, youtubeds is dead. so, i long story short converted the program to run from the youtube's dead api, to the internet archive's video api. apart from that its about the same.

# how i did it 
3 months ago, i picked up youtubeds, and realised somthing: i havent touched C or CPP... ever. ive never seen these "endif's" before. so, it was a bit of a rough start. i used AI for a lot of debugging, and here is where i ran into my first hurdle: DEVKITPRO. now, at that time i knew the project was 10 years old, but i didnt know that devkitpro changed that much over the years... so i ran make, and make clean, a few thousnd times, and HALLELUJAH! IT LINKED!! or so i thought. whenever i opened the application, i would instantly be met with my new best buddy, the guru meditation error. a few days more of this later, i finally found out about CALICO. heres the deal: devkitpro had 2 phases. <2.0 dswifi, and >2.0 dswifi. the 2.0 dswifi update, fundamentally changed the way dswifi worked- adding new connection support,wpa2 so on and so forth. it turns out, my version of devkitpro was the latest version! and it included calico! so i replaced the dswifi library, and kaboom! nothing happened. i started didding in the internet as to why nothing might be working, and soon figured out the only way to make this work would be to have a 10 year old devkitpro compiler, like gericoms: approximately R45. so, i had to painfully, find and MANUALLY REPLACE all of the libraries, and devkitarm core with r45-47 versions. once that was done, i had my first victory!! it opened on the dsi! now, we were in buisness. for 2 months i went backwards and forwards, debugging and building,  all for one common result: guru meditation error. after somthing like 78 generations, i figured out that the reason i was getting a guru meditation error, was because of ONE FLIPPING LETTER! instead of ITCM, i wrote ITC. and that meant the entire video parser didnt load 💀  eventually, i got search to work, wifi to work, then the FINAL hurdle: VIDEO PLAYBACK. now,gericom's old decoder, was EXTREMELY PICKY. it had to be EXACTLY the resolution, EXACTLY 29.9 fps, NOT 30, NOT 28, 29.9. once i got the decoder to comply, i had nother 3 weeks of debugging with audio and macroblock offset. if the audio was off by 0.1 frames, the whole thing would explode. if the macroblocks were not aligned by 0.1 frames, it would explode. i eventually replaced a small portion of the decoder to make it less picky, and finally got everything to work. 3 months later. 

# fatal flaws
unfortunatly, even after ALL OF THAT internetarchiveDS still isnt perfect.
# FATAL FLAW 1: 
EVERY APPROXIMATELY 5.6 SECONDS, THE VIDEO LAGS. i'm not sure why, ive tried to find out, and failed. because of said lag, and the audio NOT having a crashhandler, the video loops back about 2 seconds, and while its doing that, the video WAITS for the audio to catch up, causing extremely annoying delays in the video playback.
# FATAL FLAW 2
for some undiagnosable reason, many video have a major motion blur issue. lets say someon's head moves in the video, their head will be in 2 places for about 2 seconds, when the motion blur resets.
# FATAL FLAW 3
the search function HAS NO CRASHHANDLER meaning if there is no result for whatever you searched, the application will crash.

# how to download
unfortunatly, guys, internetarchiveDS is not a plug and play app. to use it, you need to compile it yourself, with a devkitpro environment of R45. reason being, you need to hardcode your ipv4 ip address, into a file, (more on that later) and because of php, you need to hardcode your FFMPEG path. IF YOU DO NOT HAVE A R45 DEVKITPRO ENVIRONMENT: do not worry: you can download one, pre made, here: just extract it into your c drive ROOT.
# step one
download the uploaded "youtubeds-master.zip" file, and extract it into your download root. 
# step 2
download the r45 devkitpro environment from here: EXTRACT IT ONTO YOUR C: DRIVE ROOT
# step 3 (annoying)
here, you can do one of 2 things: turn on your phone's hotspot and set it to OPEN/WEP if your phone's hotspot supports it- you may need to turn on the older wep/wpa protocol in the advanced menu. unfortunatly, if you connect your dsi to regular protocols e.g. wpa2/3 the old DSWIFI won't be able to support the connection, and wont connect. OPTION 2: set your router/modem to that old open wep/wpa protocol. from here, whichever path you chose, put the hotspot's name into the ds/dsi's original settings menu, and delete any other connections and their saved settings.
# step 4
once you have connected your laptop to your newly estabilished old protocol connection, you must go to firewall settings (if you have a firewall) and turn off the firewall FOR YOUR PRIVATE NETWORK you dont have to turn anything else off, stay safe twin ❤️‍🩹
# step 5
run IPCONFIG in a powershell teminal, bash terminal, or if your a linux user, whatever command prompt you have 😭 find the IPV4 ADRESS AND SAVE IT FOR LATER
# step 6
in youtubedsmaster, your extracted folder, find the folder labeled "source" and inside, look for a file named "youtube_apikey.cpp" open it with vs, vs code, notepad, nano on bash, anything really. in that file you will find this line: const char* youtube_server_ip = "";  PUT YOUR SAVED IPV4 ADRESS IN THE "" AND SAVE THE FILE.
# step 7
INSTALL PHP if you allready have it, great. if you dont, get it.
# step 8
in /source, (the folder inside youtubedsmaster) you will find a file labeled "ythttps.php" open that file, go to LINE 257: "$ffmpegPath =" HERE, you must put your exact ffmpeg path. ive put my ffmpeg path, if yours is the same just change the user. if you somehow dont even have ffmpeg, the get it before doing this. SAVE THE FILE
# step 9
now, you are ready to make the file! open the program, "MSYS2" you should have downloaded the devkitpro environment earlier, so you should have MSYS2. if youre on windows just press the windows key and open it from there. in msys2, type the path to your youtubedsmaster folder, using the CD command- e.g. cd ~/Downloads/YoutubeDS-master
once you are in the path, run this command: make clean && make   once you have done this, your nds file should be in your youtubedsmaster  ROOT, along with the elf, and build folder. copy the newly gnerated internetarchiveds onto whatever you are using to get homebrew on your ds- be it an sd card, an r4 card, etc.
# step 10
you are nearly there now! your final step, is type in your MSYS2 terminal, the command to start ths ythttps.php server! for this, you need your php path. the command is somthing like this: cd ~/Downloads/YoutubeDS-master/YoutubeDS-master && ~/Downloads/php-8.5.9-Win32-vs17-x64/php.exe -S 0.0.0.0:80 -t . MAKE SURE WHEN YOU START THE SERVER THE SERVER IS RUNNING ON PORT 80 IT WILL NOT WORK IF THE SERVER IS NOT ON PORT 80
# step 11
there is no step 11! just keep your server running, open the app on your dsi, search somthing, and enjoy!

# how to use
click the search button, and search somthing! the wifi, takes about  seconds to fully initialise, so dont search for 7 seconds after opening the application. it takes about 20 seconds for a search to come back! this is because it is filtering for videos under 20 megabytes- otherwise loading any video would take 6 years. to initialise a video, once you have your searches, click on a video title and wait for about a minute. if you dont think anything is happening, look at the php server . is it running? if it is, does it say "accepted" anywhere? if it does, just wait. if it doesnt, go through steps 1-10 again, you may have missed somthing.

# credits
TYSM TO GERICOM!! he made the original project 10 years ago and none of this would be possible without him! if youd like to check out his project, go here: https://github.com/Gericom/YoutubeDS 

# STARDANCE
if you are coming here from stardance, thanks for taking the time to read to here! i hope this peoject impresses you, and that you give me a good reveiw!

# thank you!
thank you for reading my readME, i hope you understood everything, apologies for the hassle in setting it all up! this readme took me 2 hours and 13 minutes to right 😭


