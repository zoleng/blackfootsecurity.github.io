---
layout: post
current: post
cover: assets/images/swampctf.png
navigation: True
title: We Three Keys Writeup
date: 2019-03-24 18:10:00
tags: Writeup CTF Crypto
class: post-template
subclass: "post tag-crypto tag-ctf"
author: atomheartother
---

We Three Keys was a fairly straightforward crypto challenge which required a bit of thinking, but was slightly disappointing in the end.

## The Challenge

We're given access to a server which lets us encrypt and decrypt any data. We can also pick which key out of any three keys we want to use.

Supposedly the goal of the challenge is to figure out what the three keys are. We're given the server code, and I'll include the important part:

```py
def encrypt_message(key, IV):
    ...

def decrypt_message(key, IV):
    ...

while True:
    print("1) Encrypt a message")
    print("2) Decrypt a message")
    print("3) Choose a new key")
    print("4) Exit")
    choice = str(raw_input("<= "))
    if choice == "1":
        encrypt_message(key, key)
    elif choice == "2":
        decrypt_message(key, key)
    elif choice == "3":
        key = new_key()
    else:
        exit()
```

Wow, wait, they're using the key as IV? That's a very clear weakness right here.

## IV vs. Key

IVs are sometimes misunderstood. IVs are meant to let you reuse a key between two encryption rounds. What they do is bring in an element of randomness to the encryption mechanism in AES_CBC, so that if I encrypt the same message twice with the same key, I can get different results using the same key.

Deriving the IV from the key is an awful idea becaue it completely negates the point of having an IV. I was hoping that this challenge would play on that fact, but the answer was much simplet than that and didn't involve this weakness in any way, instead it simply was about acquiring the IV by using the CBC decryption mechanism.

## CBC decryption

So I want to figure out the IV. The diagram below illustrates what AES_CBC decryption looks like:
![CBC decryption](/assets/images/threekeys/cbcdec.png)

So to get the IV I need to solve:
`IV = plaintext ^ unciphered`

Where `plaintext` is the final block of data post-xor, and `unciphered` is the data right after it's been decrypted, but not xor'd. I'm always given the plaintext, so the real question is, how do I find the decryption output pre-xor?

## Solving it

Turns out the answer is pretty simple: I decrypt the same block, only composed of 0s, twice. The first block will get decrypted, then xor'd with the IV, and that'll give me `plaintext`. The second block will also be decrypted, then every byte will be xor'd with `0`, which means every byte will remain the same - giving me `unciphered`.

I was feeling lazy so I didn't even write a script to do this three times, I did it manually and pasted the two blocks into this script each time:

```py
p1 = bytes.fromhex(BLOCK1)
p2 = bytes.fromhex(BLOCK2)

r = bytes()
for i in range(16):
    r += bytes([p1[i] ^ p2[i]])

print(r)
```

The flag was `flag{w0w_wh4t_l4zy_k3yz_much_w34k_crypt0_f41ls!}`
