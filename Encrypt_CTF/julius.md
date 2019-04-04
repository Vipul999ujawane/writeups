## Julius,Q2Flc2FyCg==
### Category : Crypto

We are given a ciphertext fYZ7ipGIjFtsXpNLbHdPbXdaam1PS1c5lQ

Now, as the chal name is , first part normal , second part what looks like base64. Let's decode that

We get `Julius,Caesar`. Clearly, the ciphertext is associated with Caeser Cipher

First, we decode the string from base64, (slight intuition because name)

and then applying `brute` force over the number of shifts, 

we get for shift +24, `encryptCTF{3T_7U_BRU73?!}`

A small julius caesar refrence XD
