# Objective
This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response. However, you can use output redirection to capture the output from the command. There is a writable folder at:

`/var/www/images/`

The application serves the images for the product catalog from this location. You can redirect the output from the injected command to a file in this folder, and then use the image loading URL to retrieve the contents of the file.

To solve the lab, execute the `whoami` command and retrieve the output.
# Lab link
[Lab: Blind OS command injection with output redirection | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection)
# Write up
From the hint, we should do something like
```bash
whoami > /var/www/images/whoami.txt
```
Then read the output file.
Firstly, there are a form with multiple fields:
![[Pasted image 20240515002105.png]]
This is the feedback form from the app.
The response seems do not show anything:
![[Pasted image 20240515002419.png]]
Try to use ping, and we got a bit longer response time. Seem like the command injection worked.
Inject the command into email parameter, then send it. 
![[Pasted image 20240515002933.png]]
Then try to retrieve the file from static source:
![[Pasted image 20240515003349.png]]
Got the code run. The lab done.
![[Pasted image 20240515003612.png]]
