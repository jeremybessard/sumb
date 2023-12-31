# Scheduled Updates Menu Bar, aka SUMB (Jamf only)
![sumbmenu](https://github.com/jeremybessard/sumb/assets/53154185/42365460-3880-4158-a351-a1b790e4ed9e)

I made something I call the [NoMAD menu](https://clementine.la/wp-content/uploads/2023/07/nomad-menu.png) for macOS updates. It provides information about the status of a MDM scheduled update and the remaining number of allowed deferrals before the Mac force-installs a system update.

The recommended method by Apple or Jamf for mass updates with user deferrals is not clear as to why the MDM command often fails. It could be related to network issues, disk space, or even battery status. I’m aware of cool tools like [Nudge](https://github.com/macadmins/nudge) or [SUPER](https://github.com/Macjutsu/super), which I have used in the past with great results! However, I find them a bit challenging to set up correctly, and they don’t work in conjunction with the MDM update command.

To address this, I came up with a solution ‘set once & forget it’ called the Scheduled Updates Menu Bar (SUMB). It’s essentially a plugin for the open-source app [SwiftBar](https://github.com/swiftbar/SwiftBar). There are no notifications to bother the users. Instead, think of it as a constant nudging in the toolbar with an ugly red countdown. Additionally, SUMB leverages the Jamf APIs and the target Macs have to be on macOS 12.3 or above.

SUMB is provided ‘AS IS’ and serves more as a proof of concept than anything else. I don’t plan to maintain it, so feel free to copy, modify, or share it with or without attribution—I don’t mind. My hope is that it will inspire someone to develop [a real application](https://developer.apple.com/documentation/devicemanagement/osupdatestatusresponse/osupdatestatusitem) based on this idea.

### 1. Deploy SwiftBar and SUMB-Plugin

[Download SwiftBar-1.5.0.pk](https://github.com/jeremybessard/sumb/releases/tag/1.0)[g](https://clementine.la/scripts/scheduled-updates-menu-bar-sumb/#) (or make [your own pkg](https://github.com/swiftbar/SwiftBar) and install it in the /Applications folder) | SHA1 : 9209284d165554cf121f4670eaac515482a06bc0

[Download SUMB-Plugin-1.01.pkg](https://github.com/jeremybessard/sumb/releases/tag/1.0) – [Source code](https://github.com/jeremybessard/sumb/blob/main/sumb.10m.sh) | SHA1 : 4088d0814ec49903ef2b5ffe48c260f4138ce635

Sorry, those are unsigned (right click > open to install them manually). You can take a look inside with [Suspicious Package](https://mothersruin.com/software/SuspiciousPackage/).

**In ⚙️ > Packages**, give SwiftBar-1.5.0.pkg higher **Priority** so that it installs before SUMB.

Create a Jamf Policy :
- Trigger : Check-in
- Frequency : Once per computer
- Payload : Packages

![jamf-policy-1024x681](https://github.com/jeremybessard/sumb/assets/53154185/dc04b5f7-6a46-4aab-b96b-059a48e489c5)

### 2. Create a jamf-api account

**In ⚙️ > User accounts and groups** : create an account named « jamf-api » with the bare minimum permissions. We only need to check READ for the Computers row.

![jamf-api](https://github.com/jeremybessard/sumb/assets/53154185/79cc11b6-15c8-4ee9-b4a2-de81ce2c9172)


### 3. Deploy SUMB-API script

[Download SUMB-API.sh](https://github.com/jeremybessard/sumb/blob/main/sumb_api.sh)

**In ⚙️ > Scripts** : Add the script and edit it with your Jamf Pro URL, username (jamf-api) and password.

Create a Jamf Policy
- Trigger : Check-in
- Frequency : Once every day
- Payload : Scripts

### 4. Launch SwiftBar at login (optional)

Run the following command line on computers : `osascript -e 'tell application "System Events" to make login item at end with properties {name: "SwiftBar",path:"/Applications/SwiftBar.app", hidden:false}'`

### How does it work?

For example, let’s consider Mary’s computer running macOS 13.3, and we intend to offer her 5 chances/days/deferrals to update it to macOS 13.5 at her convenience.

To begin, send an MDM update command with 5 deferrals.
**Action > Send Remote Commands > Update OS version > Download and allow macOS to install later : Max User Deferrals : 5**

Wait something like 20 minutes and execute the SUMB-API policy on Mary’s computer (flush it if it has been run before). This script will generate an app.sumb.managedsoftwareupdate.plist file in the /Shared/sumb folder, which will contain the status of the update command. A countdown to 5 should appear in the menu bar. Then every day, the SUMB-API script will update the deferrals remaining number.

During my tests, sending an other MDM update command while the previous one is still in progress confuses the update and deferral processes. However, you can find a Cancel button in the Software Update pane.

<img width="715" alt="cancel-update" src="https://github.com/jeremybessard/sumb/assets/53154185/71e3339e-b84b-4c9c-b805-d5ca4f84e699">

Bellow, this is where the SUMB-API script reads the info from :
<img width="797" alt="Screenshot 2023-07-24 at 22 54 59" src="https://github.com/jeremybessard/sumb/assets/53154185/d035cb0c-e2cc-4216-af26-e530df9dff52">
