# GP Support Act

<div style="margin: 10px 0px; padding: 4px 8px; font-weight: bold; color: rgb(114, 28, 36); border: solid 1px rgb(245, 198, 203); background-colour: rgb(248, 215, 218);">
Important:

* macOS only because it uses AppleScript
* Early thats needs wider testing
</div>


GP Support Act is a configurable AppleScript script to support your Gig Performer habit. Use it to check that your system is ready for the gig, then start up apps you need for your gig, and finally starts Gig Performer. Things you can do:

* Check whether the computer has internet access and that specific sites are reachable
* Check that an SSD or other external storage is connected
* Check that your audio devices, MIDI devices, and other USB and bluetooth devices are connected
* Open chart files
* Open applications like MuseScore, BOME, etc.
* Open web pages for lyrics, sheet music, etc.
* Open documents like playlists, lyrics, etc.
* Start Gig Performer with your Gig file
* And most other tasks you need that can be run from the command line

## Get the script

[1] Get [gp-support-act.applescript](./gp-support-act.applescript) from the GitHub repo.

[2] Use the "Download Raw file" button to get the file. (If you're a Github user, you can clone the repo then fork your own version.)

[3] Open the file in Apple's "Script Editor" application

[4] Configure the script for your environment

[5] Run the script


### Configure the Script

The script is written in AppleScript and is intended to be easy to modify to your environment and needs.

Change the "Configuration" section at the top. There are examples and comments to help you.

The utility scripts are the second half of the file. You can modify these if you need to, but they are intended to be general-purpose.

If you're comfortable with AppleScript and shell script, then jump right in. The detail below is intended for users who are new to AppleScript and/or the command line.


### Notes on AppleScript syntax

AppleScript is written to be readable and you don't need programming experience to configure and use the script. Two things that might help:

1. Text after "--" is a comment
2. The `'¬'` character is a 'line continuation' character meaning that the next line continues the current line. I am using it for some arrays with many items. (It is comparable to the backslash in shell scripts and Python.) 

### Copying file names to the script

If you need to copy file names to the script, then you can drag and drop the file into the Script Editor window. The full path will be copied to the script.

Alternatively, you can Copy the file in Finder then paste into the Script Editor window. The full path will be copied to the script.  You may need to remove the `\` backslash characters and/or add double quotes around the filename.


## Utility functions

### Check network access - `checkNetAccess(<web address>)`

This function checks that the computer has internet access and that specific sites are reachable. You must provide a web address to check. For general checks a site like `www.google.com` is a good choice. If your performance relies on a specific site, then use that.

```applescript
checkNetAccess("www.google.com")
checkNetAccess("www.musescore.com")
checkNetAccess("127.0.0.1")
```

### Get local copies of cloud files - `cloudDownload(<file-path>)

If you use cloud storage like Dropbox, Google Drive, or iCloud, then you may find important have removed from your computer. We need to force a download files to your local drive so your performance isn't slowed.

Depending upon the size of the file or directory, this may take a while. It's recommended that you use the "Keep Local" option in your cloud storage app to keep the files on your computer.

```applescript
cloudDownload("/Users/musios/Hey Bulldog - The Beatles")
```


### Check external disks are connected - `checkFileOrFolderAccessible(<disk-path>)`

If any plugin data, sheet music or other content is on an external drive, then you can check that the drive is connected before starting Gig Performer.

```applescript
checkFileOrFolderAccessible("/Volumes/MusicSSD/Instruments")
```

Note: `checkFileOrFolderAccessible` does not force a download of cloud files. For that, use `cloudDownload()`.


### Check audio devices are connected - `checkAudioDevice(<device-name>)`

First, to find out the exact names of your audio devices: 

1. Use the `listConnectedAudioDevices()` function in the script
2. Open the System Information app and navigate to the "Audio" section. The names of the devices are what you need to use in the script.
3. Use the `system_profiler SPAudioDataType` command in a terminal

Now you can check that your audio devices are connected:

```applescript
checkAudioDeviceConnected("MacBook Air Speakers")
checkAudioDeviceConnected("MacBook Air Microphone")
checkAudioDeviceConnected("EVO8")
```

Note: this check does not distinguish between input and output devices but this doesn't normally matter.

## Check USB & Bluetooth devices are connected - `checkUSBDevice(<device-name>)` & `checkBluetoothDevice(<device-name>)`

First, to find out the exact names of your USB & Bluetooth devices: 

```applescript
listConnectedUSBDevices()
listConnectedBluetoothDevices()
```

Or 

* Open the System Information app and navigate to the "USB" and "Bluetooth" sections.
* Use the `system_profiler SPUSBDataType` and `system_profiler SPBluetoothDataType` command in a terminal

Now you can check that your devices are connected:

```applescript
checkUSBDeviceConnected("XPIANO73")
checkUSBDeviceConnected("Stream Deck Plus")

checkBluetoothDeviceConnected("FS-1-WL")
```


### Open web pages in your preferred browser - `openWebPage(<browser>, <web-address>)`

The browser can be "Safari", "Google Chrome", "Firefox", etc. Use the full name that appears in your Application list. 

```applescript
openWebPage(browser, "https://musescore.com/official_scores/scores/6937415")
openWebPage(browser, "https://tabs.ultimate-guitar.com/tab/royal-blood/figure-it-out-official-2007289")
```

### Open a document - `openDocument(<document-path>)`

The `openDocument` function opens a document in the default application that Finder would use. For example, a PDF will open in Preview, a text file in TextEdit, a spreadsheet in Excel and so on. 

Files for musical applications also work (if they work in Finder). For example, `.gig` files for Gig Performer, `.mscz` files for MuseScore, `.bmtp` files for BOME MIDI Translator, and many more.

```applescript
openDocument("/Users/musios/charts/All Torn Down.pdf")
openDocument("/Users/musios/charts/Celebration - Kool & the Gang/Celebration.mscz")
openDocument("/Users/musios/charts/Song List.xlsx")
openDocument("/Users/musios/charts/text-doc.txt")
openDocument("/Users/musios/Documents/Bome MIDI Translator/Presets/Korg.bmtp")
```

Note: `openDocument` does not work with web pages. Also, it does not work with regular expressions to match multiple files. If you need to open multiple files, then use a terminal command (below) to open them all at once.


### Run terminal commands - `runTerminalCommand(<command>)` and `itermCommand(<command>)`

The `runTerminalCommand` and `itermCommand` functions each run a command in a terminal window - the first in the MacOS Terminal app and the second in iTerm2 (a popular alternative). 

This is useful for commands that need to be monitored or stopped (like web servers, MIDI utilities) as the terminal window will stay open after the command has run.

```applescript
runTerminalCommand("echo 'howdy'")

itermCommand("open -a 'Bome MIDI Translator Pro' '/Users/musios/Documents/Bome MIDI Translator/Presets/Casio PX-5S.bmtp'")

-- Open all PDF files in a folder
itermCommand("open -a Preview /Users/musios/charts/*.pdf")
```


### Finally, start Gig Performer

Once all the checks are complete, you can start Gig Performer with a specific Gig file. 

```applescript
openDocument("/Users/musios/Documents/Gig Performer/Gig Files/demo.gig")
```
