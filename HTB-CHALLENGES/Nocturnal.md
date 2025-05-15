For this new machine "Nocturnal" we have 2 port open:
- Port 22 SSH
- Port 80 HTTP
On the HTTP port we found a web page that look like an file share, two option is possible register and login:
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/image14.png" alt="htb">
</div>

To continue the investigation, we create an account on the register page "register.php":
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/image13.png" alt="htb">
</div>

After the registration we can sign in on the login page "login.php" and we redirected on the dashboard page "dashboard.php", this page contain a button to upload some files and an another side to view our file uploaded. For testing this app we upload a test file named hello.docx
