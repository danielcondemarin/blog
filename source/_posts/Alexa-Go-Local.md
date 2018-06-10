---
title: Alexa Go Local!
date: 2018-05-26 15:58:44
tags:
- Alexa
- Alexa Golang
- Golang
- Alexa Skill
- AWS
---

![Alexa Go Local](https://res.cloudinary.com/danielcondemarin/image/upload/v1527349668/AlexaGoLocal_xip103.png)

In the last few days, I've been exploring the world of Alexa Development. One of the challenges I've found, is that it wasn't immediately obvious how to run and test your skill locally.

In this post, I'll show you how to spin up a skill on localhost, and test it from the alexa developer portal. Ah! you probably already guessed from the title, but we'll be using Golang today!

First, you need to setup your local skill server, think of it as a Lambda function you'll host on your machine, therefore much easier to test. Luckily there is a great [go pkg](https://github.com/mikeflynn/go-alexa/tree/master/skillserver) out there already for this.

Let's create our project and call it _Greeter_, make sure you replace _github.com/danielcondemarin_ with your own namespace!

```
cd $GOPATH/src/github.com/danielcondemarin/
mkdir greeter && cd "$_"
touch main.go
```

Now _go get_ the skill server pkg:

```
go get github.com/mikeflynn/go-alexa/skillserver
```

In your favourite editor, change _main.go_ to:

{% gist 626c7bde6c5c7c558763e95b2e7b19df %}

Now head onto the [developer portal](https://developer.amazon.com/alexa) and create the skill:

![Create Skill Step 1](https://res.cloudinary.com/danielcondemarin/image/upload/v1527344795/create_skill_1_uop6tn.png)

![Create Skill Step 2](https://res.cloudinary.com/danielcondemarin/image/upload/v1527344796/create_skill_2_avh9y0.png)

After creating the skill, take a step back, copy the Skill ID on the main dashboard and set __AppID__ in __main.go__.

You can now build the go skill server, and run it. By default will be hosted on http://localhost:3000

```
go build *.go
./main
```

Next, set an invocation name, just _greeter_ will do. We will also need an _intent_ and _intent slot_ for the person name we want to greet!

![Skill Intent](https://res.cloudinary.com/danielcondemarin/image/upload/v1527345367/create_skill_4_vqzzfl.png)

Notice the name's slotType is already provided by Amazon, __AMAZON.GB_FIRST_NAME__. You will also want to set the slot required to fulfill the intent, to do that just click on the slot and enable the first toggle. Provide a prompt for the user to fill the slot too. 

Next up, the Skill Endpoint, but before we configure that, go and download [ngrok](https://ngrok.com/download). If you don't know what ngrok is, it's basically a way for you to expose on the public internet a local port or service in your machine.

Run ngrok and set it to forward to port 3000 over http:

```
./ngrok http 3000
```

Copy the HTTPS URL, not HTTP! as Amazon requires a secure connection, go to the developer portal and on the Endpoint section select an HTTPS endpoint and paste the ngrok URL plus the path __/echo/greeter__ (Mine was https://aa60ae80.ngrok.io/echo/greeter). Make sure you select _"My development endpoint is a subdomain of a domain that has a wildcard certificate ..."_.

Click Save Model, Build Model, and we are good to test! 

Here is a screenshot with a few utterances I tried:

![utterances](https://res.cloudinary.com/danielcondemarin/image/upload/v1527346120/create_skill_5_uqulio.png)

That's the end of the blog post, hope you've enjoyed it and feel free to comment below if you have any questions!











