# Objective:
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.
![[Pasted image 20240512183306.png]]
# Write up:
![[Pasted image 20240512183351.png]]
One thing that we should notice in the lab description is that it concentrates on some tracking cookies, instead of normal content:
> The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

Using Burp Suite to catch the request to analyze:
![[Pasted image 20240512184914.png]]
There are two cookies, Tracking Id and session.
What if we insert some malicious content into the request?
Just insert a quote character into the tracking id cookie, and an internal server error occurs:
![[Pasted image 20240512184959.png]]
But when using the double quote, the error seems gone:
![[Pasted image 20240512185043.png]]
This could indicate that the server is queries for tracking id in database to showing the web page back to us. And the first try with only one quote cause an syntax error.
![[Pasted image 20240512185239.png]]
In this case, the SQL query is something like:
```sql
SELECT * from SESSIONS_TRACKING where id = '{$TrackingId}';
```
Where `{$TrackingId}` is some generated cookie.
That's why one quotation mark will raise an error but two is not. Try to concatenate another query into the request cookie:
![[Pasted image 20240512185954.png]]
The request is like:
```sql
SELECT * FROM SESSIONS_TRACKING WHERE id = '{$TrackingId}' || (SELECT 1) || ''
```
But it still gave me an error. Hmm. Maybe the DBMS is Oracle which require a table in every query. Try select from dual:
![[Pasted image 20240512190856.png]]
We're right. 99% the web server is using Oracle DB.
From the hint of the labs, we should target the table users with username = 'administrator' and some passwords. A successfully attack should grant us whether 200 or 500.
The first thing I want to know it how long is the password. An administrator password seem not a very short password (as told, 8 is should be the min length). We should conduct some condition on this case, such as
```sql
IF length(password) > 8 THEN
	return error;
ELSE 
	return true;
ENDIF
```
![[Pasted image 20240512193849.png]]
8 seem a valid answer. But how long should it be?
Try to get the upper limit of the password:
![[Pasted image 20240512194003.png]]
24 is seem an upper limit. But how long is it exactly? After some trying with binary search, I found that its length is 20, when 20 gave 200 but 21 gave 500 code.
![[Pasted image 20240512194142.png]]
![[Pasted image 20240512194156.png]]
As blind approaching, we should find each location character to craft the complete password. Make change on the queries like
```python
if password.substr(position) == 'some-char':
	return error
else:
	return true
```
In this case, Intruder is a better option when it automate our attack:
![[Pasted image 20240512195704.png]]
Each successful position with gave us 500. Using `Cluster Bomb` for iterating both position (index and character) will gave us the full password. The attack should take a while. Then we can get the password to login as admin.
# Lab link
[Lab: Blind SQL injection with conditional errors | Web Security Academy (portswigger.net)](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)
