---
layout: post
title:  "HTB Cyber Apocalypse 2023 - (Hardware) HM74"
date:   2023-03-23 15:00:04 +0300
categories: HTB_Cyber_Apocalypse_2023 CTF hardware
summary: Writeup for the HM74 (Hardware, Medium) from HTB Cyber Apocalypse 2023. Here you can see my janky solution to a challenge that involved error correction codes (Hamming Codes).
---

'HM74' was one of the challenges in the 'Hardware' category at HTB's Cyber Apocalypse 2023.  
Its difficulty was 'Medium' and it involved error correcting codes (Hamming Codes).  

My solution is almost surely not the 'most correct' ... but it worked.  

### Challenge description

The description of the challenge mentioned something about noisy data transmissions.  
We got access to a docker container and an archive.  
The archive contains a single file that we got was "encoder.sv", a Verilog module.  


### Noisy data transmission

Let's take a look at the docker container first.  
If I connect to the container, it sends me some binary data in a loop.  
![Data transmission]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/hm74/data_transmission.png){: .center-image}

The data doesn't decode to anything obvious for now.  

### The encoder, parity bits

Now, let's look at the Verilog module.  

{% highlight Verilog %}
module encoder(
    input [3:0] data_in,
    output [6:0] ham_out
    );
 
    wire p0, p1, p2;
 
    assign p0 = data_in[3] ^ data_in[2] ^ data_in[0];
    assign p1 = data_in[3] ^ data_in[1] ^ data_in[0];
    assign p2 = data_in[2] ^ data_in[1] ^ data_in[0];
    
    assign ham_out = {p0, p1, data_in[3], p2, data_in[2], data_in[1], data_in[0]};
endmodule

module main;
    wire[3:0] data_in = 5;
    wire[6:0] ham_out;

    encoder en(data_in, ham_out);

    initial begin
        #10;
        $display("%b", ham_out);
    end
endmodule
{% endhighlight %}

