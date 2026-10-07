# InternetArchiveDS

Stream videos from the Internet Archive on your Nintendo DSi!

InternetArchiveDS is a remake of **YoutubeDS** by **Gericom**, a Nintendo DS homebrew app that streamed YouTube videos about 10 years ago. The old YouTube API that the app relied on is long dead, which broke YoutubeDS, and the project has been sitting on the shelf collecting dust... until now! After roughly 3 months of work, I present... **INTERNETARCHIVEDS!**

## What is it? 🧐

As explained above, YoutubeDS is dead. So, long story short, I rewired it to use the Internet Archive's search and metadata API instead of YouTube's. Apart from that, it works about the same way: search for something on the touch screen, tap a result, and watch it.

## How it works

The DS can't speak HTTPS or talk to modern APIs, so a small PHP server runs on your PC and does the heavy lifting:

```
DSi  <--Wi-Fi-->  PHP proxy on your PC (ythttps.php)  <--->  archive.org
```

1. The DS sends your search to the PHP proxy, which asks Archive.org and sends the results back.
2. When you pick a video, your PC downloads it, uses **ffmpeg** to convert it to **176×144, 29.97 fps MPEG-4 video with mono 22050 Hz AAC audio** (the only format the DS decoder understands), and streams the result to the DS over plain HTTP.

Search results are filtered to videos of **20 MiB or less**, otherwise downloading and converting would take forever.

**What you need:**
- A Nintendo DSi with homebrew set up (tested on a DSi XL running TWiLight Menu++)
- A Windows PC (the steps below are written for Windows)
- A Wi-Fi network the DS can join (Open or WEP, see step 3)
- devkitARM **r45** (GCC 5.3.0), PHP 8.0 or newer, and ffmpeg

## How I did it

Three months ago I picked up YoutubeDS and realised something: I had never touched C or C++... ever. I'd never even seen an `#endif` before. So it was a rough start. I used AI for a lot of the debugging.

**Hurdle 1: devkitPro.** I knew the project was 10 years old, but I didn't know how much devkitPro had changed since then. I ran `make` and `make clean` a few thousand times, and HALLELUJAH, IT LINKED! Or so I thought. Every time I opened the app I was met with my new best buddy, the guru meditation error. A few days of that later I found out about **Calico**: newer devkitPro releases are built on Calico, a rewritten DS system library (including a new dswifi), and the old code just isn't compatible with it. I swapped the dswifi library and, kaboom, nothing happened. Eventually I worked out that the only way to make this work was to use a 10-year-old toolchain like Gericom's: **devkitARM r45**. So I painfully replaced the libraries and the devkitARM core with r45 versions. Once that was done, I had my first victory: **it opened on the DSi!**

**Hurdle 2: one missing letter.** For two months I went backwards and forwards, debugging and building, all for one common result: guru meditation error. After something like 78 generations I finally found out why: ONE FLIPPING LETTER. The decoder's assembly said `.section .itc` instead of `.section .itcm`, so the entire video decoder was never loaded into the right memory and the DS just jumped into garbage 💀

Eventually I got search to work, then Wi-Fi to work, then came the FINAL hurdle: **video playback.**

**Hurdle 3: video.** Gericom's old decoder is EXTREMELY PICKY. It only understands one exact format: 176×144, **exactly 29.97 fps** (not 30, not 28), MPEG-4 Simple Profile. That's why the PC converts every video before it reaches the DS. Even then, the picture turned into garbage after the first frame. It took weeks and a LOT of dead ends (I checked every decoder table against the spec!) before I found the real culprit, and it wasn't the decoder at all. The video and audio are interleaved in the same file, and my code that skips over the audio chunks between video frames was initialised wrongly, so it never skipped a single one. Every frame after the first was reading audio bytes as video. Along the way I also fixed a wrong hardcoded timing value in the video header parsing (it needs 15 bits for 29.97 fps video) and ported the newer IDCT routines from Gericom's later `mpeg4player` branch. After that, it finally played. 3 months later. 😭

## Known issues (a.k.a. the fatal flaws)

