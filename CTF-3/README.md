### Micro-CMS v2

I'm going to try to handle this CTF like a real bug huinting report, using professional layout as a report that could be useful to show a real world series of bug that I found. 

## 1: Initial Assessment

The reasrche we are running is centered on "Micro-CMS" a page that is used to handle data and web pages, here we have a screenshot of the homepage:

![Pasted image 20240618145137](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/b4c71f79-a807-471f-bed4-7b6d0a289fb8)


We are presented with 3 different links to 3 different pages:

- Micro-CMS Changelog: This page lists the changes of this version from the previous we analyzed, saying that many changes were implemented to harden security. I notice that even this page can be edited if we enter a valid username and password. This page is indexed as 1. ![Pasted image 20240618145529](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/59f55759-2b4b-4c26-9bf5-4febb3621166)

- Markdown Test: This page is used to test some markdown scripts such as a button and some links ewmbedded in the page. The button does nothing. The page is indexed as 2. ![Pasted image 20240618145715](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/ff8e85f9-a593-46dd-8833-5b89e3d164d1)

- Create a new page: If we want to create a new page we are first presented with a login page. This will be our forst wall to test. ![Pasted image 20240618145836](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/3bd5e559-fca2-4ef4-ae62-c7c005a108cb)


We will be using Burp-suite as a tool to handle the analysis. 

## 2: Forcing the login

Here we try to force the login. As a first step we try to see how the form responds when a wrong user or password is inserted so we try "asd-asd" and we see that it gives "invalid username" as a response, showing on the page. 

First we tried to bruteforce the admin login trying to see if we could make it say "valid user wrong password", we stested using a limited payload of usernames and the response was good. 

By running an iterative test on page indexes we found that index 3 is probably something protected:

![Pasted image 20240618152303](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/18f24253-baff-4876-b90b-2c1a39d96d4c)


It could be the creation page we are trying to access. 

By running another bruteforce attack using common auth-breaker we encountered a valid SQLi value that can fake an admin authentication: 

![Pasted image 20240618152819](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/3a604c40-9ef9-44ed-9d6c-d9685dea6ee9)


As we can see here it is possible to receive "invalid password" instead that "unknown user" meaning that this username in particular is flagged as a valid one. We will provide the complete outcome of the attack that listed. We were able to collect a total of 9 different usernames that tricked the system like this: 

![Pasted image 20240618153230](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/e513b611-f980-44fd-bc3f-af37d1a4e0af)


At this point this will be flagged as first vulnerability with high impact we were able to find, because as we will show, thsi could lead to more severe attacks. We will provide the full results as an attchment for further readings. 

So now we manually test the username " '-' ", that from now on will be referred as "attacker-username". 

Testing with the attacker username leads to no further discoveries such the password reamins hidden even if trying to bruteforce it one char at a time. 

So, after seeing that the system is vulnerable to SQL injection attacks we try to force the login page using the following value as a payload:

``' UNION SELECT 'pass' AS password WHERE '1' = '1``

also inserting "pass" as password. This is called SQLi UNION SELECT ATTACK and it is one of the most common attacks used to force authentication. It basically forces the database into selecting a password for a user and uses it into the pass value. 

After successfully entering the private session we are met with the following page:

![Pasted image 20240618162745](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/8b1b4000-54d0-492b-85c9-d249d31f267d)


This is useful because we can clearly see a session COOKIE being used to carry the session. We could use it later. For the session of this test we had eyJhZG1pbiI6dHJ1ZX0.ZnGZLA.ypVFf500vKtU6cLvJKWGWu8-p4Y. 

At this point we were able to access page/3 we saw earlier:

![Pasted image 20240618162940](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/0871ed42-aa77-4df1-aec3-3b8a7feaaa50)


Meaning that we were able to break into the system and eventually exfiltrate important informations (6ce122ad059957cdd48fb93d32643be8bbf2fa251f54b43238ace901b7e07e2d). 

## 3: Delivering some damage in the system

Now that we can access to the private zone we can modify pages. Notice that we are now able to send general POST request authenticated as an user that we are not. Here we see an empty POST request showing response from inside the system: 

![Pasted image 20240618165120](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/d512b89f-2309-4b6c-87d9-1bad36139f7b)


(4c5b6a02e2b89574502ba3b7f9b4e7f1ff3d070c57975f364fe7fab5b4156858)

All of this is done by simply not having any username or password, from inside we could try to catch the clear version of those informations. 

## 4: Manual blind SQLi

Here is the manual SQLi for blind that is possible to apply to know clear versions of username and password. We used a method called blind SQL injection where we interpret the "invalid password" as a sign of "valid statement" in the username. So if we perform a comparison with various chars in the username tab we can know a lot more about the username details by simply going blind. 

For example we use `1' OR (select count(username) from admins)=1#` to see if the DB has exactly 1 entry. We can run this inside the intruder and loop a series of numbers from 1 to 100 for example. 

We see that only the attack with the right number gives "invalid password":

![Pasted image 20240618180939](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/1889baa2-e657-453a-8753-2dc1a0c56618)


We do the same using the following `1' OR length(substr((select username from admins limit 0,1),1))=6#` to know the length of the username. 

As we can see we have that the username is exactly 9 chars long: 

![Pasted image 20240618181146](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/b50d8be5-ff2b-4be6-9d7d-6110d5466bc3)


We apply the same concept with the following `1' OR ascii(substr((select username from admins limit 0,1),1,1))=109#` trying to match the first character of the username with 109 that ASCHII for a letter. 

Results we get are:

- 103 - g
- 114 - r
- 97 - a
- 110 - n
- 118 - v
- 105 - i
- 108 - l
- 108 - l
- 101 - e

Username: "granville". Using the same method we discover that the password is 6 chars long. 

- 98 - b
- 101 - e
- 110 - n
- 105 - i
- 116 - t
- 111 - o

The password is "benito". 

As a proof I log in using these credentials successfully obtaining the same result: ![Pasted image 20240618183128](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/5a12226d-7b4a-4d68-82f7-adc580073241)


## Conclusions

As a conclusion we showed how even in this new version the system remain vulnerable to attacks based on SQL injection, due to input sanitization problems in the login page that has been added to the system. 

We were able to both bruteforce username and password using blind SQLi and to bypass authentication using UNITE SQLi techniques. 

We also want to signal that both username and password were extremely weak that we could have them cracked with simple wordlists and some bruteforce repeater. 

We advise the developers to adopt serious and stronger defense mechanisms to prevent unsanitized input inside input fields, other than a stronger eduicational campaign in password behaviour for your users. 

