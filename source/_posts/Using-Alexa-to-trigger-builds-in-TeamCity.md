---
title: Using Alexa to trigger builds in TeamCity
date: 2018-06-10 01:53:00
tags:
---


![alexateamcity](https://res.cloudinary.com/danielcondemarin/image/upload/v1528591697/alexa_teamcity_cover_xs8rx5.jpg)

That's right, wouldn't be cool if you could trigger your builds from Alexa? Well, you can now, at least if you're using TeamCity.

**What you'll need:**
*	Go (https://golang.org/dl/)
*	ASK CLI (https://www.npmjs.com/package/ask-cli)
*	ngrok (https://ngrok.com/download)

Make sure you install and setup all of that before what comes next.

**Let's have a look at the following diagram:**

![diagram](https://res.cloudinary.com/danielcondemarin/image/upload/v1528586168/AlexaTeamCity_q4su1o.png)

Note how we're not using the typical Alexa Skill deployment approach backed by an AWS Lambda. Instead, we're hosting a local skill server which will be publicly visible on the internet via ngrok. But why!!? Well, the reason is simple, my TeamCity Server is hosted on a private network, neither addressable on the Internet or hosted in a VPC in AWS, so AWS Lambda wasn't an option. I guess there could be other options to expose your local TeamCity Server, but I just wanted to try this approach instead.

**Okay ... how do I set it all up!?**

Ngrok will expose our local skill server to the Internet. Open a terminal and run `ngrok http 3000`, take a note of the HTTPS Forwarding URL:

![ngrok](https://res.cloudinary.com/danielcondemarin/image/upload/v1528463102/AlexaTeamCity_ngrok_epnugo.png)

Keep ngrok running, we'll comeback to it later.

Next, pull and install the local skill server: 

`go get github.com/danielcondemarin/ci-commander/alexa-teamcity`

Open another terminal and go to the project src (Pro Tip: Use `tmux`):

`cd $GOPATH/src/github.com/danielcondemarin/ci-commander/alexa-teamcity/`

Edit the file `skill.json` and add your ngrok HTTPS Forwarding URL to apis.custom.endpoint.uri (Leave /echo/teamcity in the path!). Mine looked like:

```
"apis": {
      "custom": {
        "endpoint": {
          "sslCertificateType": "Wildcard",
          "uri": "https://12db3c1b.ngrok.io/echo/teamcity"
        },
        "interfaces": []
      }
    }
```

It's time now to add your build types to the interaction model. We need this so Alexa can identify them in the user utterances. For example:

![Utterance](https://res.cloudinary.com/danielcondemarin/image/upload/v1528538256/AlexaTeamCityUtterance_ejvrb4.png)

Open `models/en-GB.json`, go to `types[].values[]` and add your builds with its synonyms. If you have one of those strange build ids with underscores etc. you can set the "value" with the original build id, then synonyms can be easier to pronounce. For example, this is what my config looks like:

```
"types": [
        {
          "values": [
            {
              "name": {
                "value": "SuperPoo_Build",
                "synonyms": ["superpoo", "poo"]
              }
            }
          ],
          "name": "BuildType"
        }
      ]
```

We're nearly there now! If you haven't already, run `ask init` and setup your profile etc.

Then, make sure you're in the directory `alexa-teamcity` and deploy the skill interaction model:

`ask deploy`

Lastly, grab the alexa `skill_id` form the file `.ask/config` and spin up the local skill server. Make sure you replace the env variables with your own values:

`ALEXA_APP_ID="amzn1.ask.skill.xxxx-xxx" TEAMCITY_URL="http://127.0.0.1:8111" TEAMCITY_USER="user" TEAMCITY_PASS="pwd" /Users/daniel/Projects/go/bin/alexa-teamcity`

If you've reached this point without issues, here is a quick demo of what you should be able to do!

{% vimeo 274291984 %}

**Final Notes**

Apart from the learning involved in setting it all up, there is no reason why you couldn't use this with an amazon echo and bring your CI to a whole new level of fun in the office! 