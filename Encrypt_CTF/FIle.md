## Journey to the Centre of the File
### Category : Forensics

We are given a gzip compressed file. I generally use the 7z compressor to extract files as it supports a wide range of files.

On unzipping, we get a zip file. I think. this is going to repeat. Let's automate it.

``` 
import os
import subprocess
import shutil
import time
for iter in range(1000):
    path = "/home/vipul/Desktop/EncryptCTF/ziptunnel/files/"
    file = os.listdir()
    extract=""
    for i in file:
        if("flag" in str(i)):
            extract = str(i)
            break
    print("EXTRACT =>",extract)
    cmd = "7z x {}".format(extract)
    print(cmd)
    subprocess.Popen(cmd,shell=True)
    time.sleep(1)
    subprocess.Popen("mv {} {}".format(extract,iter),shell=True)
    time.sleep(1)
    print(os.listdir())

```
On File 68, the code breaks. 

We read the file, we get the FLAG ``` encryptCTF{w422up_b14tch3s} ```