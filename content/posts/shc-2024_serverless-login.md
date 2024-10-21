+++
title = "SHC 2024 - Serverless Login"
date = "2024-05-10T02:15:19+02:00"
tags = ["web"]
showFullContent = false
readingTime = false
hideComments = false
+++

In this web chall, we have a simple login portal in which we’re asked to enter our username and password. Looking a bit at the code, we see two interesting things : first, this link in an undisplayed div : [http://tinyurl.com/47vu98x9](http://tinyurl.com/47vu98x9) (i’ve been rickrolled, you will too.). Second, at the bottom we have the link to the script that manages the input :

```HTML
<script type="py" src="./main.py" config="./pyscript.json"></script>
```

The main.py file is, as expected, what handles our input. It looks in a database and compares the hashed input to the password. In the pyscript.json file, we have a link pointing to the database, that we can simply use to download it and check the values stored in it.

We have the valid username that isn’t hashed : admin, and a hash : `11a4a60b518bf24989d481468076e5d5982884626aed9faeb35b8576fcd223e1`

Now we just have to convert it back to text and we got a super nice password. 

The portal gives us the expected flag : `shc2024{wh0_N33d5_4_53RV3r_4nYw4Y2?}`
