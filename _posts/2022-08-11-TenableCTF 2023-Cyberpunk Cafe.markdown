---
layout: post
title:  "TenableCTF 2023 - Cyberpunk Cafe writeup"
date:   2023-08-11 00:00:00 +0300
categories: Tenable_CTF_2023 CTF stego
summary: Writeup for the Cyberpunk Cafe challenge from TenableCTF 2023. This was a relatively simple steganography challenge.
---


Cyberpunk Cafe was a challenge at TenableCTF 2023 from the 'Stego' category.  
We received a single text file for this challenge.

![The challenge]({{site.baseurl}}/assets/img/TenableCTF_2023/cyberpunk_cafe/cyberpunk_cafe_challenge.png){: .center-image}


The challenge.txt file had the following contents

<div style="word-wrap: break-word;">
{% highlight shell %}

Thanks for visiting the Cyberpunk Cafe! Please use the code below to see what we have to offer:

0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001111111000100111110110000011111110000000010000010101111101111010000100000100000000101110101010011110111100101011101000000001011101010000010110010100010111010000000010111010101011111000100110101110100000000100000101010110000110010001000001000000001111111010101010101010101011111110000000000000000010111110010000100000000000000000001001111100010110001100110111110000000001111110000010101110101111011001010000000010011011110111101111100111101110100000000011011000011111100100100101001001000000000000101111011101101011011110000000000000011110100010001100010100111100100100000000001010100000010010010011011010101000000001101000110100111111010010101010000000000000011111111001100110101110100101100000000000001001100101000000111101001001000000001001111111111000110100010010010010000000010111001111000010111101101000101000000000001010111001111010100000101101000000000000011110110010111010111100111010010000000011100111110010100110100100100000100000000001100010111100101111010111111000000000001111011000001110110100001111101110000000000000000100101011100000110001010100000000111111101111010010010110101010101000000001000001011001110010000101000110100000000010111010001010000011111111111001100000000101110100110011011110111110011101000000001011101011011000100101111000110110000000010000010000011110010011011000100000000000111111100011011111010000111000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
{% endhighlight%}
</div>

The first thing that I could think of when seeing that string was that maybe it forms some patterns if printed with a specific number of characters per row.  

So I looked at the length of that string: 1681 characters.  
If you factorize 1681, you will see that it is 41 squared. This gave me quite an obvious option: a 41x41 square.

Also, if it is a square it is probably going to be a QR code.  
This part is quite similar to an [older challenge](/De1CTF-2019-Mine-Sweeping-Writeup) that I solved, so I used a similar approach.


{% highlight python %}
from PIL import Image
import numpy as np

# Target string goes here
obfuscated = ''

# The obfuscated text is 1681 characters long
# 1681 = 41x41, we can probably arrange it in a 41x41 image
WIDTH = 41
HEIGHT = 41

image_pixels = np.zeros([41,41])

for line_nr in range(41):
	start = line_nr * WIDTH
	stop = (line_nr+1) * WIDTH
	line = obfuscated[start:stop]
	for pixel_nr, pixel in enumerate(line):
		if pixel == '1':
			image_pixels[line_nr][pixel_nr] = 255
		elif pixel == '0':
			image_pixels[line_nr][pixel_nr] = 0

img = Image.fromarray(image_pixels.astype('uint8'))
img.save('result.png')
{% endhighlight %}

The code is quite simple, it starts with a 41x41 numpy array and then it goes through the array of zeros and ones: 
- if the current char is a 0 it turns the corresponding pixel black
- if the current char is a 1, the corresponding pixel turns white  

After running it, I got a 41x41 black and white image.  
This is how it looks after magnifiying it a bit (it was 41x41 ... so it was small)

![Resulting QR Code]({{site.baseurl}}/assets/img/TenableCTF_2023/cyberpunk_cafe/qr_code_magnified.png){: .center-image}

Scan the code and you get the flag: **flag{br1ng_b4ck_phys1c4l_menu5}**