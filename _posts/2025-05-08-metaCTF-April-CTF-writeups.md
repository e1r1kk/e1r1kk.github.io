---
title: "MetaCTF April 2025 CTF Writeup"
categories:
  - Writeups
tags:
  - CTF
  - Webapp
classes: wide
---

# Overview

> On April 24th, [**MetaCTF**](https://metactf.com) hosted a Flash CTF competition, providing various challenges to see who could solve them the fastest. Although I didn't compete on the day of the competition, the challenges were still available after the competition ended, so I decided to take a stab at them to see what I could solve!

> In this post, I will be showing my thought process and approach to solving these challenges. Out of the 5 challenges provided, I was able to solve 2 of them. However, I will updating this post if I solve more of them in the future. 

# Challenges

## 1. Caffeine Conundrum

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