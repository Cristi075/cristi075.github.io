---
layout: post
title:  "TenableCTF 2022 Characters of Shakespeare's Plays Writeup"
date:   2022-06-13 20:00:00 +0300
categories: CTF_writeup TenableCTF_2022 Steganography Stegano
summary: A short writeup for "Characters of Shakespeare's Plays" (TenableCTF 2022)
---

For this challenge we got a text file that was around 508KB large.  

The first few lines of the file are

```
Project Gutenberg's Characters of Shakespeare's Plays, by William Hazlitt

This eBook is for the use of anyone anywhere at no cost and with
almost no restrictions whatsoever.  You may copy it, give it away or
re-use it under the terms of the Project Gutenberg License included
with this eBook or online at www.gutenberg.org
```

Because of these lines, we know where this text is from and where we can get it (Project Gutenberg).  
So, we get the original text from [here](https://www.gutenberg.org/ebooks/5085). We can select "Plain Text UTF-8" and we get a similar file to the one that we already have.  

Now, let's compare them.  
It looks like the original file is larger, having 518KB.  
We compare the two files with diff and differences are detected but diff might not be the right tool for this.  
Note: Later on, I leared about [kdiff3](https://kdiff3.sourceforge.net/), which might've made this easier.

For now, let's take a look at the content of the smaller file (the one from the challenge).
In the first few lines we alread notice something.

```
Title: Characters of Shakespeare's Plays

Author: William Hazlitt

Release Date: January 29, 2011 [EBook #5085]

Language: English


*** START OF THIS PROJECT GUTENBERG EBOOK CHARACTERS OF SHAKESPEARE'S PLAYS ***




Produced by Steve Harris, Charles Franks and the Online
Distributed Prooreading Team.
```

The last line has "Prooreading" instead of "Proofreading" as we'd expect.  
The missing character (f) is also the first character of the message that we're looking for (flag{...).

By looking at the next few lines, the first four missing characters are "flag". This is probably the right path.

My solution to check this was to make a Python script that finds the differences between the two files, puts them in a string and displays them.  
For finding the differences, I used difflib

{% highlight python %}
import difflib

def main():
        infile_1 = open('orig.txt', 'r')
        infile_2 = open('hazlitt.txt', 'r')

        lines1 = []
        lines2 = []

        for line in infile_1:
                lines1.append(line.strip())

        for line in infile_2:
                lines2.append(line.strip())

        infile_1.close()
        infile_2.close()

        flag = []
        # Both files have the same number of lines (manually checked this)
        for line_index in range(0, len(lines1)):
                line1 = lines1[line_index]
                line2 = lines2[line_index]
                if line1 != line2:
                        # Lines are different => Find the first different character
                        for difference in difflib.ndiff(line1,line2):
                                if(difference[0] != ' '):
                                        flag.append(difference[2])

        print(''.join(flag))


if __name__=='__main__':
        main()
{% endhighlight %}

We run this and see that we get a flag for this challenge.

![Flag]({{site.baseurl}}/assets/img/TenableCTF_2022/shakespeare_flag.png){: .center-image}
