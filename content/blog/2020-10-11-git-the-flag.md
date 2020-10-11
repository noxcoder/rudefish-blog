---
title: Git The Flag
date: 2020-10-11T00:41:13.425Z
tags:
  - ARPCon
  - CTF
  - Git
  - Log
draft: true
---
Points: 700

(At the time of this write up, the Arpcon CTF has ended and I no longer have access to the challenge details for screenshot).

We were provided with a `secret.zip` file. Extracting the file with `unzip secret.zip` and navigating to the extracted folder, looks like we got ourselves a git repo.

![](/images/git.png)

We start by checking the commits using `git log` and we can find some interesting commits.

![](/images/git_log.png)

First we checked out the `added bin` commit and we have the flag.bin, which is the encrypted flag

![](/images/flag_bin.png)

That's seems to be all in that commit, so we check another one. Checking out the `some codes` commit, we have a `main.py `

```python
import pyaes
import base64
import os
from dotenv import load_dotenv


load_dotenv()
SECRET = os.getenv("SECRET")

flag=open("flag.txt").read()

aes = pyaes.AESModeOfOperationCTR(SECRET.encode())
ciphertext = aes.encrypt(flag)
base_cipher = base64.b64encode(ciphertext)

with open("flag.bin","wb") as file:
  file.write(base_cipher)

```

Now it looks like we can reverse the code and get our flag, but first we need to find the secret, so we move to checkout another commit. Checking out the `remove flag.py` commit,we have the `.env` file which contains the secret.

![](/images/secret.png)

Now, let's get started. Import the necessary stuff first.

```python
import pyaes
import base64
```

Then set our secret 

```python
SECRET = "qyrJxvPFAbo8YDYNrCzSSj1DkHjGpLtW"
```

You can take the flag.bin and read from it, here I will just set it directly in the script.

```python
flag_enc = "/A8Mm2hbkL5yEuss8EelU8xaBsELiJOpilIqpp8="
```

Next the actual reversing

```python
aes = pyaes.AESModeOfOperationCTR(SECRET.decode())
flag_plaintext = aes.decrypt(base64.b64decode(flag_enc.encode()))
print("FLAG: %s " % flag_plaintext)
```

We set the aes mode to decode, then base64 decode the encrypted flag and then use aes to decrypt. The full script is 

```python
import pyaes
import base64

SECRET = "qyrJxvPFAbo8YDYNrCzSSj1DkHjGpLtW"

flag_enc = "/A8Mm2hbkL5yEuss8EelU8xaBsELiJOpilIqpp8="

aes = pyaes.AESModeOfOperationCTR(SECRET.decode())

flag_plaintext = aes.decrypt(base64.b64decode(flag_enc.encode()))

print("FLAG: %s " % flag_plaintext)
```

And we have the flag as: `arpcon{h1570RY_4lw4Y5_M4773R}`

```python
FLAG: arpcon{h1570RY_4lw4Y5_M4773R}
```