Hacky Easter 2016 writeup
------------------

This is a write-up of my solutions for the [Hacky Easter 2016](http://hackyeaster.hacking-lab.com/hackyeaster/) white-hat free hacking competition for education and fun.

The official solution document, which contains two of mine solutions, can be found [here](http://media.hacking-lab.com/hackyeaster/HackyEaster2016_Solutions.pdf).

Also keep in mind: These solutions may not be the fastest, most useful or most practical way of solving the given challenge and as always there is no guarantee that the code does not fry your computer and I am not to be held responsible for anything good or bad that happens to you and/or your computer because of my code snippets.

*I screwed up the images so they will be unavailable for some time.*

Table of contents
------------------

[1](#challenge-1) [2](#challenge-2) [3](#challenge-3) [4](#challenge-4) [5](#challenge-5) [6](#challenge-6) [7](#challenge-7) [8](#challenge-8) [9](#challenge-9) [10](#challenge-10) [11](#challenge-11) [12](#challenge-12) [13](#challenge-13) [14](#challenge-14) [15](#challenge-15) [16](#challenge-16) [17](#challenge-17) [18](#challenge-18) [19](#challenge-19) [20](#challenge-20) [21](#challenge-21) [22](#challenge-22) [23](#challenge-23) [24](#challenge-24) 

Challenge 1:
------------

1. Remove all spaces, newlines and the baby emote from the given text and you get

	`xthexyhiddenyystringyyisyymollycoddlexysoxxsimpley`
	
2. Remove all unnecessary x's and y's and replace with space where appropriate

	`the hidden string is mollycoddle so simple`

=> `mollycoddle`

Challenge 2:
------------

Using the [International maritime signal flags](https://en.wikipedia.org/wiki/International_maritime_signal_flags) the promotion code `enjoyafreshseabreeze` was extracted (some flags did not exist and were left out).

Challenge 3:
------------

The "little bird" phrase and the hashtag on the egg clues to twitter.
And after searching `#egg03 #nest`
=> https://twitter.com/8lueL1ttleB1rd/status/720201875945984001

Challenge 4:
------------

I installed the Hacky Easter app on the Android emulator BlueStacks and set Audacity to record the WASAPI loopback (the audio coming from my pc).

The recording looks like this:

![audacity_wave](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/04_audacity_wave.png)

Marking the region with the “beep” and selecting Analyse => Plot Spectrum:

![audacity spectrum](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/04_audacity_spectrum.png)

With the wanted frequency in the middle at 6200Hz. Do this 5 times and you got your egg!

Challenge 5:
------------

The video does not contain any clues so one of the 4 subtitle languages should contain our password.
Looking through the subtitle files which are just plain text you will find at the bottom of the Esperanto subtitles:
```
1
00:01:00.000 -=> 00:01:03.000
passphrase is "youtubelotitis"
```

Challenge 6:
------------

Klick on each elevator button and it will redirect you to `challenge06.html?floor={floornumber}` except button 13 which will bring you to `challenge06.html?sybbe=punatrzr`.
`sybbe` is rot13 of `floor` and `punatrzr` is rot13 of `changeme`.

We expect floor thirteen so rot13 of `thirteen` is `guvegrra`.
Thus, we need to visit `challenge06.html?sybbe=guvegrra` and voila there is the egg.

Challenge 7:
-------------

"The solution is in the solutions! Go back and scroll to 123!"
Probably referring to [Hacky Easter 2015](http://media.hacking-lab.com/hackyeaster/HackyEaster2015_Solutions_high.pdf) Solutions or [Hacky Easter 2014](https://www.hacking-lab.com/export/sites/www.hacking-lab.com/references/hackyeaster2014/Hacky-Easter-Solutions.pdf) Solutions and indeed on page 123 of the 2015 solutions:

	password: goldfish
	

Challenge 8:
------------

When changing your phone orientation the image either changes or goes back to the previous image.
After a few minutes of trial and error this sequence will get you the egg. Put your phone initially in Portrait orientation and tilt the top in the specified direction (Do not tilt between steps):

	left right right left left right right left right left left

Challenge 9:
-------------

The left picture represents an unplayed chess board where a 1 means a piece is present in this square and 0 the opposite.

When viewing each row as binary the rows produce the values in the red box e.g. `0b11111111 = 255` and `0b00000000 = 0`.
The provided text consists of many moves in Algebraic notation.
Using the nodejs library [node-chess](https://github.com/brozeph/node-chess) this simple nodejs program was created.

```node
var chess = require("chess");
var game = chess.create();

var moves = ["e4", "e5", "... all the other moves until", "Qf4+", "Rg3"]

moves.forEach(function (move) {
	game.move(move);
});

var squares = game.getStatus().board.squares;

var row = "";
for (var i = 0; i < squares.length; i++) {
	if (i % 8 == 0) { // next row
		console.log(line);
		row = "";
	}

	row += squares[i].piece ? "1" : "0";
}

console.log(row);
```
The output is then interpreted to binary. [This site helped me](http://www.binaryhexconverter.com/binary-to-decimal-converter).

	 output   decimal
	00000000     0
	10000011   131
	01000011    67
	00100100    36
	10101000   168
	01001100    76
	00100001    33
	00000001     1

So the result could be `0-131-67-36-168-76-33-1`  or `1-33-76-168-36-67-131-0`.
Test both and the second should get you the egg!

Challenge 10:
-------------

After analyzing the javascript code this formula was extracted and can be used to draw a pixel at (x, y) with the value v.

![Created with LaTeX](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/10_formula.png)

If the result is 1 the pixel is white if 0 blue.
This is nothing other than a mathematical way of getting the bit at a specific position.
Knowing this I wrote a simple python script to get the target plot:

```python
from PIL import Image 		# pip install pillow
im = Image.open("plot.png") # from http://hackyeaster.hacking-lab.com/hackyeaster/images/plot.png

def get(x, y):
	# each pixel is 4x4 in size
	r, g, b = im.getpixel((4 * x, 4 * y))
	return r + g + b == 255 * 3		# check if white


out = 0

for x in range(0, 106):		# 106 width
	for y in range(0, 17):	# 17 height
		bit = 1 if get(x, 16 - y) else 0
		
		out = (out << 1) | bit 	# appends bit AFTER out

		# Our image has (0, 0) in the top left.
		# The javascript sees the first pixel in the top right.
		# So normally we need to access the pixel by get(105 - x, y)
		# but because our bits are in the opposite direction as
		# they should be we flip the image horizontal and vertical
		# thus we need to access the data with get(x, 16 - y)

print(out * 17)
```
=> `17657949201581490152887262552977461550821547861463862839240664323911642807489754168164179332567124887458095049966838272395838833335464853226293169893063985683542234868393982863605544853380459104965350326137397441646486218169598347856207906783361422905911386919743769975974237367400302886153547602709155224361686573545769765610544442950623858438305126210029328322211845690185546981876389418111008050801364588449772605640341039292322155464883254208546726202316901388360683605114288184962864450110296056848249404578342545423849729556480`
which indeed matches the target.


Challenge 11:
-------------

It seems like every ring has all but one char multiple times present.
So I wrote this python script:

```python
rings = [
	"peoplle",
	"kakosfflaohls",
	"xjqsimpeljqninxmpel",
	"idlyrvchefpdopoefussycrlhuv",
	"cpidephjklonkamuffyiolyumsschajdett",
	"buwxhappysdfbkcooltqwreezymklcfdtvqhvsmrzxu",
	"beanstzkcdauueiyzybmvxgpjlcxnjqwoowqfdhilfrmgpsrtvk"
]

out = ""
for ring in reversed(rings):
	nring = list(ring)
	for char in ring:
		if nring.count(char) > 1:
			nring = [i for i in nring if i != char]

	out += "".join(nring)

print(out)
```
=> `hanisho`

Challenge 12:
-------------
The zip bomb must be decompressed then all git merges must be restored to get the next zip witch is then gonna be decompressed and so on.

I wrote this bash script to extract the zips until we meet one which needs a password:
```bash
#!/bin/bash

IFS='
'
shopt -s nullglob

if [ $# -gt 1 ]; then
	echo "Usage: $0 [start (default 1000)]"
	exit 1
fi

curr=${1:-"1000"}

if [ "$curr" -eq "1000" ]; then
	unzip -q "$curr.zip" || { echo "unzip failed"; exit 1; }
fi

while [ -n "$curr" ]; do
	echo $curr

	next=
	commits=$(git --git-dir=./$curr/.git --work-tree=./$curr log --pretty=%H) || exit 1;

	for commit in $commits; do
		git --git-dir=./$curr/.git --work-tree=./$curr checkout $commit . || { echo "checkout failed"; exit 1; }

		for zip in $curr/*.zip; do
			unzip -q -P "incorrect" "$zip" || { echo "done"; exit; }
			# panic if password is wrong
			# (we do this so it wont ask us and just fail)

			filename=$(basename "$zip")
			next="${filename%.*}" # without .zip
		done
	done

	if [ "$curr" -ne "0001" ]; then
		rm -fr $curr || exit 1;
	fi
	
	curr=$next
done
```

run it in the same directory as the 1000.zip
```sh
chmod +x extract
./extract
```
This will extract until 0045.zip which needs a password, that is in the name of one of the merges in the 0046 directory.
`Pass is fluffy99`

Extract the protected zip `unzip -q -P fluffy99 0046/0045.zip` and `./extract 0045` to let it finish extracting.

Folder 0001 should contain your egg.


Challenge 13:
-------------

The secret can't be in the first and second layer of the QR-codes because that would be too trivial for Hacky Easter so it must be in the third!

I wrote this small QR-code parser (unrelated to Hacky Easter):
https://github.com/TheSiki24/qrparse

This simple script will parse all the small 21*21 QR-codes:
```python
from qr import QR		# https://github.com/TheSiki24/qrparse
from PIL import Image
im = Image.open("wallpaper.jpg")

width, height = im.size

for x in range(0, width // 21):
	for y in range(0, height // 21):

		qr_data = []
		for ys in range(y * 21, y * 21 + 21):
			row = ""
			for xs in range(x * 21, x * 21 + 21):
				r, g, b = im.getpixel((xs, ys))
				row += "#" if r + g + b <= 60 else " "

			qr_data.append(row)

		if sum(row.count("#") for row in qr_data) < 28:
			# no qr code because of to few black pixel
			continue

		try:
			qr = QR(lambda x, y: qr_data[y][x] == "#")
			print(qr.text)
		except Exception, e:
			print("error: " + str(x) + ", " + str(y))


```
After inspecting the output I noticed, that all 52750 parsed QR-codes start with `xx` except the one containing the password:

=> `fractalsaresokewl`

Challenge 14:
-------------

To not trip into traps or having to deobfuscate code just ask your browser what he executes:
Inspect the form

![form](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/14_form.png)

Ask console what the function is and click on it.

![console](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/14_console.png)

Perfect source code:

![javascript source](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/14_jssrc.png)

=> user: `elsa`

I'm to lazy to use my brain to get the password... lets bruteforce:
```python
import itertools

for val in itertools.imap("".join, itertools.permutations(map(str, range(10)))):
    ok = True
    for i in range(10):
        part = int(val[0:i + 1])

        if part % (i + 1) != 0:
            ok = False
            break

    if ok:
        print(val)
        exit()
```
=> pw: `3816547290`

Challenge 15:
-------------

Mount image using `mount` command on linux or a mount tool on Windows.

`wolf.jpg` contains `pig 2: Cassadee` in the EXIF section "comments".
[This website](http://regex.info/exif.cgi) helped me (on windows just check the properties of the file).

`story.txt` contains unnecessary tabs and spaces between and after lines.
This is called "Whitespace steganography" and using the tool [SNOW](http://www.darkside.com.au/snow/) the string `pig 1: Filbert` was extracted.

Since `song.mp3` did not contain any clues on the cipher used I had to guess. I found the program [MP3stego](http://www.petitcolas.net/steganography/mp3stego/) which could successfully decode the mp3 and yield `pig 3: Wynchell`.

What I did not know was that when the decoder asks you for a passphrase you don't actually have to enter one. So, of course, I tried way too long to crack the password.

Challenge 16:
--------------
Using [dex2jar](https://github.com/pxb1988/dex2jar) and [jdgui](http://jd.benow.ca/) the app was decompiled which was not actually needed because the app is so kind and sends the hmac secret `eggsited` with every message.
The hmac used is just the color in hex with the key.

Using [this hmac generator](http://www.freeformatter.com/hmac-generator.html#ad-output) and selecting `ffff00` (yellow in hex) as message and `eggsited` as key the hmac `1da02c68080863fa302c20c3312371f4e365a5f9` was generated.

Now you just need to GET http://hackyeaster.hacking-lab.com/hackyeaster/egg?code=ffff00&key=eggsited&hmac=1da02c68080863fa302c20c3312371f4e365a5f9.

The returned base64 encoded png file can be converted online [here](http://www.freeformatter.com/base64-encoder.html) and after renaming the downloaded file your egg should present itself.

Challenge 17:
-------------

The language reminded me of the programming Language [Logo](https://en.wikipedia.org/wiki/Logo_%28programming_language%29).

The only missing commands were `hop, egg and lineofeggs`
After a **lot** of trial and error these are the functions I created: 

```logo
to hop :i
	forward :i 
end

to backhop :cnt
	left 180
	hop :cnt
	right 180
end

to egg
	backhop 5
	right 90
	forward 5
	left 90

	filled 000000 [
		forward 10
		left 90
		forward 10
		left 90
		forward 10
		left 90
		forward 10
		left 90
	]

	hop 5
	left 90
	hop 5
	right 90
	fill
end

to lineofeggs :cnt
	repeat :cnt [egg hop 10 ]
	backhop 10
end
```
also 

	cs
	penup

must be called before the supplied commands.

Using this online [Logo Interpreter](http://www.calormen.com/jslogo/) the turtle drew this:

![turtle with qr code](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/17_turtle.png)

Challenge 18:
-------------

![what a noob](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/18_noobcode.png)

fixed:
```csharp
public static string generateKey() {
	int a = 1 * 1111;
	int b = 2 * 1111;
	int c = 3 * 1111;
	int d = 4 * 1111;

	for (int i = 0; i < 1000; i++) {
		if (c > d) {
			c *= 2;
			d *= 2;
		}
		
		a = (a + b) * (c + d);
		b *= 3;

		int tmp = c;
		c = d;
		d = tmp;

		a %= 10000;
		b %= 10000;
		c %= 10000;
		d %= 10000;
	}

	return a + "X" + b + "X" + c + "X" + d;
}
```


Challenge 19:
-------------

After reversing the binary with ida I came up with this approximate c pseudo code:
```c
#include "stdlib.h"
#include "stdio.h"
#include "stdint.h"
#include "string.h"

int main(int argc, const char** argv) {

	char raw[18] = {
		// the 17 values to find
		, 0
	};

	char* input = raw;

	if (strlen(input) <= 16)
		goto fail;

	int i = 0;

	int c = *input++;

	int v8 = 0;
	int v9 = c;

	int va = 0;
	int vb = 0;
	int vc = 0;
	int vd = 0;

	while (1) {
		if ((i % 3) == 0)
			vd += c;
		
		if ((i & 0b11) == 2)
			vc += c;

		i++;

		va = (uint8_t)(v9);
		vb = (uint8_t)(v8);
		vc = (uint8_t)(vc);
		vd = (uint8_t)(vd);

		if (i == 16)
			break;

		c = *input++;

		v9 = va + c;
		v8 = vb + c;

		if (!(i & 1))
			v8 = vb;
		
	}

	if (va == 133 && vb == 67 && vc == 176 && vd == 79)
		printf("yuss %i %i %i %i\n", va, vb, vc, vd);
	else
fail:
		printf("nope %i %i %i %i\n", va, vb, vc, vd);
	
	return 0;
}
```

and after logging every operation this is the flow of the variables:
```
# charindex operation
0:	vd += c
2:	v8 = vb
2:	vc += c
3:	vd += c
4:	v8 = vb
6:	v8 = vb
6:	vd += c
6:	vc += c
8:	v8 = vb
9:	vd += c
10:	v8 = vb
10:	vc += c
12:	v8 = vb
12:	vd += c
14:	v8 = vb
14:	vc += c
15:	vd += c
```
Which could be used to construct these terms:
(numbers in sum are char index, numbers right of == are the real target numbers)
```
sum(1, 3, 5, 7, 9, 11, 13, 15) == 67
sum(0, 3, 6, 9, 12, 15) == 79
sum(2, 6, 10, 14) == 176
sum(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15) == 133
```
And after setting each char value to its initial value of 1 I could, with a bit of patience, solve this equation by hand.

These are one of the many possible values for the chars:
`74, 60, 102, 1, -86, 1, 1, 1, -99, 1, 1, 1, 1, 1, 72, 1`
or in hex `0x4a3c6601aa0101019d01010101014801`
(make sure no `0x0a` (`\n`) are included)

Sending this and newline to the server:
```bash
echo -ne "\x4a\x3c\x66\x01\xaa\x01\x01\x01\x9d\x01\x01\x01\x01\x01\x48\x01\n" | nc hackyeaster.hacking-lab.com 1234
```

=> `ida.lo\/es.you` And I love you IDA :)

Challenge 20:
-------------

The 3 tables were restored from the mysql dump to a running mysql server.

After joining all three tables on userid and keeid those are the interesting fields:

`blahb (blob), kee (key), puzz (pass) and sawlt (salt)`

The procedures `DeekryptKee, GetPuzzMishMash and KryptKee`  were provided by the dump.
Using the length and encoding of the input and output parameters of the procedures I guessed that this is what happened:

 1. `GetPuzzMishMash` was called with the user supplied password and the corresponding salt. The result was stored in `puzz`.
 2. `KryptKee` was called with the user supplied password and the key used for the aes encryption of the blob.
 3. The blob was aes encrypted using `AES_ENCRYPT`.

`KryptKee` is using 10000 nested sha1 so this function could not be reversed to get the user supplied password.

`GetPuzzMishMash` can be used because it is only one salted sha1 calculation:
`sha1(salt + "." + pass + "." + salt)`

Lets hit it with a password list (get the list [here](https://github.com/danielmiessler/SecLists/blob/master/Passwords/10_million_password_list_top_1000000.txt) and name it `pwlist.txt`):

```python
import hashlib

hashs = {
	"de2278f5bcafcbb097ecc1fb54e5ab8a9e912c55": "efgh",
	"943f9ecbbd91306a561d0e3c15e18ee700007083": "abcd",
	"0cf32f8f418659f23f8968d4f63ea5c98b39f833": "zyxw",
	"1742ae4507fc480958e2437104e677e70aa5e857": "jklm",
	"915d253cb5ba6f0a220bca83e2d6d3258af15e68": "nmlk"
}

hashs = dict((k.decode("hex"), v) for k, v in hashs.iteritems())

with open("pwlist.txt", "rb") as file:
	for line in file:
		for target, salt in hashs.iteritems():
			h = hashlib.sha1(salt + "." + line.strip() + "." + salt)

			if h.digest() == target:
				print(h.hexdigest() + " found " + line.strip())
				del hashs[target]
				break
```

=> `0cf32f8f418659f23f8968d4f63ea5c98b39f833 found snakeoil`

Knowing the password the key was decrypted:

```sql
call DeekryptKee('snakeoil', '1ABF4B7CD25C61FDF0E74EC2BFB43BD1C2D8ECD803AFA3AA376F4C0000052813', @out);
SELECT @out;
```

=> `jpP8HeoEC5OCCBqdf9N3`

Finally decrypt the blob
```sql
SELECT AES_DECRYPT(blahb, 'jpP8HeoEC5OCCBqdf9N3') from fyle;
```
And surprise, one turned out to be a png file :)

![wow a png](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/20_wowapng.png)

Just export and there is the egg.

Challenge 21:
-------------

Just do this 4 times and search for online decipher:

![enter image description here](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/21_googleisop.png)

Caesar cipher: [Use this website](http://www.xarg.org/tools/caesar-cipher/) and select 'guess' as key.
=> `AS A RULE MEN WORRY MORE ABOUT WHAT THEY CANT SEE THAN ABOUT WHAT THEY CAN PASSWORD IS CARTHAGO`

Polybius cipher: [Use this website](http://www.braingle.com/brainteasers/codes/polybius.php).
=> `there is no witness so dreadful no accuser so terrible as the conscience that dwells in the heart of every man peloponnese is the password`

Vigenère cipher: [Use this website](http://www.mygeocachingprofile.com/codebreaker.vigenerecipher.aspx) and select the option without a key.
This results in almost correct messages with similar looking keys (`parispanis` and similar). What key could it possible be??? `panispanis` or `parisparis`

=> `phrase you need is alchemy...`

Playfair cipher: [Using this project](https://github.com/N8Stewart/PlayfairCrack) which needs to be compiled in linux the cipher was cracked (spaces were added manually): 
=> `THE PLAYFAIR CIPHER WAS THE FIRST PRACTICAL ... PASSWORD IS BLETCHLEY`

Challenge 22:
-------------

The supplied code snippet was looking just like the source code of the sha1 hash function and indeed after looking through [this](https://github.com/ajalt/python-sha1/blob/master/sha1.py) implementation the only noticeable change I found were the initial state variables h0 - h4.

After modifying the code the hashes could be cracked using [this](https://github.com/danielmiessler/SecLists/blob/master/Passwords/merged.txt.tar.gz) password list.

Just use the changed implementation and feed it with every password... Just like [Challenge 20](#challenge-20)

	0: zombie
	1: Denver1
	2: Placebo
	3: SHADOWLAND

Challenge 23:
-------------

When reading the pixel values of the image and test if they are even or odd this is the result:

![blue channel even](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/23_coincidence.png)

Coincidence? I don't think so!

So obviously the data is encoded so that one bit is an even / odd pixel for each of the 3 channels.
After trying for two months how these bits should be interpreted the simplest solution did not come to my mind:
Just merge the 3 bit streams to one file. Why did I never try this :( ARGHHH

```python
from PIL import Image
from bitarray import bitarray

im = Image.open("heizohack.bmp")

pixels = zip(
	im.getdata(0), # r
	im.getdata(1), # g
	im.getdata(2)  # b
)

stream = bitarray()
for pixel in pixels:
	r, g, b = pixel

	stream.append(r % 2 == 1)
	stream.append(g % 2 == 1)
	stream.append(b % 2 == 1)

with open("merged.png", "wb") as f:
	stream.tofile(f)
```
This will provide you with this png image:

![merged](https://raw.githubusercontent.com/skirsten/hacky-easter-2016/master/res/23_merged.png)

The red bit is the data to extract and if r, g, b and a xor'd together are 1 the data is [authentic](https://en.wikipedia.org/wiki/Message_authentication_code) and should be included in the output.

```python
im = Image.open("merged.png")

pixels = zip(
	im.getdata(0), # r
	im.getdata(1), # g
	im.getdata(2), # b
	im.getdata(3)  # a
)

def get_bit(val, offset):
	mask = 1 << (7 - offset)
	return val & mask

stream = bitarray()
for pixel in pixels:
	for i in range(8):
		r, g, b, a = map(lambda p: get_bit(p, i), pixel)

		if r ^ g ^ b ^ a == 1: # authenticity test
			stream.append(r != 0) # int to bool

print(stream.tobytes())
```
=> `lostinthewoooods`

Challenge 24:
-------------

Using the provided source code we first need to generate a short url whose long url starts with `http://evileaster.com` and collides with the official short url `IU66SMI`:
```python
import hashlib, base64, itertools, string

query = base64.b32decode("IU66SMI=")
prefix = "http://evileaster.com/"

i = 0
for l in range(1, 10):
	for suffix in itertools.imap("".join, itertools.product(string.ascii_lowercase, repeat=l)):
		url = prefix + suffix

		if hashlib.sha256(url.encode("UTF-8")).digest()[0:4] == query:
			print(url)
			exit()

		i += 1

		if i % 1000000 == 0:
			print(i, suffix)
```
Note that pytohn is very slow and I recommend implementing a cracker in C++ using [this sha246 source code](http://bradconte.com/sha256_c) which in my case did speed up the cracking a lot. Also, look into using ocl hashcat!

One possible url is `http://evileaster.com/jghirfj`

Next, we need to find the censored key.
The key is 5 chars long (1 + length(key) * 3 == 16) and thus can be easily bruteforced:

First, we shorten a sample url eg. `http://asdf` and log the requests our browser does. The browser recieves the shorted url `http://crunch.ly/TIBSPOQ` and the ticket in base64.

Knowing this we can bruteforce the cryptTicket function to get the 5 char secret key.
```python
import base64, string, itertools
from Crypto.Cipher import AES


full_url = "http://asdf"
short_url = "http://crunch.ly/TIBSPOQ"
plain = base64.b64encode(full_url) + "@" + base64.b64encode(short_url)
target = base64.b64decode("ykryhYxlXP/yUKSz4xyIPR9hrmIgq+TG8ty2vy/TP/XSDUNuSBkX16JGijd9f8QpK6DtL2sJDMLgnOf/zCw3qw==")

plain_block = plain[0:16]
target_block = target[0:16]

mask = string.ascii_lowercase + string.ascii_uppercase

i = 0
for key in itertools.imap("".join, itertools.product(mask, repeat=5)):
	aes = AES.new("x" + key + key + key, AES.MODE_CBC, "hackyeasterisfun")

	if aes.encrypt(plain_block) == target_block:
		print(key)
		exit()

	i += 1
	if i % 1000000 == 0:
		print(i, key)
```
This provides us with the key `tKguF` which we can use to craft our own token:

```python
import base64
from Crypto.Cipher import AES

full_url = "http://evileaster.com/jghirfj"
short_url = "http://crunch.ly/IU66SMI"
plain = base64.b64encode(full_url) + "@" + base64.b64encode(short_url)
key = "tKguF"
aes = AES.new("x" + key + key + key, AES.MODE_CBC, "hackyeasterisfun")

BS = 16
pad = lambda s: s + (BS - len(s) % BS) * chr(BS - len(s) % BS)

enc = aes.encrypt(pad(plain))
print(base64.b64encode(enc))
```
=> `Hgo3UsPWbH+4kkfQwZ0dOIeuq+Qsa9B10/AgrVa+ykCWWsOqBV2+tk+69gsDJnrINX+q/+coOiRo7jKv6WTrMflA9VFgEoHTfGTrwtdRZAg=`

Now we just have to save our shorturl:
The javascript suggest requesting `http://hackyeaster.hacking-lab.com/hackyeaster/crunchly/crunch?service=save&ticket=TICKET`

I just opened it in my browser (remember to [urlencode](http://meyerweb.com/eric/tools/dencoder/)).

Finally, open the shorturl `IU66SMI` and you should be greeted with an egg.


Epilog
------

Thank you for reading through this way too long writeup of my solutions for the [Hacky Easter 2016](http://hackyeaster.hacking-lab.com/hackyeaster/) hacking competition.

*Simon Kirsten*