## How to Decompile a APK to .java Files
---
This is a topic that I had to google multiple times during multiple CTF sessiosn, so decided to write a small post regarding it.

APK reversing has been a newer topics in recent CTFs. You can use APKTool, but it gives out smali files as a output (but also Non-java files such as `AndroidManifest.xml`), which is not very comfortable to read. So We'll be using 2 tools:

1) [Dex2Jar](https://github.com/pxb1988/dex2jar)
2) [Procyon](https://bitbucket.org/mstrobel/procyon/src)

--> Dex2Jar converts an APK File to a Jar File with .class files 

--> Procyon Converts .class files to .java files for easy readibility


## Usage
---
-  Step 1
  ---
-> Use the Dex2Jar Script

 ```
 $ ~/Desktop/CTF/dex2jar-2.0/d2j-dex2jar.sh <APK>
```
This returns a jar file.

---

- Step 2
 ---

-> Use procyon jar to generate the file structure with `.java` files

```
$ java -jar decompiler.jar -jar myJar.jar -o out
```

You can change Directory to `Out` to view the code.

NOTE : For non-java files use 

```
$ apktool d <APK>
```

Let the reversing begin.