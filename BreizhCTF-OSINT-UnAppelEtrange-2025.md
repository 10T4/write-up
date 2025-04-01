# Breizh CTF OSINT Challenge: Un appel étrange

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

