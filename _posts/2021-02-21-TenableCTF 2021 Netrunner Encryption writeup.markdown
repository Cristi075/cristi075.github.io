---
layout: post
title:  "TenableCTF 2021 Netrunner Encryption writeup"
date:   2021-02-21 18:00:00 +0300
categories: CTF_writeup TenableCTF_2021 CTF_Crypto
summary: This is my writeup for the Netrunner Encryption  challenge. This cryptography challenge was part of TenableCTF 2021. 
---
This is my writeup for the `Netrunner Encryption` challenge. This challenge was part of [TenableCTF 2021](https://ctftime.org/event/1266).

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