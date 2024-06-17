Here I'm tasked with a simple CTF challenge. I fire the link and I met thw following: 

![Pasted image 20240617165741 1](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/e6c6b9bf-815a-46fa-bbe2-948a4255eb02)

By exploring the tree viwe of BURP I spot something: 

![Pasted image 20240617165813](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/c01f0c2e-fb23-437d-bf21-a04657cfd2dc)


Basically there is a resource that I could access under the same branch. I then decide to turn on the intercept tool and to change the requested page to "backgroundimage.png":

![Pasted image 20240617165946](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/527262c3-8fb6-43b1-9b60-b071bcf89828)


Capturing the flag:

187ef06c01278df6b6d509737015c976445df6993b16a3d26d193053413ea149

![Pasted image 20240617165540](https://github.com/RBraga-droid/hacker101-CTFs/assets/62329743/b4f0ff9d-559c-49d2-856e-0b0c0de89bab)
