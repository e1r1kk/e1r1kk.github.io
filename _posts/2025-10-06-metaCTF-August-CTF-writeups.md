---
title: MetaCTF August 2025 Writeups
categories:
  - MetaCTF Writeups
tags:
  - CTF
  - Webapp
  - Injection
classes: wide
---
## Table of Contents
 - [Overview](#overview)
 - [Challenges](#challenges)
    - [1. Baby Something (VERY Simple)](#1-baby-something)
    - [2. Super Quick Logic Invitational](#2-super-quick-logic-invitational)
 - [Conclusion](#conclusion)

## Overview

Hey there! In the previous post, I had mentioned that I was going to try and post a writeup once a month until August. As you can see, that didn't really pane out... but we're back! I'm behind quite a few months in regards to the monthly MetaCTF events, but I recently went back and attempted the challenges posted in August and was able to solve two: one forensic challenge that was quite simple, the other being a very interesting injection exploit! 

In this post, I will be covering my thought process as to how I solved these challenges.


## Challenges

### 1. Baby Something

#### Brief

We are provided the following message: 

> How does the song go again? Baby *something* do do do do do do...

Alongside this, we are provided a .pcap file named *babysomething.pcap,* thus based on the lyrics heavily suggesting that *something* is "shark," we most likely have to analyze the .pcap using Wireshark.

### Info Gathering

Upon opening the .pcap file, we're provided with the following network traffic: 
![network traffic](\assets\images\August2025MetaCTF\NetworkTraffic.png)

If we look through the traffic, we see that the sixth frame has text/html info which could be of interest to us. 

### Solution

When we open up the sixth frame, the flag is found within the text data: **MetaCTF{w1r3sh4rk_do0_d0o_d00_d00_doo_do0}**

![flag1](\assets\images\August2025MetaCTF\flag1.png)


This was a very simple challenge that was most likely intended to introduce new CTF challengers to Wireshark and promote them to learn how it works. 
### 2. Super Quick Logic Invitational

### Brief

We're provided the following message:

>I'll take "Rather Vexing" for 500 Alex.
>
>This new trivia game is pretty fun, but one of the challenges is impossible! "What is the flag for this CTF challenge?", how would I know?! Maybe you'll fare better?

In addition, we're provided a link to the trivia game which looks like this: 

![website hub](\assets\images\August2025MetaCTF\websitehub)

Based on this briefing, we most likely have to manipulate the site in some way to retrieve our flag.

### Info Gathering

After clicking "Start Game," we immediately jump into the questions. We're given 30 seconds to solve each question and if we fail, we're given a summary of how we did.

Game format (endpoint \game):

![game](\assets\images\August2025MetaCTF\game.png)

Summary format (endpoint \game_end):

![gameSummary](\assets\images\August2025MetaCTF\gameSummary.png)

My immediate question when presented with a single text box for submitting answers was, "I wonder if I can inject some SQL?" So I inputted **' OR 1=1;--** just to test the waters and submitted it as my answers.. and was rewarded the points? Interesting...

![firstInjection]"\assets\images\August2025MetaCTF\firstInjection.png"

After finding out that SQL Injection was present, I wanted to know if any verbose information would be present if I were to do an invalid query. Thus, I inputted a broken SQL query and got the following output: 

```SQL
**Error:** unrecognized token: "'"  
**Query:** SELECT * FROM problems WHERE id = 135 AND answer = '' OR 1=1'
```
Based on this output, we begin to understand more about the structure of the database. So far, we know that within the table **problems**, there are columns **id** and **answer**.

We can find more information about the table by capturing the traffic of the original **' OR 1=1;--** submission and looking at the response from the website:

```JSON
{"correct":true,
"next_problem":{"difficulty":4,"id":86,
"question":"If two players play Rock-Paper-Scissors infinitely, what is the expected value of the game for Player 1?"},
"score":40
}
```

Based on the response, we uncover two more columns: **difficulty** and **question.** To see if these are all the columns, we can do a simple **UNION SELECT NULL** query to check the number of columns within the **problems** table. Since we know 4 of the columns already, we can input " **' UNION SELECT NULL,NULL,NULL,NULL from problems;--**  to see if we get rewarded points. Once inputted, we got the following response:

![unionSelect](\assets\images\August2025MetaCTF\unionSelect.png)

This confirms that there are only 4 columns within this table. Now we can start making a query that will retrieve our final flag.

At first, I tried doing " **' UNION SELECT answer, id, question, difficulty from problems where question = "What is the flag for this CTF challenge?"**, using the question provided from the briefing to try and limit down the output to just our flag. At first, I thought this wasn't doing anything and I was starting to wonder if I was going down the wrong rabbit hole. But after the game ended, I noticed that where it records your answer, it was storing the question instead: 

![weirdAnswerOutput](\assets\images\August2025MetaCTF\weirdAnswerOutput.png)

Based on this, I was able to conclude that it was most likely outputting whatever was found in the third section of my SELECT statement as the answer. Thus, I was ready to make my finalized query.

### Solution

After clicking "Start Game," as my answer for the question, I submitted the following answer: 

```SQL
 ' UNION SELECT NULL, NULL, answer, NULL from problems where question = "What is the flag for this CTF challenge?"
 
``` 

Utilizing the question column within a WHERE statement helped limit the scope of possible answers to just the flag in question. In addition, having the answer column within the third section of my SELECT statement ensured that the flag would be outputted as my answer, regardless of what question was asked.

Thus, after submitting this query and ending the game, I got the following output:

![flag2]((\assets\images\August2025MetaCTF\flag2.png)

With this output, we get our flag: **MetaCTF{wh4t_1s_7h3_fl4g_f0r_7hi5_ch5l1eng3}**
## Conclusion

Thank you for taking the time to read my process work! Solving these challenges are always a fun (though sometimes frustrating) time and it really helps me develop my skills using industry standard tools. For anyone who may have struggled with these two challenges, I hope these writeups helped you understand where you may have gone wrong and how to approach problems like these in the future!

I've gotten quite busy with school now that my third year at Sheridan has started, so unlike last post, I won't be making any promises as to when my next post will be! However, as a CTF developer for [ISSessions](https://issessions.ca/) annual CTF, stay tuned for a post around the end of January or the beginning of February which will provide the challenges that I've created for the event, alongside their corresponding writeups!