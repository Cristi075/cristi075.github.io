---
layout: post
title:  "De1CTF 2019 Mine Sweeping Writeup"
date:   2019-08-06 20:00:00 +0300
categories: CTF_writeup De1CTF_2019
---
This is my writeup for the `Mine Sweeping` challenge. This challenge was part of [De1CTF 2019](https://ctftime.org/event/843).  
The files for this challenge might be still available on [ctftime](https://ctftime.org/task/8921) so you can also give it a try if you want.  

For this challenge we had access to an archive containing a game made in Unity and its dependencies (DLLs and assets). At a first glance, the game looked like the classic minesweeper (That was probably expected with that title). Also, I couldn't find any stated goal or objective that would lead me to the flag so I assumed that I probably have to finish/win the game in order to get the flag.

![Minesweeper game]({{site.baseurl}}/assets/img/De1CTF_2019/minesweeping_1.png){: .center-image}

Because the game was created using Unity my first thought was to decompile it using [dnSpy](https://github.com/0xd4d/dnSpy). As reverse isn't my thing (yet...) I couldn't get any code by doing this.  
Also, some quick static analysis performed on the game files did not reveal anything useful.  

My next approach was trying to see if I find any useful information by playing the game for a bit. It is the minesweeper game that I'm already familiar with but it looks like there are quite a lot of mines. Sadly, I was never good at minesweeper.  
After a few attempts at playing the game I noticed that the layout of the game board is always the same. This meant that I could memorize where the mines are by playing multiple times and than do a perfect run using the memorized location.  
Or I could write a script for that ...  

The only tool that I knew about with for writing a script that interacts with desktop apps was [AutoIt](https://www.autoitscript.com/site/).  
I used the [helper tool](https://www.autoitscript.com/autoit3/docs/intro/au3spy.htm) that comes with AutoIt to get the coordinates for the top-left cell from the game board and also to estimate what is the distance between two cells.
The script that I created can be seen below.  

<details>
  <summary>autosweep_explore.au3 (click to expand)</summary>
<p>
{% highlight autoit %}
#include <AutoItConstants.au3>
#include <FileConstants.au3>

# Coordinates for the top-left corner
$start_x = 880
$start_y = 110

$offset_x = 25
$offset_y = 25

# One of the pixels that turns yellow if we click on a mine
$check_pixel_x = 1100
$check_pixel_y = 465


# return False if the 'Game over' message is displayed and True otherwise
Func CheckColor() 
   $color = PixelGetColor($check_pixel_x, $check_pixel_y)
   $bad = 0xF5E200 # The yellow that appears when the 'game over' message is displayed

   If $color == $bad Then
	  return False
   Else
	  return True
   Endif
EndFunc


# Click on a cell specified by its coordinates 
# (game board coordinates, not screen pixel coords)
Func CheckCell($row, $col)
   CheckColor()

   $x = $start_x + $row * $offset_x
   $y = $start_y + $col * $offset_y
   MouseClick($MOUSE_CLICK_LEFT, $x, $y)
   Sleep(50)
   If CheckColor() Then
	  ConsoleWrite("Checking " & $row & " - " & $col & " - OK" & @CRLF)
	  return 1
   Else
	  ConsoleWrite("Checking " & $row & " - " & $col & " - BAD" & @CRLF)
	  MouseClick($MOUSE_CLICK_LEFT, $check_pixel_x, $check_pixel_y)
	  return 0
   EndIf
EndFunc

ConsoleWrite("Starting autosweep_explore" & @CRLF)

$min_row = 0
$max_row = 28
$min_col = 0
$max_col = 28

$hFile = FileOpen("mines.csv", $FO_OVERWRITE)

For $i = $min_row To $max_row Step 1
   For $j = $min_col To $max_col Step 1
	  $tmp = CheckCell($j,$i)

	  FileWrite($hFile, $tmp)
	  FileWrite($hFile, ",")

	  Sleep(20)
   Next

   FileWrite($hFile, @CRLF)
Next
{% endhighlight%}
</p>
</details>

This should click on each cell, check if the yellow 'Game over' message appeared and mark the result with a 0 (mine) or 1 (safe) in a csv file.  
This part worked really well so I created a second autoit script that would play the game using the data from the csv file. That script can be seen below.

<details>
  <summary>autosweep_play.au3 (click to expand)</summary>
<p>
{% highlight autoit %}
#include <AutoItConstants.au3>
#include <FileConstants.au3>

# Coordinates for the top-left corner
$start_x = 880
$start_y = 110

$offset_x = 25
$offset_y = 25

# One of the pixels that turns yellow if we click on a mine
$check_pixel_x = 1100
$check_pixel_y = 465


# return False if the 'Game over' message is displayed and True otherwise
Func CheckColor() 
   $color = PixelGetColor($check_pixel_x, $check_pixel_y)
   $bad = 0xF5E200 # The yellow that appears when the 'game over' message is displayed

   If $color == $bad Then
	  return False
   Else
	  return True
   Endif
EndFunc

Func ClickCell($row, $col)
   $x = $start_x + $row * $offset_x
   $y = $start_y + $col * $offset_y
   MouseClick($MOUSE_CLICK_LEFT, $x, $y)

EndFunc

ConsoleWrite("Starting autosweep_play" & @CRLF)

$min_row = 0
$max_row = 28

$min_col = 0
$max_col = 28

$hFile = FileOpen("mines.csv", $FO_READ)

For $i = $min_row To $max_row Step 1
   For $j = $min_col To $max_col Step 1

	  $t = FileRead($hFile, 1)
	  FileRead($hFile, 1)

	  $row = $j
	  $col = $i

	  If $t == '1' Then
		 ConsoleWrite("" & $row & " - " & $col & " = 1 => CLICK" & @CRLF)
		 ClickCell($row, $col)

		 Sleep(60)
		 If CheckColor() == False Then
			ConsoleWrite("ERROR")
			Exit
		 EndIf
	  Else
		 ConsoleWrite("" & $row & " - " & $col & " = 0 => NOPE" & @CRLF)
	  EndIf


   Next

   FileRead($hFile, 2)
Next
{% endhighlight%}
</p>
</details>

After running the second script, I quickly realized that I made some mistakes on the first one and as a result I got most of the cells that were covered by the yellow 'Game over' message wrong (almost all of them were marked as empty even if they were not). This was probably the result of a timing error;  I suspect that the program was not resetting the game after the 'Game over' message appeared by clicking on a regular cell that was not covered by that message.  

However, when I took at the csv file in a text editor I noticed something.

![CSV data]({{site.baseurl}}/assets/img/De1CTF_2019/minesweeping_2.png){: .center-image}

It looks similar to a QR code (look at the corners). And qr codes are quite resilient so even if a part of them was damaged they can still be read.  
Maybe that's where the flag was hidden so I wrote a very quick python script to convert the 1s and 0s to a PNG file that I could scan more easily.  


<details>
  <summary>draw_qr.py (click to expand)</summary>
<p>
    {% highlight python %}
    from PIL import Image
import numpy as np

t = np.zeros([30, 30])

f = open('mines.csv')
i = 0
j = 0
for line in f:
	line = line.strip()
	for char in line.split(','):
		if char == '1':
			t[i][j] = 255
		elif char == '0':
			t[i][j] = 0
		j+=1

	i+=1
	j=0

img = Image.fromarray(t.astype('uint8'))

img.save("result.png")
    {% endhighlight %}
</p>
</details>

The result was a QR code that was a bit damaged in the center, as expected. It was only 29x29 pixels so I had to magnify it a bit to make it easily readable. This can be easily done using an image viewer like [nomacs](https://github.com/nomacs/nomacs) (open-source) or an image editor.  
The QR code that was obtained by me can be seen below.  

![Damaged QR code]({{site.baseurl}}/assets/img/De1CTF_2019/minesweeping_3.png){: .center-image}

Reading the QR code got me the following URL `http://qr02.cn/FeJ7dU`.  
After accessing the URL using a browser I was redirected to another page that contained the flag.

![Flag]({{site.baseurl}}/assets/img/De1CTF_2019/minesweeping_4.png){: .center-image}