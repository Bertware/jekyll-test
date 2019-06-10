---
layout: post
title: "Creating a voice assistant for public transport"
author: bert
categories: [ tutorial ]
tags: [google home, google assistant, sl, resrobot]
image: assets/images/2019-06-10-voice-assistants.png
description: "Hey Google, when does the next train leave from Stockholm? Building your own Google Assistant app with dialogflow"
featured: true
hidden: true
---
[Image courtesy of Techcrunch](https://techcrunch.com/2017/11/22/snips-lets-you-build-your-own-voice-assistant-to-embed-into-your-devices/)

Most of us have experienced this before.
You wake up in the morning, eat some breakfast, get dressed and run out of the door to catch a train or bus. 
Only to find out it is cancelled, or there are major delays.

While it's impossible to have every train and bus running on time, open data can at least prevent needless runs to the station 
by letting you shout at your Google Home, Echo Speaker, or your phone to ask about the next bus before you leave. 
Today we will create our own voice assistant app to provide us with real-time transport data through Google Assistant.

Example code is written in PHP, but any language works. All frameworks are free. No (credit) card is required for sign-up.

## I'm sorry, could you repeat that?
Language is hard. The first challenge when creating a (voice) assistant is understanding users, by both recognizing *what they say*, and figuring out *what they mean*. 
Luckily we don't need to solve this problem entirely by ourselves. There are already platforms available which take care of the speech recognition and the natural language processing (NLP).
Examples are Dialogflow (Google), Lex (Amazon), Bot Framework (Microsoft Azure), Snips, ...

Today we will explore Dialogflow, a service through which anyone can create apps on the Google Home platform, and with little extra effort on other platforms as well.
Other frameworks might have small differences in naming, but generally have the same structure. 

Head over to [the dialogflow website](https://dialogflow.com/) and sign in through a Google account. This Google account can be used later on to test on your device (you can add additional testing accounts as well).
Create a new _agent_ (this is what an 'app' is called) by filling in the name, and possibly changing the language. You can add other languages to your _agent_ later on.

There are 3 core concepts needed to build our app.

### Intents
With intents, you configure which actions your user can undertake, and what they will say to take a certain action. Take for example the following sentence:

    When does the next train leave from Stockholm City ?
    
When a user asks this, you want to show the next departures, for stop "Stockholm City", where the vehicle is a train. In order to realise this, we configure various training phrases.
In our example, we used 10 different ways of asking for the next departure, with different stations and vehicles.

Now we create a new parameter for our intent. We will store the stop name here, and call our parameter location.
We use the Entity type `@sys.any`. More about this entity type later. Check the box to make it required, and enter some 
questions which your assistant can ask when this parameter is missing. An example is `From where do you want to leave?`.

Now we can mark the station names in our training phrases, and indicate that they will fill the location parameter. 
Now our agent will accept any station name, and extract it as a variable.    

When it comes to the train type, we do the same by creating a transportation-method variable. 
This case is a little special though, as we only want a limited list: only bus, ferry, metro, tram or train is valid. 
To make things more complex, not everyone will say metro. Some may use the word undergound instead. And is it ferry or boat? 
For this, we can use entities with synonyms. 


### Entities


### Fulfillment
