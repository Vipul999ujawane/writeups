## Molecule Shirts
### Category : Forensics

So, I participared in the RedPWN CTF over the last few hours. Pretty cool CTF, and this challenge is especially VERY different. So, we are given a PNG image, which on checking with `file` is a BMP. Change the extension to .bmp, and we get this!

![Image](picture.bmp)

So, after a lot of while thinking this is a Steganography image, I decided to contact my bio-tech friends. What if this image a clue to something?

So there are two compounds in this image, and both are [peptides](https://en.wikipedia.org/wiki/Peptide). The thing about peptides is that, they have a One-Letter Symbol based Nomenclature!. [Link](https://www.tocris.com/resources/peptide-nomenclature-guide)

So, upon figuring out each and every peptide, we get the name `DR ARMSTRONG`

I thought this could be a password to `steghide` or something. But apparently, this itself was the flag. `flag{DRARMSTRONG}`