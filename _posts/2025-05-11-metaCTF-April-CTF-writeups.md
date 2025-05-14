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
## Table of Contents
 - [Overview](#overview)
 - [Challenges](#challenges)
    - [1. Caffeine Conundrum (Simple)](#1-caffeine-conundrum)
    - [2. Metashop (Complex)](#2-metashop)
 - [Conclusion](#conclusion)
 
## Overview
On April 24th, [**MetaCTF**](https://metactf.com) hosted a Flash CTF competition providing various challenges to see who could solve them the fastest. Although I didn't compete on the day of the competition, the challenges were still available after the competition ended, so I decided to take a stab at them to see what I could solve!

In this post, I will be showing my thought process and approach to solving these challenges. Out of the 5 provided, I was able to solve 2 of them, but I will update this post if I solve any more in the future. 




## Challenges

### 1. Caffeine Conundrum
#### Brief

We are provided with the following message:

> The barista left you a note in the java shop, but it seems to be encrypted, can you read it?

Based on this briefing, our goal seems to be that we must decrypt a message to find our flag.


#### Info Gathering 
We're provided with the following artifact:

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

After looking through the artifact, there are two main pieces of code that we're concerned with:

1. The encoded flag: **ZrgnPGS{pncchppvab_jvgu_n_ebgngvba_bs_pernz_cyrnfr}**
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

#### Solution

Using the information that we've found, I used [CyberChef](https://gchq.github.io/CyberChef/) to decrypt the flag using the ROT13 recipe, providing me with the final flag: **MetaCTF{cappuccino_with_a_rotation_of_cream_please}**

---

### 2. MetaShop

#### Brief

We're provided with the following message:

> I've been looking at a shop website one of my friends made. Among the quotes, it contains one very important flag. Can you exploit the shop and retrieve it for me?

Alongside this, we're provided a Docker image alongside a link to its corresponding website:  

![metashop homepage](\assets\images\April2025MetaCTF\MetaShopHomepage.png)

Based on this briefing, our goal seems to involve manipulation of a site to buy our flag.

### Info Gathering

After signing up and logging in, we can load the Products page and see the different products we can buy - one of which being our $1000 dollar flag:  

![product page](\assets\images\April2025MetaCTF\ProductPage.png)

If we go to our profile, we see that we have $100. The question is, how can we gain more money if we can only buy and return products we've bought?  

![base profile](\assets\images\April2025MetaCTF\BaseProfile.png)

The first thing I wanted to do was see what cookies were being sent when loading the Profile page. My goal was to manipulate whatever "balance" variable was being used to store the user's balance, giving myself the exact balance I would need to buy the flag.

Using Burpe Suite, I intercepted the traffic for the profile page and found a **JSON Web Token** (JWT) cookie:  

![base profile](\assets\images\April2025MetaCTF\BaseProfileRequest.png)

For those who don't know what a JWT cookie is (as I didn't upon finding it), it's a form of authentication which utilizes encryption and a signature to store JSON data. Utilizing a [JWT Debugger](https://token.dev/), I took a look at what data was being stored inside.

![jwt debugger](\assets\images\April2025MetaCTF\JWTDebugger.png)

Within the payload, I found two parameters being stored: The account's email address and the accounts **balance**. To see how this JWT is being made and authenticated, I looked inside the **app.rb** file witin the Docker image: 
<p style="font-size:20px; font-style:italic;">JWT Creation</p>

```ruby
post "/login" do
  if users[params[:email]] && users[params[:email]][:password] == params[:password]
    user_info = { email: params[:email], balance: users[params[:email]][:balance] }
    token = JWT.encode(user_info, SECRET_KEY, "HS256")
    response.set_cookie("jwt", value: token, path: "/")
    redirect "/"
  else
    @error = "Invalid email or password"
    erb :login
  end
end
```
  
<p style="font-size:20px; font-style:italic;">JWT Account Authentication</p>  

```ruby
SECRET_KEY = SecureRandom.hex(64)
helpers do
  def logged_in?
    token = request.cookies["jwt"]
    return false unless token

    begin
      decoded_token = JWT.decode(token, SECRET_KEY, true, { algorithm: "HS256" })
      @current_user = decoded_token[0]
      true
    rescue JWT::DecodeError
      false
    end
  end

  def current_user
    @current_user
  end
end
```

This authentication method was used with any GET request done throughout the website. Because of this, manually changing the balance to have $1000 dollars wasn't possible since it caused the signature of the JWT to change, thus making it invalid. However, there were two spots where the JWT wasn't being verified: when you **buy** and **sell** products.

#### Solution

In the **app.rb** file, I found that when something was being bought, it updated your balance and created a new JWT to reflect the new balance:

```ruby
  product_id = params[:product_id].to_i - 1
  product = PRODUCTS[product_id]
  user_email = current_user["email"] 
 if product && users[user_email][:balance] >= product[:price]
    users[user_email][:balance] -= product[:price]
    users[user_email][:quotes].push((product[:Product]))

    user_info = {
      email: user_email,
      balance: users[user_email][:balance]
    }
    new_token = JWT.encode(user_info, SECRET_KEY, "HS256")
    response.set_cookie("jwt", value: new_token, path: "/")
```

I noticed that the code never verified the balance before being updated, thus I could abuse this JWT update process through the use of Burpe Suites Repeater through the following steps:

1. I Captured the traffic of buying a product (in this case, the second one as to optimize profits).  

![Initial Buy Captured](\assets\images\April2025MetaCTF\InitialBuyCaptured.png)

2. **Before forwarding the captured traffic within the proxy,** I sent it to the Repeater and processed that request as many times as I could until I had "Insufficient funds."  

![Repeater Abuse](\assets\images\April2025MetaCTF\RepeaterAbuseResponse.png)

3. I then processed the captured traffic from the proxy itself and turned off the intercept.  
  
Since I froze the traffic with my original JWT that had a balance of $100 dollars, sending it only after I had bought as many products as I could through the Repeater, I confused the app into setting the new JWT as the one with $100 dollars, resulting in no lost of funds!  

![Free Products Who Dis](\assets\images\April2025MetaCTF\FreeProductsWhoDis.png) 

4. After selling all of my "purchased" products, I repeated Step 1-3 until I had enough money to purchase the flag!  
  
Eventually, my profile looked like this:

![Holy Money](\assets\images\April2025MetaCTF\HolyMoney.png)  

Now that I had enough money, I could buy the flag and see it at the bottom of my quotes:

![MetaShop Flag](\assets\images\April2025MetaCTF\MetaShopFlag.png)  

## Conclusion

Thank you for taking the time to read this post! I will be posting my own writeups for MetaCTF's competitions once a month until August. Once September comes, I will be back in full time classes, so posting will slow down for a little while.  
  
If you would like to see the official writeups for these challenges, check them out on [MetaCTF's](https://metactf.com/blog/tag/flash-apr2025/) website.

