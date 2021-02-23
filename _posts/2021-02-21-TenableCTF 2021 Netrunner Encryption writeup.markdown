---
layout: post
title:  "TenableCTF 2021 Netrunner Encryption writeup"
date:   2021-02-21 18:00:00 +0300
categories: CTF_writeup TenableCTF_2021 CTF_Crypto
summary: This is my writeup for the Netrunner Encryption  challenge. This cryptography challenge was part of TenableCTF 2021. 
---
This is my writeup for the `Netrunner Encryption` challenge. This challenge was part of [TenableCTF 2021](https://ctftime.org/event/1266).  
I'll try to make this be readable by someone who is barely familiar with these type of challenge (but who knows something about cryptography) so it will have a lot of details.  
If this is something you're familiar with, you'll probably be bored. Maybe, you might want to just grab the code.


We got an URL that lead to a PHP webpage. It was taking one input and was returned some encrypted text probably based on that input.  

![Netrunner webpage]({{site.baseurl}}/assets/img/TenableCTF_2021/netrunner_web.png){: .center-image}

We could also view the source of the page.

<p>
{% highlight php %}
<html>
<body>
  <h1>Netrunner Encryption Tool</h1>
  <a href="netrun.txt">Source Code</a>
  <form method=post action="crypto.php">
  <input type=text name="text_to_encrypt">
  <input type="submit" name="do_encrypt" value="Encrypt">
  </form>

<?php

function pad_data($data)
{
  $flag = "flag{wouldnt_y0u_lik3_to_know}"; 

  $pad_len = (16 - (strlen($data.$flag) % 16));
  return $data . $flag . str_repeat(chr($pad_len), $pad_len);
}

if(isset($_POST["do_encrypt"]))
{
  $cipher = "aes-128-ecb";
  $iv  = hex2bin('00000000000000000000000000000000');
  $key = hex2bin('74657374696E676B6579313233343536');
  echo "</br><br><h2>Encrypted Data:</h2>";
  $ciphertext = openssl_encrypt(pad_data($_POST['text_to_encrypt']), $cipher, $key, 0, $iv); 

  echo "<br/>";
  echo "<b>$ciphertext</b>";
}
?>
</body>
</html>
{% endhighlight %}
</p>

We can observe the following things by analyzing this code:
- The output is indeed based on the input (this is kind of expected...)  
To be more precise, the output is based on the input concatenated with the flag and then with some padding.
- The cipher used is AES-128 in ECB mode
- The padding is not random. In fact, it is easily predictable
- The key and flag are just there as examples. These are not their actual values.  
However, that didn't stopped me from trying them anyway

### AES-128-ECB

First, something about the cipher being used here.  
AES stands for [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (with its original name Rijndael, which I cannot pronounce). This is, as the name suggests, the standard for encryption and there are no known practical attacks against it when it is correctly implemented and used.  

As this challenge was solved by many people, the *correctly implemented/used" part from above is surely not true in this case.  
Let's see what were the mistakes that we can observe here:  
- First, the algorithm is used in ECB (Electronic CodeBlock) mode.  
This mode turns identical cleartext blocks into identical encrypted (ciphertext) blocks.  
A very good example involving an image can be seen on the [Wikipedia article](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB)) on block cipher modes.  

- And the second mistake is the use of predictable padding. By computing/guessing the length of the cleartext, we can compute characters that are used as padding

Also, the -128 part indicates that the algorithm will work on blocks of 128bits (or 16 bytes / 16 characters). This will become important a bit later.  

### Analyzing the length

First, let's take a look at the length of the returned ciphertext.  
An useful thing here would be to use the code and host the server on your own machine so that you're able to add some extra logging to help you see what's going on in there (ex: see the padding that is being used or other tricks like that).  
Just don't forget to change the URL to the one in the challenge after you confirm that your script works ...

I used requests to send .. well, requests to the server and I printed the length of the returned ciphertext in each case.  
The code that I used can be seen below.

<p>
{% highlight python %}
import requests
import re
import base64

pattern_response = re.compile('<h2>Encrypted Data:</h2><br/><b>(.+)</b></body>')

target_url = 'http://167.71.246.232:8080/crypto.php'


def get_ciphertext(cleartext):
	post_body = {
		'text_to_encrypt': cleartext,
		'do_encrypt': 'Encrypt'
	}

	response = requests.post(target_url, data=post_body)
	if response.status_code == 200:
		matches = pattern_response.search(response.text)
		if matches is None:
			print('Error. No matches were found')
			print(response.text)
		else:
			ciphertext_b64 = matches.groups()[0]
			ciphertext = base64.b64decode(ciphertext_b64)
			return ciphertext
	else:
		print('Error. Non-200 code returned')
		print(response.status_code)


def main():
	for i in range(0,32):
		my_input = '0' * i
		ciphertext = get_ciphertext(my_input)
		print('My length: %02d. Ciphertext length: %d' % (i, len(ciphertext)))


if __name__=='__main__':
	main()
{% endhighlight %}
</p>

Now, let's take a look at the result that I got
![Analyzing the length of the ciphertext]({{site.baseurl}}/assets/img/TenableCTF_2021/netrunner_length.png){: .center-image}

We notice the following:
- if our input is **between 0 and 5** characters the output will be **48 characters long** (3 blocks)
- if our input is **between 6 and 21** characters the output will be **64 characters long** (4 blocks)
- if our input is **more than 21 characters** characters the output will be **80 characters long** (5 blocks)

Also, because AES works on blocks of data, its input (cleartext) and output (ciphertext) will be the same size.  
It working on blocks is why padding is used; if the input is not long enough, padding will be added to make sure that the input is made of a whole number of blocks.  
Padding should be **unpredictable / random**. This not being the case here is part of why we will be able to get the flag without having the key.

### Padding

![Padding]({{site.baseurl}}/assets/img/TenableCTF_2021/diagram_padding.png){: .center-image}

Now let's take a look at how this implementation handles padding.

<p>
{% highlight php %}

function pad_data($data)
{
  $flag = "flag{wouldnt_y0u_lik3_to_know}"; 

  $pad_len = (16 - (strlen($data.$flag) % 16));
  return $data . $flag . str_repeat(chr($pad_len), $pad_len);
}

{% endhighlight %}
</p>

Let's consider the message to be the text that was received from the user concatenated with the flag.  
First, the length of the padding is determined by how many characters are missing in order to have a complete block, or in simpler terms, a message length that's a multiple of 16.  
So **16-(*length_of_message*%16)** characters will be added.  
This also means that when the message would not need padding because it is already the 'correct' length, some padding is still going to be added. In fact, the last block will only contain padding in that case.  

Now for the characters used for padding. The code from the *pad_data* function will use the character that has the ASCII code equal to the length of the padding for this.  
This means that characters with ASCII codes from 1 to 16 will be used.  
These characters are not printable and you won't be able to write them by using your keyboard (ex: in the browser).  
That doesn't matter. We won't be manually crafting the requests anyway. 

### ECB mode

Now to the other problem that I mentioned before: ECB mode.  
It is usual to use CBC mode or any other mode that would make sure that the same block is **NOT** encrypted into the same output block.  
However, ECB doesn't do that. If we see two identical blocks in the ciphertext, these bocks were identical in the cleartext too.

Let's consider the example from this drawing.  
![ECB mode problem]({{site.baseurl}}/assets/img/TenableCTF_2021/diagram_ECB.png){: .center-image}
Let's ignore the blocks that are blue for this example.  

Let's consider that the following are true:
- we know this cipher used here was AES in ECB mode
- we know the first block of the cleartext (the green one)
- we can see that the first and the last blocks of the ciphertext are identical
- we don't know the value of the last block from the ciphertext (the pink one) but we want to

Because the first and the last block from the ciphertext are identical and ECB mode was used, this means that the first and the last blocks from the cleartext are also identical.  
But we know the first block from the cleartext. This means that the last block has the same value as the one that we know.  

### Overview on this attack

And this is exactly how this attack is going to work: we're going to control the first block in order to make it identical to the last block.  
We'll do that by knowing that the last block contains padding characters and by trying to guess one character at a time.  
We can do that by controlling how many characters from the message get into that last block.  
Also note that the message has our input first and the flag last. This is what allows us to control the *first* block.

Now, let's say that we go for 5 blocks (80 characters). This means that our input should be between 22 and 38 characters.  
First, let's say that we want to check if this idea works, without revealing any characters (so without any bruteforcing). This would mean crafting an input message that makes the first block and the last block be identical.  
If we go for the minimum, 22 characters, we know that the last block contains only padding.  
To be precise, 16 characters of padding that have their value equal to 16 (or 10 in hex).  

The cleartext will look something like this:  

![Checking with an empty block]({{site.baseurl}}/assets/img/TenableCTF_2021/diagram_22len.png){: .center-image}
Note: p there means a character used for padding.

The code for actually doing this would be:  
( the get_ciphertext function was defined in the code snipped presented before)

<p>
{% highlight python %}

def check_null():
	padding_char = chr(16)
	crafted_cleartext = padding_char * 16
	ciphertext = get_ciphertext(crafted_cleartext)
	first_block = ciphertext[:16]
	last_block = ciphertext[-16:]
	if first_block == last_block:
		print('Exploit works (for null)')
	else:
		print('Exploit does not work')
{% endhighlight %}
</p>

If everything went right, we'll see the 'exploit works' message and we can also print out the ciphertext and check out the blocks.

**Useful trick:** I would recommend printing out the base64 output and then using [CyberChef](https://gchq.github.io/CyberChef/) to decode and view it. Chain the 'From Base64' and 'To Hex' operations in CyberChef and take a look at the blocks.  
Using hex is a good idea because you will probably have a good amount of characters that are not printable.  



![Getting the flag characters]({{site.baseurl}}/assets/img/TenableCTF_2021/netrunner_01.png){: .center-image}


![Getting the last characters]({{site.baseurl}}/assets/img/TenableCTF_2021/netrunner_02.png){: .center-image}