Unfortunately, even after ALL of that, InternetArchiveDS still isn't perfect.

1. **The video lags roughly every 5 to 6 seconds.** The DS plays audio from a looping buffer that it can't pause, so when the video stalls, the audio replays a short section while the video waits for it to catch up. This causes extremely annoying delays. I'm not sure why the stalls happen: lowering the video bitrate all the way down to 50 kbps changed nothing, so it doesn't seem to be raw decoding speed. If you figure it out, please open an issue or a pull request!
2. **Motion ghosting on many videos.** For some undiagnosed reason, moving objects leave a trail. If someone's head moves, it will appear in two places for about 2 seconds until the picture refreshes.
3. **Searching for something with no results crashes the app.** There's no error handling for an empty result list yet. Restart the app and try a different search.

Also worth knowing: every video is shrunk to 176×144, because that's the only size the decoder supports.

## Setup guide

InternetArchiveDS is not plug and play. You have to compile it yourself with a devkitARM r45 environment, because you need to put **your PC's IP address** and **your ffmpeg path** into the code. It takes a while, but you only have to do it once. (Apologies for the hassle!)

### Step 1: Get the project

On this page click the green **Code** button, choose **Download ZIP**, and extract it somewhere easy, like your Downloads folder. (Or use `git clone`.) The folder you extracted is your **project folder**.

### Step 2: Get the devkitARM r45 environment

The project only builds with an old devkitPro (devkitARM release 45). Newer versions won't work, see "How I did it". I made a ready-to-use package so you don't have to hunt for old libraries:

**Download: [DOWNLOAD LINK COMING SOON]**

Extract it into the **root of your C: drive**, so that the folder `C:\devkitPro` exists. It includes MSYS2, which you'll use to build in step 9.

### Step 3: Set up Wi-Fi the DS can use

The DS can't use modern Wi-Fi security. Its homebrew Wi-Fi only supports **Open** or **WEP** networks, and it will not connect to WPA2/WPA3. You have two options:

- **Phone hotspot (easiest):** turn on your phone's hotspot and set its security to **Open** (or WEP, if your phone still offers it; some phones hide it in the advanced hotspot settings).
- **Router/modem:** set up a network on your router that uses Open or WEP.

Then on the DS/DSi go to **System Settings → Internet → Connection Settings**, set up a connection for that network, and delete any other saved connections.

⚠️ An Open network has no password, so anyone nearby could join it. Only use it while you're playing, then switch it off.

### Step 4: Connect your PC and allow the server through the firewall

Connect your PC to the **same network** as the DS. When you first start the PHP server in step 10, Windows will probably ask whether to allow `php.exe` through the firewall: click **Allow** for **Private networks**. If the DS can't reach your PC later, the firewall is the first thing to check.

### Step 5: Find your IP address

Open PowerShell, Command Prompt or an MSYS2 terminal and run:

```
ipconfig
```

Find the adapter that's connected to your hotspot/router and write down its **IPv4 Address** (something like `192.168.x.x` or `10.x.x.x`).

### Step 6: Put your IP address in the code

The project keeps your private settings in a file that isn't uploaded to GitHub. In your project folder:

1. Go into the `source` folder.
2. **Copy** `youtube_apikey.cpp.template` and name the copy `youtube_apikey.cpp`.
3. Open `youtube_apikey.cpp` in any text editor and make sure it contains these lines (with no `//` at the start):

```cpp
const char* youtube_apikey = "";
const char* youtube_server_ip = "PUT-YOUR-IPV4-ADDRESS-HERE";
```

Replace `PUT-YOUR-IPV4-ADDRESS-HERE` with the address you saved in step 5, keeping the quotes. The file is still called "youtube_apikey" because that's what the original project called it. Leave the API key empty, it isn't used.

⚠️ **Your PC's IP address can change** (especially on a phone hotspot). If the app suddenly stops working one day, run `ipconfig` again, and if the number is different, update this file and redo step 9. Setting a fixed/reserved IP for your PC in your router avoids this.

### Step 7: Install PHP

