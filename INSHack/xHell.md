## xHell
### Category : Reversing

We are given a xlsx file with tonnes of formulae. My initial approach was to backtrack the code, so that inputs are correct.

The one particularly important Formula was 
```
=IF(AND(E82=1,B1>1,B1<=256,C1>1,C1<=256,D1>1,D1<=256,E1>1,E1<=256,B1-C1=46,E1-D1=119),"Congrats! Here is yout flag: INSA{"&TEXT(B1,"0")&"-"&TEXT(C1,"0")&"-"&TEXT(D1,"0")&"-"&TEXT(E1,"0")&"}","Wrong input")
```
It looks at B1, C1, D1, E1 which are supposedly the inputs and E82, which is suppossed to be 1.

Now, backtracking E82 seemed long, and B1, C1, D1 and E1 have relatively small ranges, so I resorted to brute force. 

The max range of C1 varies from 0 -> 211 and value of D1 varies from 0 -> 138. The value of E1 is dependent on D1 and value of B1 is dependant on C1.

I found this awesome python library  ``` xlwings ``` which uses Microsoft Excel as a backend engine to evaluate Formulae.


The Following script gives us the Flag
```
import xlwings as xw

wb = xw.Book("path")
ws = wb.sheets.active
for i in range(138):
    for j in range(211):
        ws['D1'].value = i
        ws['C1'].value = j
        ws['E1'].value = i+119
        ws['B1'].value = j+46
        val = ws['G2'].value
        if 'flag' in val:
            print(val)
```

We get the following flags in somewhat 5 mins

```
Congrats! Here is yout flag: INSA{75-29-13-132}
Congrats! Here is yout flag: INSA{203-157-13-132}
```

We choose the second one according to the Problem Statement