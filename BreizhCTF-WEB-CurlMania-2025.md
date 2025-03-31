# Breizh CTF Web Challenge: Curl Mania

## Overview

In this **web challenge**, we need to send a request with a **modified header** using **Burp Suite**.

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image10.png" alt="curlmania">
</div>

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

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image11.png" alt="curlmania">
</div>


