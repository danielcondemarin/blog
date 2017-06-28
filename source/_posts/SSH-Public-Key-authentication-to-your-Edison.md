---
title: SSH Public Key authentication to your Edison
date: 2017-06-25 09:43:38
tags:
---

I believe the best developers are the laziest ones, those who *do more by doing less*. That includes not having to type in your password every time you want to login to your **Intel Edison**.

If you have worked with ***nix** systems, you might be familiar with SSH Public Key authentication. If you don't know what that is, I strongly recommend having a read of this [article](http://blakesmith.me/2010/02/08/understanding-public-key-private-key-concepts.html), it is by far one of the best explanations I've seen. 

In a nutshell, we will be generating a private and a public key. Then we will copy ONLY the public key to the **Edison**. That way you will be able to login to the Edison automatically as long as you have the private key.

Let's start, first open a terminal, then run the following commands:

1. Generate private / public key pair via [ssh-keygen](https://linux.die.net/man/1/ssh-keygen):

	`ssh-keygen -t rsa`

	I named the key "**edison_key**", but feel free to use your own name. When you're prompted for the **passphrase** just press enter, to avoid creating one. It should all look like this:

	![](/images/ssh-keygen.png)

2. Copy the public key (**edison_key.pub**) to your intel edison:

	`cat /Users/daniel/.ssh/edison_key.pub | ssh root@192.168.0.10 "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"`

	Make sure you replace */Users/daniel/.ssh/edison_key.pub* with your own path. Also, I'm assuming the Edison is accessible via **192.168.0.10** with SSH enabled.

3.  That's it! Let's connect now:

	`ssh root@192.168.0.10 -i /Users/daniel/.ssh/edison_key`

	![](/images/login-via-pub.png)


Tip: If you want to avoid specifying the private key *... -i /Users/daniel/.ssh/edison_key*, just use the default name when generating the key on step *1*, which is id_rsa.
