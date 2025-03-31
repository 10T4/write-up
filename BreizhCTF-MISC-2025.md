# BREIZHCTF Misc Challenge: Burner Phone
## Objective
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image4.png" alt="breizhctf">
</div>

The goal of this challenge is to track a target who frequently changes their phone number to avoid being traced.

## Initial Investigation

This **MISC** challenge starts with access to an FBI investigation tool for phone numbers. We are given the last **three phone numbers** used by the target. Using the **FBI tool's** web interface, we can retrieve various details about a phone number, including:
- The **Owner**
- The **operator**
- The **last known location**
- The Range numbers phone
- The **phone's password** (which is of key interest to us)

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image6.png" alt="breizhctf">
</div>

Our first attempt is to create a script that lists all phone numbers from each operator (**BreizhCTF, Blue, C6**), but this yields no useful information. We then try filtering based on location, but the resulting list is too long to be practical.

### Recognizing a Pattern

We notice that for the last **three phone numbers** used by the target, the **password is always "Ashley"**. This suggests that the **new phone number** will likely have the **same password**.

## Extracting the Phone Number Range

Using the FBI tool, we retrieve the **entire phone number range** for the target’s operator, which contains **693 numbers**.

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image6.png" alt="breizhctf">
</div>

### Automating the Search

We write a script to identify which phone numbers use "Ashley" as their password. The script iterates through the list of phone numbers stored in a JSON file, queries the password for each, and prints any matches.

```python
import requests
import json
import time

def check_phone_crack(phone):
    """
    Sends a request to the crack-password URL with the specified phone number and returns the response.
    """
    url = f"http://burner-phone-15.chall.ctf.bzh/app/burner/crack-password/{phone.strip()}"
    try:
        response = requests.get(url)
        return phone, response.status_code, response.text
    except Exception as e:
        return phone, None, f"Error: {str(e)}"

def main():
    """
    Reads the JSON file, extracts phone numbers, queries each number using the crack-password endpoint,
    and prints interesting results.
    """
    try:
        # Read the JSON file
        with open("phone.json", "r") as file:
            content = file.read()
            
            # Ensure proper JSON formatting
            if not content.strip().startswith('{'):
                content = '{' + content
            if not content.strip().endswith('}'):
                content = content + '}'
                
            data = json.loads(content)
            phones = data.get("phone_numbers", [])
    except FileNotFoundError:
        print("File phone.json not found.")
        return
    except json.JSONDecodeError as e:
        print(f"JSON decoding error: {str(e)}")
        return
    except Exception as e:
        print(f"File reading error: {str(e)}")
        return
    
    print(f"Processing {len(phones)} phone numbers for password cracking...")
    print("="*70)
    
    phone_count = len(phones)
    interesting_count = 0
    
    for i, phone in enumerate(phones):
        phone_num, status, response = check_phone_crack(phone)
        
        # Show progress
        if (i+1) % 10 == 0:
            print(f"Progress: {i+1}/{phone_count} numbers processed", end="\r")
        
        # Ignore responses with status 400
        if status == 400:
            continue
        
        # Display full responses for status 200
        if status == 200:
            if response and response.strip() and response != "Not Found":
                interesting_count += 1
                print(f"\n--- Phone {i+1}/{phone_count}: {phone_num} ---")
                print(f"Status: {status}")
                print(f"Response: {response}")
                
                # Check for flag patterns in the response
                if any(pattern in response for pattern in ["FLAG{", "CTF{", "flag{", "ctf{", "BZHCTF{", "bzhctf{"]):
                    print("!!! POTENTIAL FLAG FOUND !!!")
        elif status != 404:  # Ignore 404 errors as they likely mean "Not Found"
            print(f"\n--- Phone {i+1}/{phone_count}: {phone_num} ---")
            print(f"Status: {status}")
            print(f"Response: {response}")
        
        # Pause to avoid overloading the server
        time.sleep(0.1)
    
    print("\n" + "="*70)
    print(f"Processing complete: {interesting_count} interesting responses found out of {phone_count} numbers tested.")

if __name__ == "__main__":
    main()
```
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image7.png" alt="breizhctf">
</div>

## Conclusion

Running this script returns the **three initial phone numbers** and a **fourth number**, confirming that we have found the target’s **new phone number and password**. 

 **FLAG: BZHCTF{0733891373}** 

