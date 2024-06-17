Here I see a series of pages that creates and edit pages. 
![Pasted image 20240617175243](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/fafe03f9-fa69-4e36-9d93-780bc586eb2c)


I see that it is possible to create pages and I notice that after a while each page is indexed skipping numbers from 3 to 7. 
![Pasted image 20240617175159](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/ef83ca7a-1827-46c4-8c81-7351d317f605)


By selecting a simple GET request I click on "send to repeater" and repeat the same query chaging the index of the page from 3 to 7. I get error 404 on every page except that on id 5, that is restricted. 

I then remember that the "edit" function is also indexed using numbers so I basically repeat a normal edit query by chaing the ID into 5:

![Pasted image 20240617175444](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/44003d20-e693-4265-aaa7-a15eccd84d00)

![Pasted image 20240617175504](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/5b4ead73-da41-448a-b0c0-facf502caffb)

And I'm rewarded with the first flag: 

294b16c8a13d4742618a24a86ecb04078cb6a52e7e6791d34d8b51575edee89f

What I can learn from this is that sometimes it is possible to apply lateral movement on pages. Page 5 was protected by normal access but not by the edit function, that I used to access the resource. 

Now it's time for basic XSS tentatives so I try to inject the following:

![Pasted image 20240617180151](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/468a467c-1e4b-4c9b-beb5-027a6e941539)

Unfortunately I notice that the input is sanitized:

![Pasted image 20240617180333](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/68ac54a3-c9e2-43da-8448-ef1da5410d07)


I try more general special characters:

![Pasted image 20240617181014](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/59f8ea54-9be1-47e6-beb7-bee2e9e3f0f0)

Obtaining the flag.

![Pasted image 20240617180553](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/c412bab1-83f9-477c-98fc-0e22c36192ef)


In the page "markdown test" there is a button that does nothing. It is a good spot to test if it is possible to make it perform some actions. I go the edit page and edit the button function:

![Pasted image 20240617181014](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/4e7e8214-b959-4fa0-b3bb-3e0a8817a27e)


And inside the code of the page I'm rewarded with third flag: 03d9c33fc71398eb51fc9f34e495cd9032b52e55c6789eb2944b97817c202bc8

![Pasted image 20240617181240](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/ca483d07-8c67-44f3-a744-b21620ed81e0)


Discovering the possibility of inserting a custom function of a script is important inside a page like this that does not have any sanitization filters on iput on functions of buttons. 

The last important tool that I can use is URL injection where basically I try to see if it is possible to inject some special characters inside the URL of specific page. I try on the edit page:

![Pasted image 20240617181639](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/19919853-40ac-4c24-87cb-62917581be0f)


Here I basically add the " ' " inside the URL of the get request, simulating an attack where an hacker forcefully terminates the command and inject some other. 

I am rewarded with the last flag: 6af056d8ff90bbb7a7340f4eb1ef451730ff417fa8bf426d57fac4c9db6714de

![Pasted image 20240617181753](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/2079518c-7f52-41b3-9e90-8e011d4a2e9b)

