## Encrypted Pastebin

As before and probably from now on I will try to handle this CTF as a real world simulation for bug huntin, making it similar to a real security report. 

## 1: Initial assessment

Our initial analysis starts on the homepage of pastebin, a service that holds text in an encrypted format: 

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/59ff171a-718c-4f85-bf8f-a4bcfc50076f)

By making a simple test we notice that a page is generated and indexed using a long string probably an encrypted information. 

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/35b8dde9-ac14-4989-91b2-39059e81a78e)

Here we have one as a proof: 

4H2uaKRL7lBLZUkVaRm3RIjT92oyOvxJBfiha73DzjJPIfjMRHc0wDVES6i9ICbE!9ibqIm17u0tQXW1Jrmcl!sksuq-u8GkkrTUorVkAWKM5jBeqiWJFU0LG8zgdy9XiRmC0CXh6BThi5z9Ev7kKs016g8ViPgoCc4B0GHxiIG9oSNsEcik0VGZSK5!l!TmXhACSOlqwrHvUqrgZmg3WA

This index is unique, meaning that if a user post something on the pastebin, it is virtually impossible for a hacker to find it because it can't be indexe without the encrypted string that refers to it. Infact, if we try to repeat the query for a certain encrypted string we are met with the page we previously created. This means that there could be basically a large amount of pages that we are not seeing due to the fact that are indexed by encrypted indexes. We could try to access or enumerate them. 

What do we know about AES: it is a block cypher, meaning that data is divided into blocks of fixed lenght and eventually padded to meet the length requirements. This data is then encrypted and translated into a fixed length string so probably the indexing string is in fact containing the data we put into it. We can prove this by repeating a get request removing a char from the string:

![image](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/ad8a8a46-733b-4ec5-bfdd-bab2b86c5372)

As we can see we are rewarded with the flag meaning that we were right, the string must be of a fixed length. Flag: b2be5cd3723080f9662f639d1f4eae0deec93ac62b6465b8fd678b34598bf991 .  

Moreover, receiving the message about padding could give us the clue that the message is not hashed and validated by the fact that the error message is very specific to the problem. Non hashed AES algoprithm are vulnerable to padding oracle attacks. We will se if this is the case. 

What we can understand at this point is also that there is a function while decoding that replaces characters: `b64d = lambda x: base64.decodestring(x.replace('~', '=').replace('!', '/').replace('-', '+'))` This is some key information to know before trying to decrypt the IDs.  

## 2: Padding oracle attack

We can try the padding oracle attack wehere we use the fact that the padding of the message made to match the dimension of a block can be used against the encryption mechanism itself. We basicaly know how a padding word is encrypted and we can leverage this. Morover, we have a message that says "Invalid padding" that works as a oracle, telling us if the padding is good or not. To leverage this vulnerability we try to use [PadBuster](https://github.com/AonCyberLabs/PadBuster) an automated tool that can take a link to a page and try to apply padding oracle attack automatically. All we needed was a command like:

`./padBuster.pl https://24ca3ce1f2575b422afcab5d1936cc01.ctf.hacker101.com/?post=hOYVv/NRC7sPBv4KTz14FCiTq/XDWUK2rTnKwILmprn9pUE48Mj/7a359faz2VD+3t4QgEYGPJFszqiS0FhvFa6jzdwbtTL2fe/aQpGLc3OK0amlS4UoyEf/RAiJPW7DiqebWNMy4xwT6ZIsZUX6suQDlILqxTRISDVWxBdpttLVSnSiQJ8qdXW44fu5nd1XGBPeVAJKG0tlRgCsh0ALxQ== hOYVv/NRC7sPBv4KTz14FCiTq/XDWUK2rTnKwILmprn9pUE48Mj/7a359faz2VD+3t4QgEYGPJFszqiS0FhvFa6jzdwbtTL2fe/aQpGLc3OK0amlS4UoyEf/RAiJPW7DiqebWNMy4xwT6ZIsZUX6suQDlILqxTRISDVWxBdpttLVSnSiQJ8qdXW44fu5nd1XGBPeVAJKG0tlRgCsh0ALxQ== 16 -encoding 0`

Buildt by applying sostitution to a post request we catched using burp suite. 

The steps for a padding oracle attack are:

1. See if the target is vulnerable, i.e. as we demonstated before there is a hidden message saying "invalid padding" that can be used as a lead.
2. When vulnerable, the attacker proceeds with tentatives trying to understand the valid padding value trying different cyphertext and seeing the response of the oracle.
3. When the attacker knows the paddin method, it can send the original encrypted message as a plaintext do decrypt, and hopefully obtain the clear text as a result.
4. Having both clear text and cyphertext the attacker can also understand the cypher key, officially breaking the mechanism.

Our idea is that if we can understand the encryption key we can force the decription mechanism to apply malicious actions on the web page that decrypts the message. By skipping the translation form for example we can insert speial charaters such as `'` to perform SQLi, or even some tags and script that would be otherwise sanitized. We could also tweak the indexing system and probably access various protected pages. 

`https://99ab65737b8707f999ca6c871c1d478b.ctf.hacker101.com/?post=g+nb8y8mLZ6zSepuo0SBK81gmjyX7OyYTf5OxHn0NMVji8rlLmSKw3K+2W1n4NpMlGDlQsMl7mRl4oBZrPW1Vkxjm2SPLvdo+Hdl2NuOgFis9d0lx+A7PwhwRcY5VHxkJw5owSoGz4IbbxVope3OA4ijmj7ZmkVotzc+jctoPYznt2BCcvG9o+c4OgPmkc84AR79e4INmp5+tJ4lcsLRyQ== g+nb8y8mLZ6zSepuo0SBK81gmjyX7OyYTf5OxHn0NMVji8rlLmSKw3K+2W1n4NpMlGDlQsMl7mRl4oBZrPW1Vkxjm2SPLvdo+Hdl2NuOgFis9d0lx+A7PwhwRcY5VHxkJw5owSoGz4IbbxVope3OA4ijmj7ZmkVotzc+jctoPYznt2BCcvG9o+c4OgPmkc84AR79e4INmp5+tJ4lcsLRyQ==`

FLAG:

503c3888b834985c
031fce375fb24ecb