Install **PHP 8.0 or newer** (the proxy uses PHP 8 functions). On Windows, download the x64 **Non Thread Safe** zip from https://windows.php.net/download and extract it somewhere, for example `C:\php`.

Then, in the PHP folder, copy `php.ini-development` to `php.ini`, open it, and make sure these lines are enabled (no `;` at the start):

```ini
extension_dir = "ext"
extension=openssl
allow_url_fopen=On
```

Without `openssl`, the proxy can't talk to Archive.org (you'll see "Unable to find the wrapper https" in the server window).

### Step 8: Install ffmpeg and set its path

Install ffmpeg if you don't have it, for example from https://www.gyan.dev/ffmpeg/builds/ or with `winget install Gyan.FFmpeg.Essentials`.

Then open `ythttps.php` (it's in the **project folder**, not in `source`), search for `$ffmpegPath` and put the full path to your `ffmpeg.exe` there, using forward slashes:

```php
$ffmpegPath = 'C:/path/to/ffmpeg/bin/ffmpeg.exe';
```

Save the file.

### Step 9: Build it

Open **MSYS2** from the Start menu (it came with the devkitPro environment). Go to your project folder with `cd` and build:

```
cd ~/Downloads/<your project folder>
make clean && make
```

Adjust the `cd` path to wherever you extracted the project. When it finishes, `InternetArchiveDS.nds` will be in your project folder (next to the `.elf` and the `build` folder). Copy the `.nds` to whatever you use to run homebrew on your DS (SD card, flashcard, etc.).

Whenever you change a setting or a source file, rebuild **and copy the new `.nds` to the SD card again**, otherwise the DS keeps running the old version.

### Step 10: Start the server

In MSYS2, go to your project folder and start PHP's built-in server **from that folder**, on **port 80**:

```
cd ~/Downloads/<your project folder> && /path/to/php.exe -S 0.0.0.0:80 -t .
```

Replace `/path/to/php.exe` with where you put PHP, for example `/c/php/php.exe`. It **must** be started from the project folder (otherwise it serves the wrong directory) and it **must** be on port 80, or the DS won't find it. If port 80 is already in use, close whatever program is using it.

### Step 11: There is no step 11!

Keep the server running, open InternetArchiveDS on your DSi, search for something, and enjoy!

## How to use

1. Wait about **7 seconds** after opening the app before you search, because the Wi-Fi needs time to connect.
2. Tap the search button, type something, and search. Results take about **20 seconds** to come back, because the server is filtering for videos of 20 MiB or less. (Otherwise loading any video would take ages.)
3. Tap a video title and wait. Starting a video can take up to about a minute, because your PC has to download and convert it first.
4. The PHP server handles one request at a time, so wait for a video to start before searching again.

If you don't think anything is happening, look at the PHP server window. Is it running? Does it say **"Accepted"** anywhere? If it does, just wait. If it doesn't, go through steps 1 to 10 again, you may have missed something.

### Troubleshooting

- **Search never returns / DS can't connect:** check the server is running, your IP in step 6 is current, the firewall allows `php.exe`, and the network is Open/WEP.
- **Search works but videos never start:** check the `$ffmpegPath` in step 8.
- **"Unable to find the wrapper https" in the server window:** enable `extension=openssl` in `php.ini` (step 7).
- **You changed something and nothing changed on the DS:** you forgot to rebuild, or to copy the new `.nds` to the SD card.

## Credits

**THANK YOU SO MUCH to Gericom!** He made the original project 10 years ago and none of this would be possible without him. Go check out the original: https://github.com/Gericom/YoutubeDS

Also thanks to:
- The **FFmpeg** project, whose IDCT routines (via Gericom's `mpeg4player` branch) are used for decoding
- The **devkitPro** team and the **Internet Archive**

InternetArchiveDS is a fan project and is not affiliated with or endorsed by the Internet Archive, Nintendo, or Gericom.

## Thank you!

Thank you for reading my README, I hope you understood everything. Apologies again for the hassle in setting it all up! This README took me 2 hours and 13 minutes to write 😭
