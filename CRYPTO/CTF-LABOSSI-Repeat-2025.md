Write-up: Challenge "REPEAT"

Challenge Description
```
Each CTF has its own XOR challenge

Flag: RETRO{value}
```

Given Python Code:
```
import os
key = os.urandom(6)

def encrypt(data):
    xored = b''
    for i in range(len(data)):
        xored += bytes([data[i] ^ key[i % len(key)]])
    return xored

def decrypt(data):
    return encrypt(data)
````

Encrypted Flag (flag_enc.txt):
````
a42bafb2f37db7028cd4c575a93a93d3e355c2039ebfef72c63c829d
````

Understanding the Code

From the given script, we can deduce several key points:

A random 6-byte key is generated: key = os.urandom(6).

XOR encryption is applied, where each byte of the plaintext is XORed with a repeating sequence of key bytes.

The decryption function is identical to the encryption function due to the involution property of XOR.

The main challenge is that the key is not provided, so we must retrieve it to decrypt the message.

Attack Strategy

Since XOR encryption is vulnerable to a known-plaintext attack, we can leverage our knowledge that the decrypted message starts with RETRO{ (a common flag format in CTF challenges). This gives us 6 known bytes, exactly the length of the key.

Using this information, we can recover the key:

```
known_text = b"RETRO{"
key_bytes = []

for i in range(len(known_text)):
    key_byte = encrypted_bytes[i] ^ known_text[i]
    key_bytes.append(key_byte)

discovered_key = bytes(key_bytes)
print("Discovered key:", discovered_key.hex())
```

Recovered Key:

f66efbe0bc06

Decryption Process

Now that we have the key, we can use it to decrypt the entire message:
```
import binascii

# Encrypted flag (hex format)
encrypted_hex = "a42bafb2f37db7028cd4c575a93a93d3e355c2039ebfef72c63c829d"
encrypted = binascii.unhexlify(encrypted_hex)

# Retrieved key
key = binascii.unhexlify("f66efbe0bc06")

# XOR decryption
decrypted = b''
for i in range(len(encrypted)):
    decrypted += bytes([encrypted[i] ^ key[i % len(key)]])

print(decrypted.decode('ascii'))
```
Final Decrypted Flag:

** RETRO{Alw4ys_Th3_S4me_St0Ry} **

