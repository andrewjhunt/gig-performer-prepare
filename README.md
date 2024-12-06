# Gig Performer Preparation Script

NOTE: **macOS only**

Configurable AppleScript script that checks your system is ready, then starts up everything you need for your gig, and finally starts Gig Performer.

For example, you can check that you're SSD with sample files is attached, the audio device is active, start up your preferred sheet music reader, and finally, start Gig Performer with your Gig file.

For now, this is recommended for users with some knowledge of MacOS shell scripting. The shell is a powerful but pedantic tool.

Status: Alpha version which needs to be tested on a variety of systems.

## Getting Started

[1] Open Apple's "Script Editor" application

[2] Tn the File menu, open a "New" script

[3] From the [gig-performer-prepare.txt](./gig-performer-prepare.txt) page, copy the script text to your new script file

[4] Configure the script for your environment

[5] Run the scipt

Note: Github users can clone of fork the repo to maintain your version.


## Configuration

The only part of the script you need to change is the "Configuration" section at the top with 7 sections available.

### requiredFiles

If you're instrument files, Gig files or any content is stored on an SSD or may be in the cloud, then you can check that these files are available before starting Gig Performer.

You don't need to check every file, but just a top-level folder. If the folder is missing, then the script will stop and display a dialog box with "Cancel" or "Continue" buttons. If you click "Cancel", then the script will stop and you can find and connect the external storage. Sometimes it is ok to "Continue" with missing content but Gig Performer capability will be reduced.

This example checks for a set of folders on an SSD:

```applescript
set requiredFiles to { ¬
	"/Volumes/MusicSSD/Instruments", ¬
	"/Volumes/MusicSSD/Instruments/Native Instruments/", ¬
	"/Volumes/MusicSSD/Instruments/ROLI/", ¬
	"/Volumes/MusicSSD/Instruments/Spitfire/Spitfire Audio - BBC Symphony Orchestra"}
```

If all your content is on a local drive and your're confident that it will always be available, then specify an empty array:

```applescript
set requiredFiles to {}
```



### audioInterfaces


### usbDevices


### bluetoothDevices

### terminalCommands

### shellCommands

### gigFile


## AppleScript oddities

Comment: text after "--" is a comment
The "¬" character is a line continuation character. It is not a space, so don't remove it.  (It is comparable to the backslash in shell scripts and Python.)

## Script

```applescript
--------------------
-- Script that prepares for and starts up Gig Performer
-- Author: Andrew Hunt - andrew at musios.app
-- License: Creative Commons CC0 1.0 Universal
--------------------

--------------------
-- Configuration
--------------------

-- List of files and/or directories that are necessary for your Gig file to work. Empty array if none.
-- set requiredFiles to {}
set requiredFiles to {¬
	"/Volumes/SilverMusic", ¬
	"/Volumes/SilverMusic/Instruments/Native Instruments/", ¬
	"/Volumes/SilverMusic/Instruments/ROLI/", ¬
	"/Volumes/SilverMusic/Instruments/Spitfire/Spitfire Audio - BBC Symphony Orchestra"}

-- Audio interfaces that must be connected. Empty array if none.
set audioInterfaces to {"XPIANO73", "EVO8"}

-- USB devices that must be connected. Empty array if none.
set usbDevices to {"XPIANO73"}

-- Bluetooth devices that must be connected. Empty array if none.
set bluetoothDevices to {"FS-1-WL"}

-- Commands to run in a terminal. Empty array if none.
set terminalCommands to {¬
	"cd /Users/andrew/Dropbox/code/musios/web-sheet-music-midi; https-server", ¬
	"node ~/localssd/numaxpiano-mapper/index.js"}

-- Shell commands to run. Empty array if none.
set shellCommands to {¬
	"sleep 2; open 'https://127.0.0.1:8080'"}
-- "cd '/Users/andrew/Library/CloudStorage/Dropbox/Music/Cover Songs/_CB_CHARTS'; open -a Preview *.pdf", ¬

-- Optional Gig Performer gig file
set gigFile to "/Users/andrew/Dropbox/Music/Gig Performer/Gig Files/Bands/Cover Brothers/Cover Brothers - 2024-11-26.gig"



---------------------------
-- Do the checks and start the Gig!
---------------------------

log "Checking required_files"
check_required_files(requiredFiles)

log "Checking audio interfaces"
check_system_profiler("Audio interface", "SPAudioDataType", audioInterfaces)

log "Checking USB connections"
check_system_profiler("USB Connection", "SPUSBDataType", usbDevices)

log "Checking Bluetooth interfaces"
check_system_profiler("Bluetooth Connection", "SPBluetoothDataType", bluetoothDevices)

log "Executing commands in terminal"
execute_terminal_commands(terminalCommands)

log "Executing shell commands (no terminal)"
execute_shell_commands(shellCommands)

log "Start Gig Performer"
tell application "System Events"
	if (exists disk gigFile) then
		set gpStartScript to "open -a GigPerformer5 '" & gigFile & "'"
	else
		set gpStartScript to "open -a GigPerformer5"
	end if
	log "Start GP: " & gpStartScript
	set report to do shell script gpStartScript
end tell



---------------------------
-- Supporting functions
---------------------------

on check_required_files(requiredFiles)
	set OK to true
	
	tell application "System Events"
		repeat with idx from 1 to length of requiredFiles
			set filePath to item idx of requiredFiles
			
			if (exists disk item filePath) then
				log "  Required file/folder: " & filePath & " OK"
			else
				log "  Required file/folder: " & filePath & " MISSING"
				display dialog ("Missing file or directory: " & filePath) buttons {"Continue", "Cancel"} default button "Cancel" with title "WARNING"
				set OK to false
			end if
		end repeat
	end tell
	
	return OK
end check_required_files


on check_system_profiler(dataTypeText, dataType, array)
	set OK to true
	
	set shellScript to "system_profiler -detailLevel full " & dataType
	set report to do shell script shellScript
	-- log "Shell: " & report
	
	repeat with idx from 1 to length of array
		set search_text to item idx of array
		
		
		if report contains search_text then
			log "  " & dataTypeText & ": " & search_text & " OK"
		else
			log "  " & dataTypeText & ": " & search_text & " MISSING"
			display dialog ("Missing " & dataTypeText & ": " & search_text) buttons {"Continue", "Cancel"} default button "Cancel" with title "WARNING"
			set OK to false
		end if
	end repeat
	
	return OK
end check_system_profiler


on execute_terminal_commands(commands)
	tell application "iTerm"
		activate
		
		set W to create window with default profile
		
		tell W's current session
			repeat with idx from 2 to length of commands
				split horizontally with default profile
			end repeat
		end tell
		
		set T to W's current tab
		repeat with idx from 1 to length of commands
			set command to item idx of commands
			log "  Shell command: " & command
			write T's session idx text command
		end repeat
	end tell
end execute_terminal_commands


on execute_shell_commands(commands)
	repeat with idx from 1 to length of commands
		set command to item idx of commands
		log "  Shell command: " & command
		set report to do shell script command
	end repeat
end execute_shell_commands
```
