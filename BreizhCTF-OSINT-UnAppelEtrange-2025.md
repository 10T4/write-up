# Breizh CTF Web Challenge: Modified Header Request

## Overview

In this **web challenge**, we need to send a request with a **modified header** using **Burp Suite**.

## Request Structure

We send the following **GET request**:

```
GET /1337?param1=foo&param2=bar&param3=35 HTTP/2

Host: curlmania.ctf.bzh
Cache-Control: max-age=0
Sec-Ch-Ua: "Not?A_Brand";v="99", "Chromium";v="130"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: J'aime la galette saucisse
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-site
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: Je jure solennellement que mes intentions sont mauvaises mais je ne vais pas taper sur l'infra
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Liberez-Gcc: OUI
Content-Type: application/json
Cookie: jaiplustropdinspi=1
Content-Length: 1337

{"enbretagne": "il fait toujours beau"}
```

## Challenge Complexity

The main difficulty in this challenge lies in managing the **Content-Length** correctly. This can be adjusted by adding spaces within the **JSON data payload**.

## Solution

By ensuring the correct **Content-Length** value and maintaining the required headers, we successfully bypass the challenge and retrieve the flag.

🎯 **FLAG: [Insert Flag Here]** 🎯

---

# Breizh CTF OSINT Challenge: A Strange Call

## Challenge Description

> "I met a man at a party last night. His story was incredible, but it intrigued me and deserves to be public. I had too much to drink, but fragments keep coming back to me: It seems that his story began in a region I had visited at the same time as him: Casamance. He told me about an incident he experienced there at that time. I remember thinking that it happened the day after the Reds qualified against I Giallorossi."

**The flag format:** `BZHCTF{Location_03052018_ACRONYM}`

## Investigation Process

1. **Understanding the location**: 
   - Casamance is a region in the **south of Senegal**

2. **Finding the date**:
   - "I Giallorossi" refers to **AS Roma**, while "The Reds" refers to **Liverpool**.
   - Searching **"Liverpool qualified against AS Roma"**
   - The match took place on **May 2, 2018**, meaning the incident in Casamance happened on **May 3, 2018**.

3. **Identifying the incident**:
   - A search for **"Casamance" "Bridge" "May 3, 2018"** leads to the **Niambalang Bridge attack**.
   - The acronym associated with this event is **MFDC (Movement of Democratic Forces of Casamance)**.

## Flag Retrieval

Using the gathered details, we construct the flag:

 **FLAG: `BZHCTF{Niambalang_03052018_MFDC}`** 

