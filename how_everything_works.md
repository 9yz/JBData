# How does this work?
This repo is the culmination of like 5 different programs duct-taped together to get you your Daily Jailbreak Updates™.

## 1. Data Collection
The data is being automatically collected from the gameME [API](http://tangoworldwide.gameme.com/api/serverinfo/104.153.108.145:27015/players) every 5 minutes using a JavaScript program written FetchBot. It is automatically compiled into `results.csv`. This file is then proccessed by the next program.

You can find the source code for this program [here](https://github.com/TangoWorldWide/JBDataScripts).
## 2. Data Proccessing
Once the data is collected, it gets proccessed by a C++ program I wrote. It takes `results.csv` and crunches the numbers to find the maps that were played the most. To do this, it adds up the number of players online every time a map is recorded in `results.csv`. It then nultiplies this number by 5 (because data is recorded every 5 minutes) to get the total number of player-minutes spent on each map. The output of this program can be found in lastDay.txt (which contains the playtime data from the last day) and lastWeek.txt (the same, but with a week).

You can find the source code for this program [here](https://github.com/TangoWorldWide/JBDataScripts).
## 3. Uploading
Once it's been proccessed, `lastDay.txt`, `lastWeek.txt`, and `results.csv` get uploaded to GitHub.
## 4. Phone Notification
Because I'm usually away from my computer when this happens, I want to make sure everything worked correctly. By using a modified version of [this autoHotKey script](https://www.autohotkey.com/boards/viewtopic.php?t=4842), I get a push notification on my phone when everything is done.

```
PB_Token   := "[redacted]"
PB_Title   := "Upload Complete"
PB_Message := "Nothing broke (yet)!"

WinHTTP := ComObjCreate("WinHTTP.WinHttpRequest.5.1")
WinHTTP.SetProxy(0)
WinHTTP.Open("POST", "https://api.pushbullet.com/v2/pushes", 0)
WinHTTP.SetCredentials(PB_Token, "", 0)
WinHTTP.SetRequestHeader("Content-Type", "application/json")
PB_Body := "{""type"": ""note"", ""title"": """ PB_Title """, ""body"": """ PB_Message """}"
WinHTTP.Send(PB_Body)
Result := WinHTTP.ResponseText
Status := WinHTTP.Status

Exit
```

## The Cron Job
All of this (with the exception of the data collection) only needs to happen once per day. So, I use a [cron job](https://en.wikipedia.org/wiki/Cron) running in my Ubuntu VM to automatically execute these tasks. Here's the command:

```
@daily /mnt/c/Users/dashz/Documents/Tango/JBData/playerHours results.csv -d > lastDay.txt && /mnt/c/Users/dashz/Documents/Tango/JBData/playerHours results.csv -w > lastWeek.txt && git add /mnt/c/Users/dashz/Documents/Tango/JBData/results.csv && git add /mnt/c/Users/dashz/Documents/Tango/JBData/lastDay.txt && git add /mnt/c/Users/dashz/Documents/Tango/JBData/lastWeek.txt && git commit -m "Daily Update" && git push && /mnt/c/Users/dashz/Documents/Macros/uploadFinishedNotification.exe
```
Here's how it breaks down:
```
@daily <-- do this once per day

/mnt/c/Users/dashz/Documents/Tango/JBData/playerHours results.csv -d > lastDay.txt <-- analyze last day, save it to lastDay.txt

&& <-- only do the next command if the last one finished successfully

/mnt/c/Users/dashz/Documents/Tango/JBData/playerHours results.csv -w > lastWeek.txt && <-- analyze the last week into lastWeek.txt

git add /mnt/c/Users/dashz/Documents/Tango/JBData/results.csv &&  <-- these files should be included in the gitHub upload
git add /mnt/c/Users/dashz/Documents/Tango/JBData/lastDay.txt &&  <-|
git add /mnt/c/Users/dashz/Documents/Tango/JBData/lastWeek.txt && <-|

git commit -m "Daily Update" && <-- Upload the files
git push &&                     <-|

/mnt/c/Users/dashz/Documents/Macros/uploadFinishedNotification.exe <-- Run the notification program (compiled AHK script)
```
