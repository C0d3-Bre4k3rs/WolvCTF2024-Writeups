
# Crypto: tag-series-1
solver: [N04M1st3r](https://github.com/N04M1st3r)  
writeup-writer: [L3d](https://github.com/imL3d)   
___
**Author:** retu2libc  
**Description:**
> Don't worry, the interns wrote this one.  
  
**files (copy):** [chal.py](files/chal.py)  

## Solution

### Preview

In this challenge we need to input a plaintext string and a guess to the last block of it's AES (ECB) encryption 3 times in a row - if one of those guesses match we get the flag:  
  
For each 3 attempts a new random key is generated (16 bytes).  
We need to submit an input that has to follow these requirements:  
- Be unique from the other previous tries
- Be aligned blocks of 16 bytes
  
If we can match the ouput of the last block of the AES encryption, and our plaintext-input has started with the string: `"GET FILE: flag.txt"` we get the flag. Otherwise, **the last block of the AES encryption is being shown to us.**  

So, we basically need to find the output of a random AES encryption on our input.. How is that possible?! Especially when each input needs to be unique.  
We can't manipulate the output of a RANDOM encryption.. can we?  
Before we will showcase the solution, a basic understanding of how the `AES ECB` mode works is needed (the mode that is being used in this case), so we can properly try and exploit this algorithm and it's usecase.  
 
### AES (ECB) mode

The Advanced Encryption Standard (AES) is a symmetric encryption algorithm that was established by NIST in 2001. It has five standard modes of operation, but here we give an overview of the simplest mode, ECB.  
While using ECB mode, the message is divided into blocks, and each block is encrypted separately with the given key, as shown in the image below:  

![ECB Encryption](_images/ecb.png)  

For our purposes in the tag-series challenges, we don't really need to know much about the [Block Cipher algorithm](https://en.wikipedia.org/wiki/Block_cipher), apart from it being a *deterministic algorithm*, meaning that if we give it the same key and the same input, it will always give us the same output.  
  
ECB is the least recommended encryption mode, as it's main disadvantage is the lack of *cryptographic diffusion* - it can fail to hide data patterns between the text and the ciphertext.

### The exploit

Our goal is to find a way to find out what the last part of the block encryption will be for a certain payload, that is different from the ones that came before.  
This is only possible if we have sent some other payload beforehand, that has resulted in the same output that this payload will return - since we can't perdict the output of the Block Cipher Encryption itself.  
Putting it shortly: **We need to give two different payloads, that will result in the same last block of ciphertext.**
  
This task is fairly easy, as none of the previous blocks in the AES (ECB) mode encryption, nor the amount of the previous blocks really affect the output of the blocks that come after it. This implies that as long as we keep the last block the unchanged, the result will be the same!  
Armed with this knowledge, we create the following script:  
```python
from pwn import *

HOST, PORT = 'tagseries1.wolvctf.io', 1337

FIRST_BLOCK = b'GET FILE: flag.t'
LAST_BLOCK = b'xt' + b'give_me_flag:)'

conn = connect(HOST, PORT)

conn.recvuntil(b'== proof-of-work: disabled ==')

conn.sendline(LAST_BLOCK)
conn.sendline(b'Irrelevant')

conn.readline()
result = conn.readline()

conn.sendline(FIRST_BLOCK + LAST_BLOCK)
conn.sendline(result)

flag = conn.recv()
print(flag)

conn.close()
```  

This first gets the ciphertext output of the last block of our payload, then it adds another block before it, and receives the same result!  
Note, that the first block and the second block togther both create the requested string that should be at the start of the plaintext:  
`b'GET FILE: flag.t' + b'xt...'`
  
We get the flag🚩: `wctf{C0nGr4ts_0n_g3tt1ng_p4st_A3S}`  
  
To the [next one](https://github.com/C0d3-Bre4k3rs/WolvCTF2024-Writeups/tree/main/tag-series-2) ➡️