This encoder module takes a 4-bit signal and turns it into a 7-bit signal that contains the original 4 data bits and 3 parity bits.  
The parity bits are computed by XORing 3 of the data bits.  
This scheme is also known as Hamming(7,4), which you can read more about [here](https://en.wikipedia.org/wiki/Hamming(7,4)).  

The server is sending messages that were encoded that way and the messages get corrupted somehow before reaching us.  

I knew about Hamming codes, but I had no idea how to actually implement the error correction.  
On top of that, I thought that I can solve it without that because the server keeps transmitting the message.  
I was right ... but the solution is not the most pretty one.

### Decoding using Python

In order to test my solution offline, I took the received data and put it in a folder (after stripping anything that wasn't the message from it).
Analyzing that data, I noticed that every message had 136\*7 bits. This meant that the message is 136\*4 bits long.  
That would be 68 bytes (or 136 nibbles).  

For solving this, I implemented a python function that took 7bits (what I called an N-gram) from the received string, checked that the parity bits are correct and then returned the 4 data bits.  

{% highlight python %}
def decode_ngram(ngram):
	i0 = ngram[6]
	i1 = ngram[5]
	i2 = ngram[4]
	i3 = ngram[2]

	src_p0 = int(ngram[0])
	src_p1 = int(ngram[1])
	src_p2 = int(ngram[3])

	dst_p0 = int(i3) ^ int(i2) ^ int(i0) 
	dst_p1 = int(i3) ^ int(i1) ^ int(i0)
	dst_p2 = int(i2) ^ int(i1) ^ int(i0)

	error = False
	if dst_p0 != src_p0:
		error = True

	if dst_p1 != src_p1:
		error = True

	if dst_p2 != src_p2:
		error = True

	if not error:
		return i3 + i2 + i1 + i0
	else:
		return None
{% endhighlight %}

### Reconstructing the message

I knew that the message is 136 nibbles long, so I created an array and initialized it with 136 instances of '????'  
Then, I went through each n-gram of a message and if it was valid, I added it to the result array.  

In order to print the message stored in that array, I joined everything in that array, looked at every 8-gram and did the following:
- if there was any '?' character in the 8-gram, I decoded the character as '?'
- if there was no '?' character in the 8-gram, I parsed the 8-gram as binary (the int() function allows you to specify the base) and decoded that

The full code looks like this


<details>
  <summary>decode_v1.py (click to expand)</summary>
<p>
{% highlight python %}
repaired_text = []

def decode_ngram(ngram):
	i0 = ngram[6]
	i1 = ngram[5]
	i2 = ngram[4]
	i3 = ngram[2]

	src_p0 = int(ngram[0])
	src_p1 = int(ngram[1])
	src_p2 = int(ngram[3])

	dst_p0 = int(i3) ^ int(i2) ^ int(i0) 
	dst_p1 = int(i3) ^ int(i1) ^ int(i0)
	dst_p2 = int(i2) ^ int(i1) ^ int(i0)

	error = False
	if dst_p0 != src_p0:
		error = True

	if dst_p1 != src_p1:
		error = True

	if dst_p2 != src_p2:
		error = True

	if not error:
		return i3 + i2 + i1 + i0
	else:
		return None


def decode_line(line):
	ngram_length = 7
	ngrams = [line[i:i+ngram_length] for i in range(0, len(line), ngram_length)]
	fragments = []
	for index,ngram in enumerate(ngrams):
		decoded = decode_ngram(ngram)
		if decoded is not None:
			repaired_text[index] = decoded


def print_current_text():
	text = ''.join(repaired_text)
	repaired_bytes = [text[i:i+8] for i in range(0, len(text), 8)]
	repaired_chars = []
	for byte in repaired_bytes:
		if '?' in byte:
			repaired_chars.append('?')
		else:
			repaired_chars.append(chr(int(byte,2)))

	print(''.join(repaired_chars))


def main():
	# Initialize
	for i in range(136):
		repaired_text.append('????')

	f = open('encoded_text.txt', 'r')

	index = 0
	for line in f:
		line = line.strip()

		index += 1
		broken_nibbles = [nibble for nibble in repaired_text if nibble=='????']
		print('Iteration %d - Broken nibbles: %d' % (index,len(broken_nibbles)))
	
		decode_line(line)

	f.close()

	print_current_text()



if __name__=='__main__':
	main()
{% endhighlight%}
</p>
</details>

After running it, I got the following message:
**HTB{hmmßs1h\_s0m3\_ana1ysÑ5\_yu\_c6l\_6(7ract\_7h;\_h4mmYn9\_7Ý4o3nc\_Ve49}**

It's clear that I'm getting close, but this is nowhere near as close as it needs to be ...  
For troubleshooting, I printed the flag after every iteration (an iteration = another line/message from the server).  

When running it like that, I could see the issue: sometimes correct characters are overwritten with wrong ones.  
- Iteration 15-16 I can see 'w1th' becoming 'y1th', which seems like the wrong choice
- Iteration 17-18 I can see 'HTB' becoming 'HTA', which is surely the wrong choice

![Troubleshooting the decoder]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/hm74/wrong_flag.png){: .center-image}

I don't know why this happens. My best two guesses are:
- the message might get corrupted in *just the right way* so that the parity bits are also flipped and the message seems correct
- I don't know what I'm doing (very, very possible situation)
![]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/hm74/meme_no_idea_what_im_doing.jpg){: .center-image}

### Fixing the issue

A quick & dirty solution that I came up with is not adding any n-gram that was valid to the answer.  
Instead, I made a list of all valid values for every nibble in the answer.  

Then, at the end, I took the value that had the most occurrences for each nibble.

Another changes that I made:
- instead of relying on a file, I used pwntools to connect to the server and parse messages live
- after the first 30 iterations, the program stopped if nothing changed in the last 5 iterations

The full code looks like this

<details>
  <summary>decode.py (click to expand)</summary>
<p>
{% highlight python %}
import collections
from pwnlib.tubes.remote import remote

retrieved_data = {}
flag_length = 136

target_host = '139.59.176.230'
target_port = '31089'

def decode_ngram(ngram):
	i0 = ngram[6]
	i1 = ngram[5]
	i2 = ngram[4]
	i3 = ngram[2]

	src_p0 = int(ngram[0])
	src_p1 = int(ngram[1])
	src_p2 = int(ngram[3])

	dst_p0 = int(i3) ^ int(i2) ^ int(i0) 
	dst_p1 = int(i3) ^ int(i1) ^ int(i0)
	dst_p2 = int(i2) ^ int(i1) ^ int(i0)

	error = False
	if dst_p0 != src_p0:
		error = True

	if dst_p1 != src_p1:
		error = True

	if dst_p2 != src_p2:
		error = True

	if not error:
		return i3 + i2 + i1 + i0
	else:
		return None


def decode_line(line):
	ngram_length = 7
	ngrams = [line[i:i+ngram_length] for i in range(0, len(line), ngram_length)]
	fragments = []
	for index,ngram in enumerate(ngrams):
		decoded = decode_ngram(ngram)
		if decoded is not None:
			retrieved_data[index].append(decoded)


def get_current_flag():
	tmp_flag = []
	for i in range(flag_length):
		if retrieved_data[i]:
			values = retrieved_data[i]
			val = collections.Counter(values).most_common(1)[0]
			tmp_flag.append(val[0])
		else:
			tmp_flag.append('????')

	text = ''.join(tmp_flag)
	repaired_bytes = [text[i:i+8] for i in range(0, len(text), 8)]
	repaired_chars = []
	for byte in repaired_bytes:
		if '?' in byte:
			repaired_chars.append('?')
		else:
			repaired_chars.append(chr(int(byte,2)))

	return ''.join(repaired_chars)


def main():
	# Initialize
	for i in range(flag_length):
		retrieved_data[i] = []

	connection = remote(target_host, target_port)
	print('Connected to %s:%s' % (target_host, target_port))

	index = 0
	found = False
	last_flags = []
	while not found:
		index += 1

		line = connection.recv()
		line = line.decode('utf-8')
		line = line.strip()
		line = line.replace('Captured: ', '')

		flag = get_current_flag()
		last_flags.append(flag)
		print('Iteration %02d. Current flag: %s' % (index, get_current_flag()))
		decode_line(line)

		# Evaluate if we should stop
		if len(last_flags) > 30:
			if last_flags[-5] == last_flags[-4] == last_flags[-3] == last_flags [-2] == last_flags[-1]:
				print('Flag did not change in the last 5 iterations. Stopping')
				found = True

	connection.close()
	print('Final flag:')
	print(get_current_flag())


if __name__=='__main__':
	main()
{% endhighlight%}
</p>
</details>

After running it, I got a flag that looked valid.

![Getting the flag]({{site.baseurl}}/assets/img/HTB_Cyber_Apocalypse_2023/hm74/flag.png){: .center-image}

And after submitting it, that was indeed the correct flag:  
**HTB{hmm\_w1th\_s0m3\_ana1ys15\_y0u\_c4n\_3x7ract\_7h3\_h4mmin9\_7\_4\_3nc\_fl49}**


Can't wait to find out what was the proper way of solving this one...

### More hardware challenges

These are the other hardware challenges from this CTF
- Hardware 1/5 - Very Easy - [Critical Flight](/HTB-Cyber-Apocalypse-2023-Critical-Flight)
- Hardware 2/5 - Very Easy - [Timed Transmission](/HTB-Cyber-Apocalypse-2023-Timed-Transmission)
- Hardware 3/5 - Easy - [Debug](/HTB-Cyber-Apocalypse-2023-Debug)
- Hardware 4/5 - Easy - [Secret Code](/HTB-Cyber-Apocalypse-2023-Secret-Code)
- Hardware 5/5 - Medium - HM74 (you are here)
