---
layout: post
---

Hi, this is my first (real) Post on my Blog! Thanks that you are taking time out of your day to read my stuff, appreciate it!
with that out of the way:

# How to create your own "Open Here" tool for MacOS

### backstory time
I've owned a Mac since the big rise of Apple silicon circa 2021 and in the beginning it was really hard to switch to MacOS and getting accustomed to all the little quirks of it. By a wide margin my favorite util from linux and also windows in this case was missing: "Open Here" in muh context menu is not there... no open terminal here, no open vscode in this folder. Luckily i ain't the first person that has this problem and the most recommended solution is obviously [FinderGo](https://github.com/onmyway133/FinderGo) which i used a lot at first but i quickly realized that its too limited for my purposes (sorry cute lil turtle 🐢)

the next thing that got recommended a lot was [OpenInTerminal and OpenInTerminal-Lite](https://github.com/Ji4n1ng/OpenInTerminal) which i used for around 2 months (primarily the lite version). it was pretty great actually and does most of the things i've wanted but unfortunately by then i've got a severe case of windows'ism and missed my old friend notepad++ of which there existed an awesome multiplatform port: [NotepadNext](https://github.com/dail8859/NotepadNext).

at first this does not seem like that big of a problem until you realize OpenInTerminal does kinda but not really support arbitrary Apps which is bad because the whole point of Notepad++ was _right click a file and edit_... so if that aint working lets do this ourselves!

### the interesting part
_Question: Whats so special about mac apps?_ <br>
well... nothing really. in reality they are just a folder ending with ".app" containing a specific file structure<br><br>
_Another Question: Are you going to use XCode?_ <br>
in short no. i dont like it, like _really_.<br>

since apps are only some kind of fancy file structure we can simply create our own app packages without touching xcode once. sadly we need to install it anyways because we need them devtools<br>
we're just going to use swiftc aka the swift compiler and good ol' shell scripting. why swift? well we dont have much choice because we need to interact with Apples Scripting Bridge and AppleScript is **_just dont do it trust me_**

#### quick summary what the hell is ScriptingBridge:
a cursed way of communicating and interacting with macos native apps through Objective-C interfaces. but you really dont need to know that. what we need to know is how "Finder" exposes its scripting bridge api which we can luckily steal from OpenInTerminal or generate ourselves with [this](https://github.com/tingraldi/SwiftScripting).


**now** with that in mind its quite easy:
- get the finder application (like windows multiple finder windows are still the same process)
- get a list of the currently selected files/folders (they are automagically sorted... i think...)
- if this list is empty we get the current location of the frontmost finder window
- open our process with the current selected file or current window

in short this will look something like this
```swift
var finder = SBApplication(bundleIdentifier: "com.apple.Finder")! as FinderApplication

var selected = finder.selection?.get() as! Array<FinderItem>
var focusedWindow = finder.FinderWindows?().firstObject as! FinderFinderWindow?

var url: URL?;

if selected.count > 0 {
	url = URL(string: selected[0].URL!);
} else if focusedWindow != nil {
	url = URL(string: (focusedWindow!.target! as FinderItem).URL!)
}

let whatToOpen = Bundle.main.infoDictionary?["OpenBundleID"] as! String;

if url != nil {

    let openProc = Process()

    openProc.executableURL = URL(string: "file:///usr/bin/open")
    openProc.arguments = ["-b", whatToOpen, url?.path as! String];
    try openProc.run()
}
```

for real, this is basically the full program code.
run that through swiftc, place our binary in apples magical .app folder structure, place some fancy plist file for "configuration" and steal the icon of our target app that we want to open.

easy! we now made our own app without touching xcode. to be honest i didnt test if this app bundle works on other macs because i believe apple checks for code signing or something similar but should be no biggie to build it yourself. now we can basically do whatever we want with this base!

i personally dragged my favorites like vscode, iterm and notepad into my finder bar 
![](https://github.com/Complexicon/OpenHere/blob/main/pics/example2.png?raw=true)

you can checkout my github repository [here](https://github.com/Complexicon/OpenHere/)

thanks again for reading my first blog post and have a nice day!
