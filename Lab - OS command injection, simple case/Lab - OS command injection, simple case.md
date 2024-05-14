# Objective
This lab contains an OS command injection vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

To solve the lab, execute the `whoami` command to determine the name of the current user.
# Lab link
[Lab: OS command injection, simple case | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/os-command-injection/lab-simple)
# Write up
By somehow, the application cannot check for some information directly, and must call out for external command (for example: `python checkStock.py` instead of calling backend server as normal web server). By this way, we can inject some commands into the request. The way we abuse this vulnerability make the web assets turned into a `webshell`. (From my `red team experience`).
Simply pipe the command with another command in the request to make a command injection:
After some reconnaissance, I know that when press check stock, there is a POST request will be send to server:
![[Pasted image 20240514235743.png]]
Pipe a `whoami` command to the request body:
![[Pasted image 20240515000646.png]]
The username of the server is `peter-TsUa8p`
We can also retrieve other important things on the server such as passwd file too:
![[Pasted image 20240515000844.png]]
Done the lab![[Pasted image 20240515000744.png]]