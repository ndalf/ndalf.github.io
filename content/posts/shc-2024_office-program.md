+++
title = "Shc 2024 - Office program"
date = "2024-05-06T16:54:58+02:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["SHC-2024", "pwn", "reverse-engineering"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

This was the easiest pwn challenge of the ctf. It didn’t require any overflow or anything.

Here’s the most interesting part of the program :

```C
puts("\nSelect an action:");
puts("0 - Exit (like leaving the offic…");
puts("1 - Print favourite excel column");
puts("2 - Call Rebecca from front desk");
puts("3 - Get secret sauce (only for f…");
printf("Enter your choice: ");
int32_t input; // Lost a lot of time trying to figure out if this was overflowable
__isoc99_scanf("%d", &input);
important_work_or_attend_a_meeting();
if (input == 3)
{
    break;
}
if (input < 0)
{
    puts("\nInput out of range. You confus…");
    input = -(input);
}
input = (input + 5);
if (input < 0)
{
    puts("\nInput out of range. You confus…");
    print_flag();
}
```

The goal is to reach the `print_flag` function. To do so, we have to send the program a value that will be transformed in its negative value. After, 5 will be added to that value, and after this that number has to be less than zero to call the function. At first I thought that sending any negative number less than 5 would make the cut, but it did not, simply because the scanf function expects a `%d`, thus an integer.

Then, I tried to send a value closed to the INT_MIN, in my case `-2147483648`, which worked.
