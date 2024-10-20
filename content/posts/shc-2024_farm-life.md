+++
title = "Shc 2024 - farm Life"
date = "2024-05-06T16:54:58+02:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["SHC-2024", "crypto"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

Nice challenge, had a pretty hard time with it as i didn’t see a tiny detail.

We’re given the source python code of what’s on the server. Here’s what it looks like, commented :

```Python
#!/usr/bin/env python3
import secrets

FLAG = "FAKE_FLAG"

# the encrypt function takes two parameters, sends back the xor of them two. 
def encrypt(key, plaintext):
    return ''.join(str(int(a) ^ int(b)) for a, b in zip(key, plaintext))


def main():
    # keygen
    key = format(secrets.randbits(365), 'b')
    print("Welcome to the CryptoFarm!")
    while True:
        command = input('Would you like to encrypt a message yourself [1], get the flag [2], or exit [3] \n>').strip()
        try:
            if command == "1":
                data = input('Enter the binary string you want to encrypt \n>')
								# Will allow us to know the key if we feed it a 365 bits long string of 1s. 
                print("Ciphertext = ", encrypt(key, data))
								# THIS !!!! THE KEY VARIABLES IS UNCHANGED AS LONG AS WE DON'T DO COMMAND 1 
                key = format(secrets.randbits(365), 'b')
            elif command == "2":
								# Encrypts the flag and sends it back to us
                print("Flag = ", encrypt(key, format(int.from_bytes(FLAG.encode(), 'big'), 'b')))
            elif command == "3":
                print("Exiting...")
                break
            else:
                print("Please enter a valid input")
        except Exception:
            print("Something went wrong.")

if __name__ == "__main__":
    main()
```

First thing to do : get the encrypted flag. If we encrypt a message first, the key will be regenerated, as commented in the code.

So, we select command 2 and get our encrypted binary flag, that we'll use as a variable for our next python script. 

Now, we store it somewhere for later, and create a 365-bits long string filled with 1s (print('1'*365) in python). It will allow us to find back the key, as the cipher produced is nothing more than a xor operation of our input with that key, right ?

We get a binary output again, let's keep it close, we'll need it soon. 

Now that we’ve got all our ingredients, let’s start cooking ! We can take back some stuff from the python script that we’ll reuse as it is for our final script, like the encrypt function that also serves for decryption as it’s just a xor. Here’s how that python script looks like :

```Python
encrypted_flag = '011110001110000110110110101111011000010001110010011100101001001000000010111001011110010001100010011111100111000111110111100100011111010110110101111010010100111100110010111010001001110111010011001111011000111000010100010010011011101111100101000111101001100000101010000010011101011110100010101000110100110'

ciphertext = '01100001110011101000111100100110000110111110100111100101100110110110001111000010110100110010001100101101010001001101010010110000110101101000100010101000001000000000111111011111110111001010111001111100111001110110000100000100111110101001000000101011101101010001111100111100111110101000001110001100010010010101011101100001010010111001111111111111111100101011110101000'
input_value = '11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111'

def encrypt(key, plaintext):
    return ''.join(str(int(a) ^ int(b)) for a, b in zip(key, plaintext))

key = encrypt(ciphertext, input_value)

flag_binary = encrypt(key, encrypted_flag)
print('This is the binary-encoded flag : ',flag_binary)

def binary_to_text(binary_value):
    decimal_value = int(binary_value, 2)
    text = decimal_value.to_bytes((decimal_value.bit_length() + 7) // 8, 'big').decode()
    return text

FLAGOSSSS = binary_to_text(flag_binary)
print('And this is the flag, at least I hope so : ',FLAGOSSSS)
```

`shc2024{Old_Venona_Had_A_KEY_Eeieeioh}`
