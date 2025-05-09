---
title: "MetaCTF April 2025 CTF Writeup"
categories:
  - MetaCTF Writeups
tags:
  - CTF
  - Webapp
  - Encryption
classes: wide
---

# Overview

On April 24th, [**MetaCTF**](https://metactf.com) hosted a Flash CTF competition, providing various challenges to see who could solve them the fastest. Although I didn't compete on the day of the competition, the challenges were still available after the competition ended, so I decided to take a stab at them to see what I could solve!

In this post, I will be showing my thought process and approach to solving these challenges. Out of the 5 challenges provided, I was able to solve 2 of them. However, I will updating this post if I solve more of them in the future. 

# Challenges

## 1. Caffeine Conundrum
### Brief

We are provided with the following message:

> The barista left you a note in the java shop, but it seems to be encrypted, can you read it?

Based on this briefing, our goal seems to be that we must decrypt a message to find our flag.


### Info Gathering 
In this challenge, we're first provided with the following artifact:

```java
public class chal {
    public static void main(String[] args) {
        String encodedFlag = "ZrgnPGS{pncchppvab_jvgu_n_ebgngvba_bs_pernz_cyrnfr}";
        
        if (args.length == 0) {
            System.out.println("Usage: java caffeine_conundrum.java <your_guess>");
            System.out.println("Hint: The barista left a flag, but it seems to be encoded...");
            return;
        }

        String userGuess = args[0];
        String encodedGuess = encodeFlag(userGuess);
        
        if (encodedGuess.equals(encodedFlag)) {
            System.out.println("Correct! Time for a coffee break!");
        } else {
            System.out.println("Not quite it! Keep trying...");
        }
    }

    // The barista's encoding method
    private static String encodeFlag(String input) {
        StringBuilder result = new StringBuilder();
        for (char c : input.toCharArray()) {
            if (Character.isLetter(c)) {
                // ROT13 transformation
                char base = Character.isUpperCase(c) ? 'A' : 'a';
                result.append((char) (base + (c - base + 13) % 26));
            } else {
                result.append(c);
            }
        }
        return result.toString();
    }
}
```

After looking through the code, there are two main pieces of code that we're concerned with:

1. The encoded flag: ```ZrgnPGS{pncchppvab_jvgu_n_ebgngvba_bs_pernz_cyrnfr} ```
2. The encoding method used:
```java
private static String encodeFlag(String input) {
        StringBuilder result = new StringBuilder();
        for (char c : input.toCharArray()) {
            if (Character.isLetter(c)) {
                // ROT13 transformation
                char base = Character.isUpperCase(c) ? 'A' : 'a';
                result.append((char) (base + (c - base + 13) % 26));
            } else {
                result.append(c);
            }
        }
        return result.toString();
    }
```

Within the **encodeFlag()** function, there's a comment provided which indicates the encryption method used - ROT13. Knowing this, we can easily decrypt the encrypted flag to its original form.

### Solution

Using the information that we've found, I used [CyberChef](https://gchq.github.io/CyberChef/) to decrypt the flag using the ROT13 recipe, providing me with the final flag: **MetaCTF{cappuccino_with_a_rotation_of_cream_please}**

## 2. MetaShop
