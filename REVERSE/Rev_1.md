# Reverse ingenering
On a un fichier Rev_1

on le passe dans ghidra: 
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/rev1.png" alt="rev1">
</div>

on remarque un base64 qui contient une un xor 105

on le passe dans l'outil en ligne CyberChef:

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/cyberchef.png" alt="cyberchef">
</div>

et on a le flag: SSI{Uncr4ckab13_paSSw0rD}
