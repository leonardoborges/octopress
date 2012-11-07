---
layout: post
title: "Build automation with XCode 4.3, KIF and Jenkins"
date: 2012-05-03 13:17
comments: true
categories: [ios, iphone, ipad]
---

__UPDATE:__ As pointed to me in [this commment](http://www.leonardoborges.com/writings/2012/05/03/build-automation-with-xcode-4-dot-3-kif-and-jenkins/#comment-703288043), sym links aren't necessary for Waxsim anymore. Check [Jonathan Penn's fork on github](https://github.com/jonathanpenn/WaxSim).


* * *


After coming back from my holidays in China - which were awesome - I had no downtime at ThoughtWorks and started at a brand new client/project - a much needed change from what I had been working on lately.

It's an iOS project, more specifically an iPad app and that's pretty much all I can tell you about it. I will however tell you how my first few days in the project have been so far.

Being a brand new project I wanted to get our [CI][1] environment up and running as fast as possible - after all the project has only a couple of tests now. For context, here's our setup:

- XCode 4.3
- [GHUnit][2] (I'm not gonna talk too much about it since this was the easy part - their docs are pretty decent)
- [KIF][3] (for UI tests)
- [Jenkins][4]
- [WaxSim][5] (Hack to get the iPhone Simulator to run on the command line)

In most environments I had worked so far - Java, Ruby, Clojure etc.. - running acceptance tests from the command line is a given and so is integrating that into your build system.

Well, that's just not the case with iOS projects. These tasks can be a real pain and take a while to complete. Here's what I had to do: 

### On your development Mac ###

I'm assuming here you have an XCode project with at least one KIF test and that it builds successfully from the GUI, when you hit play. Now we can move on to run the tests from the command line.

This is the script I'm using to run my KIF tests. I call it `RunKIF.sh` and  will walk you through it:

```bash
#!/bin/bash

killall "iPhone Simulator"
rm /tmp/KIF-*

echo "Build the "UI Tests" target to run in the simulator"
xcodebuild -target "UI Tests" -configuration Release -sdk iphonesimulator clean build

OUT_FILE=/tmp/KIF-$$.out
# Run the app we just built in the simulator and send its output to a file
# /path/to/MyApp.app should be the relative or absolute path to the application bundle that was built in the previous step
echo "Running KIF tests"
/tmp/waxsim -f "ipad" "build/Release-iphonesimulator/MyApp.app" > $OUT_FILE 2>&1

# WaxSim hides the return value from the app, so to determine success we search for a "no failures" line
grep -q "TESTING FINISHED: 0 failures" $OUT_FILE

success=`exec grep -c "TESTING FINISHED: 0 failures" $OUT_FILE`


# if there was a failure, show what waxsim was hiding and crucially return with a non-zero exit code
if [ "$success" = '0' ]
then 
    cat $OUT_FILE
    echo "==========================================="
    echo "GUI Tests failed"
    echo "==========================================="
    exit 1
else
    echo "==========================================="
    echo "GUI Tests passed"
    echo "==========================================="
fi
```

You'll notice there are mainly two things happening in this script: 

* It runs xcodebuild to build the target against which your tests will run 
* uses WaxSim to finally run your tests

To get the building step to work, despite what KIF's documentation DOESN'T tell you, you need to add KIF as a target dependency to your KIF Tests target. [This SO thread][6] can walk you through it.

Now on to the fun stuff! 
For some reason I didn't have the time to dig into, WaxSim seems to expect XCode to be under `/Developer/` on your Mac. 

The problem is that since Apple started distributing Xcode through the AppStore, it now lives under `/Applications/Xcode.app/Contents/Developer/`. Bam! This can cause all sorts of issues when building both WaxSim and your project. So if you see weird error messages complaining about not being able to find `DevToolsFoundation.framework` and the like, these sym links will probably fix things for you:

```bash
sudo ln -s /Applications/Xcode.app/Contents/Developer/ /Developer
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsCore.framework /Developer/Library/PrivateFrameworks/
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsCParsing.framework /Developer/Library/PrivateFrameworks/
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsFoundation.framework /Developer/Library/PrivateFrameworks/
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsInterface.framework /Developer/Library/PrivateFrameworks/
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsKit.framework /Developer/Library/PrivateFrameworks/
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsRemoteClient.framework /Developer/Library/PrivateFrameworks/
sudo ln -s /Applications/Xcode.app/Contents/OtherFrameworks/DevToolsSupport.framework /Developer/Library/PrivateFrameworks/
```

I know, right? Moving on.

By now you should be able to successfully run `RunKIF.sh` and get meaningful results from your test. Let's configure our CI server now, shall we?

### On your CI Mac ###

Depending on which state your build box is, you might need to create the same sym links I created in the previous section. So ssh into your build box and try to build your project from the command line:

```bash
xcodebuild -target "UI Tests" -configuration Release -sdk iphonesimulator clean build
```        

This time I'll assume you already have Jenkins installed and running. 

Your Jenkins server is probably running as the jenkins user. You need to change that. Jenkins has to be running within a Desktop session - that's a requirement from WaxSim. 
It needs an active desktop session in order to launch the simulator - and thus needs to run as a user who can login to your CI Mac.

If you installed Jenkins using Homebrew, here's a plist you can use to specify which user jenkins should run as:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>Jenkins</string>
    <key>ProgramArguments</key>
    <array>
    <string>/usr/bin/java</string>
    <string>-jar</string>
    <string>/usr/local/Cellar/jenkins/1.439/lib/jenkins.war</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>UserName</key>
    <string>UserWhoCanLogintoThisMac</string>
</dict>
</plist>
```

Then load the new plist:

```bash
launchctl load -w  /Users/UserWhoCanLogintoThisMac/Library/LaunchAgents/org.jenkins-ci.plist
```

Don't forget to enable automatic login on the CI Mac for the user Jenkins is using. Check [this article][7] to learn how to do it.

Now we're mostly done. Just go ahead and configure your project.

Add a build step of the type `Execute Shell` and provide `sh RunKIF.sh` as the command string. Trigger your build and everything should work as expected.


### Final thoughts ###

Once I actually got these steps together to write this post, it didn't look like much. But the amount of time I wasted finding out what had to be done to actually make this work is just insane.

I'm shocked at how much behind iOS development tools seem to be. It just shouldn't take this much effort and time only to get some UI tests automated. So far I'm disappointed with the tooling around the Apple ecosystem and I really hope this improves sooner rather than later.


[1]: http://en.wikipedia.org/wiki/Continuous_integration
[2]: https://github.com/gabriel/gh-unit
[3]: https://github.com/square/KIF
[4]: http://jenkins-ci.org/
[5]: https://github.com/square/waxsim
[6]: http://stackoverflow.com/questions/10000600/build-error-on-zapp-when-running-iphone-app-with-kif-testsuites
[7]: http://www.tech-faq.com/how-to-re-enable-mac-os-x-automatic-login.html