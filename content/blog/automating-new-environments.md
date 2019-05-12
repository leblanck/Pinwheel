---
title: "Automating New Web Dev Environments"
date: 2019-05-05T13:52:34-04:00
draft: false
tags: ["automation", "bash", "homebrew"]
description: "Automating Environment Setups"
meta_img: ""
author: "Kyle LeBlanc"
---

"We need to re-image your laptop" "Time to nuke and repave" "Your machine is totally borked"

No one wants to hear those words come from IT. *Especially* developers. I've had to deliver this news more times than I can count from users destroying their own machines either from messing with permissions, malware, or just from updates gone wrong.

![Yes, I did really get this Slack message](/img/undo.png)

I've done my best to help alleviate some of this pain from the dev crowd by assisting in automating their environment setup on a net-new machine.

I run this script as soon as I log into a new machine - curl/clone it from [github](https://github.com/leblanck/bustit), `chmod +x`, then let it run. The script/process I'll share below is for my own personal use (yes, I do re-image my own laptops a lot), however it is *very* similar to what I used in prod. I've just removed some of our corporate specific env variables from this equation for wellbeing of all parties involved. My personal version of this script is called bustIT - like bust (break) IT - idk, just go with it.

**All Hail Homebrew**

This entire process pretty much all revolves around Homebrew. If you don't know about Homebrew please do yourself a favor and [check it out.](https://brew.sh/)

The first thing I do, which I pretty much do in every script over 10 lines is to setup a log file in `/Library/Logs/bustIT.log`. The first function that runs installs Homebrew, taps brew to enable casks, branches to a second function to intall all necessary casks, installs [PWGen](https://blog.tnyc.me/pwgen-on-linux-and-osx), branches to a third function to setup the local macOS environment (very ~aesthetic~).

```
homebrewInstall() {
  echo "`date` Installing Homebrew..."
  /usr/bin/ruby -e "$(curl -fsSL URLHere)"

  echo "`date` Tapping Cask..."
  brew tap homebrew/cask

  echo "`date` Installing Apps..."
  caskInstaller

  echo "`date` Installing PWGen..."
  brew install pwgen

  echo "`date` Setting up macOS local preferences..."
  localMacOSSetup

  echo "`date` Done!"
}
```

My favorite thing about this script is how easy it makes it to add a new cask (.app) to be installed. All I need to to is add the cask name to the `casks` variable that I have set up in the `caskInstaller` function. You can see I have a few listed below like Slack, Spectacle, etc. This function will iterate through that array and install each cask listed. When it gets to a special cask, like Atom for example, it will launch a function called `atomExtras` which will install Atom themes and packages via apm (my second favorite thing about this process)


```
caskInstaller() {
  # This will iterate through cask installs, only stopping when a trigger
  # is hit to execute
  # additional commands for that specific installer.
  # To add additional casks, add them into $casks array.
  declare -a casks=("spectacle" "wireshark" "clipy" "atom" "slack" "zeplin")
  atomTrigger="atom"
  adobeTrigger="adobe-creative-cloud"

  for i in "${casks[@]}"
  do
    if [ "$i" == "$atomTrigger" ]; then
      #Triggered when current cask = Atom
      echo "`date` Installing" $i"..."
      brew cask install $i
      atomExtras
    elif [ "$i" == "$adobeTrigger" ]; then
      #Triggered when current cask = Adobe CC
      echo "`date ` Installing" $i"..."
      brew cask install $1
      sleep 5
      open -a /usr/local/Caskroom/adobe-creative-cloud/latest/CC\ Installer.app
    else
      #All other casks
      echo "`date` Installing" $i"..."
      brew cask install $i
    fi
  done
}
```
Eventually we get to the function to set up the local macOS environment. This includes wallpaper, Finder theme (for Mojave), and dock preferences. In enterprise prod I have this set the default corp wallpaper etc.

```localMacOSSetup() {
  #The following will setup local macOS preferences

  # Set Dark Mode
  tell application "System Events"
      tell appearance preferences
          set dark mode to true
      end tell
  end tell

  #Only show open apps in the dock; remove everything else
  defaults write com.apple.dock static-only -bool true

  #Auto-hide dock
  defaults write com.apple.dock autohide -bool true

  #Download new wallpaper
  curl -L -o ~/Pictures/wallpaper.png URLHere
  #Set new wallpaper
  osascript -e ‘tell application “Finder” to set desktop picture to POSIX file “~/path.png”’

  #restart dock process
  killall Dock
}
```

The function above results in a desktop looking like this:
![~Aesthic macOS Desktop~](/img/desktop.png)
