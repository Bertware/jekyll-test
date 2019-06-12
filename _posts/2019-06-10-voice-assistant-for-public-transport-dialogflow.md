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
Most of us have experienced this before.
You wake up in the morning, eat some breakfast, get dressed and run out of the door to catch a train or bus. 
Only to find out it is cancelled, or there are major delays.

While it's impossible to have every train and bus running on time, open data can at least prevent needless runs to the station 
by letting you shout at your Google Home, Echo Speaker, or your phone to ask about the next bus before you leave. 
Today we will create our own voice assistant app to provide us with real-time transport data through Google Assistant.

Example code is written in PHP, but any language works. All frameworks are free. No (credit) card is required for sign-up.

 [Featured image courtesy of Techcrunch](https://techcrunch.com/2017/11/22/snips-lets-you-build-your-own-voice-assistant-to-embed-into-your-devices/)
## I'm sorry, could you repeat that?
Language is hard. The first challenge when creating a (voice) assistant is understanding users, by both recognizing 
*what they say*, and figuring out *what they mean*. Luckily we don't need to solve this problem entirely by ourselves. 
There are already platforms available which take care of the speech recognition and the natural language processing (NLP).
Examples are Dialogflow (Google), Lex (Amazon), Bot Framework (Microsoft Azure), Snips, ...

Today we will explore Dialogflow, a service through which anyone can create apps on the Google Home platform, and with little extra effort on other platforms as well.
Other frameworks might have small differences in naming, but generally have the same structure. 

Head over to [the dialogflow website](https://dialogflow.com/) and sign in through a Google account. This Google account 
can be used later on to test on your device (you can add additional testing accounts as well).
Create a new _agent_ (this is what an 'app' is called) by filling in the name, and possibly changing the language. You 
can add other languages to your _agent_ later on.

There are 3 core concepts needed to build our app.

### Intents
With intents, you configure which actions your user can undertake, and what they will say to take a certain action. 
Take for example the following sentence: `When does the next train leave from Stockholm City ?`
    
When a user asks this, we want to show the next departures, for stop "Stockholm City", where the vehicle is a train. 
In order to realise this, we configure various training phrases. In our example, we used 10 different ways of asking for 
the next departure, with different stations and vehicles.

Now we create a new parameter for our intent. We will store the stop name here, and call our parameter location.
We use the Entity type `@sys.any`, meaning this can be anything. Setting this to, for example, @geo.city, would mean . Check the box to make it required, and enter some 
questions which your assistant can ask when this parameter is missing. An example is `From where do you want to leave?`.

Now we can mark the station names in our training phrases, and indicate that they will fill the location parameter. 
Now our agent will accept any station name, and extract it as a variable.    

When it comes to the train type, we do the same by creating a transportation-method variable. 
This case is a little special though, as we only want a limited list: only bus, ferry, metro, tram or train is valid. 
To make things more complex, not everyone will say metro. Some may use the word undergound instead. And is it ferry or boat? 
For this, we can use entities with synonyms. 


### Entities

Entities let you define `types` of variables. This influcences how the speech recognition will autocorrect the input.
 Built-in entities are for example 
 - `sys.geo-city`, which will autocorrect with city names
 - `sys.geo-capital`, which will autocorrect with capital names
 - `sys.airport`, which will autocorrect with airport names
 - `sys.number`, `sys.flight-number`, `sys.weight-unit`, ...  
 
Using `@sys.any` will accept any input, without trying to correct it to a certain set of names or a specific format.

While there is a wide range of built-in entities, we can also create our own. We will do this to recognize types of transport. 
The advantage in doing this is that there will be less errors in the user input, and our application will receive one of our 
given possibilities as transport mode. We create our own transport-method entity, and accept bus, tram, train, metro and ferry. 
We enter some synonyms for each method, to ensure we understand our user well, but dialogflow will internally replace the synonym with the 
reference value ( bus, tram, train, metro or ferry) when storing it in a variable. This means our own little server application which will
deliver the data doesn't need to think about synonyms.

**Important:** When adding a new language for your _agent_, you will need to create the entity again. In this case, use the same reference values
but use translated synonyms.

![Our own dialogflow entity]({{ site.baseurl }}/assets/images/2019-06-10-dialogflow-entity.png)
 
We now change our transportation method in our intent from `@sys.any` to `@transportation-method` to make use of our newly created entity.
Previously, when the user said "When does the next train in Stockholm leave", the voice assistant might understand "train in" as "training", and store "training" in the variable.
Now that we use our own entity, the voice assistant knows what can go in the transportation-method variable. If it recognizes "training" where a transportation method should go,
it will automatically correct it to "train in", and store "train" in the variable.

At this point our users can ask "When does the next subway leave from Stockholm", and 
- The intent will be detected
- The parameters will be extracted
- Subway will be replaced with metro in the variable

If either the location or transportation method aren't present, the user will receive one of the questions configured for that parameter.

### Fulfillment

While we now have an agent that can understand what users want, it still doesn't know what to answer. This is where fulfillment comes in.

> Fulfillment is code that's deployed as a webhook that lets your Dialogflow agent call business logic on an intent-by-intent basis. 
> During a conversation, fulfillment allows you to use the information extracted by Dialogflow's natural language processing to generate 
> dynamic responses or trigger actions on your back-end.

Whenever your intent is called, dialogflow will send an HTTP POST request to a URL of your choosing. Information about the intent which was used and the
parameters which were extracted is sent as JSON data in the request body. The server application now needs to process this data and formulate a response,
which will then be passed back to the digital assistant. You can see the entire process in the image below.

![The dialogflow process]({{ site.baseurl }}/assets/images/2019-06-10-dialogflow-process.png)

In order to enable fulfillment, head over to the fulfillment page on DialogFlow by clicking _fulfillment_ in the left sidebar.
Switch the slider to enabled, and enter the address of your server app here. You can also use the inline editor to write your fulfillment code in NodeJS on Firebase cloud functions.
If you don't have any URL just now, and you don't want to use the built-in editor, you can come back to this step later on when your fulfillment server is ready.
 
In order to be able to code a fulfillment application, we need to know the request and response format.
[This format is explained fully on the dialogflow website](https://dialogflow.com/docs/fulfillment/how-it-works).
You can find an example Dialogflow request and response in the V2 format below.
 
Headers:
 ```
 POST https://my-service.com/action
 
 Headers:
 //user defined headers
 Content-type: application/json
```
Request Body:
```json 
 {
   "responseId": "ea3d77e8-ae27-41a4-9e1d-174bd461b68c",
   "session": "projects/your-agents-project-id/agent/sessions/88d13aa8-2999-4f71-b233-39cbf3a824a0",
   "queryResult": {
     "queryText": "user's original query to your agent",
     "parameters": {
       "param": "param value"
     },
     "allRequiredParamsPresent": true,
     "fulfillmentText": "Text defined in Dialogflow's console for the intent that was matched",
     "fulfillmentMessages": [
       {
         "text": {
           "text": [
             "Text defined in Dialogflow's console for the intent that was matched"
           ]
         }
       }
     ],
     "outputContexts": [
       {
         "name": "projects/your-agents-project-id/agent/sessions/88d13aa8-2999-4f71-b233-39cbf3a824a0/contexts/generic",
         "lifespanCount": 5,
         "parameters": {
           "param": "param value"
         }
       }
     ],
     "intent": {
       "name": "projects/your-agents-project-id/agent/intents/29bcd7f8-f717-4261-a8fd-2d3e451b8af8",
       "displayName": "Matched Intent Name"
     },
     "intentDetectionConfidence": 1,
     "diagnosticInfo": {},
     "languageCode": "en"
   },
   "originalDetectIntentRequest": {}
 }
```
Response body:
```json 
{
  "fulfillmentText": "This is a text response",
  "fulfillmentMessages": [
    {
      "card": {
        "title": "card title",
        "subtitle": "card text",
        "imageUri": "https://assistant.google.com/static/images/molecule/Molecule-Formation-stop.png",
        "buttons": [
          {
            "text": "button text",
            "postback": "https://assistant.google.com/"
          }
        ]
      }
    }
  ],
  "source": "example.com",
  "payload": {
    "google": {
      "expectUserResponse": true,
      "richResponse": {
        "items": [
          {
            "simpleResponse": {
              "textToSpeech": "this is a simple response"
            }
          }
        ]
      }
    },
    "facebook": {
      "text": "Hello, Facebook!"
    },
    "slack": {
      "text": "This is a text response for Slack."
    }
  },
  "outputContexts": [
    {
      "name": "projects/${PROJECT_ID}/agent/sessions/${SESSION_ID}/contexts/context name",
      "lifespanCount": 5,
      "parameters": {
        "param": "param value"
      }
    }
  ],
  "followupEventInput": {
    "name": "event name",
    "languageCode": "en-US",
    "parameters": {
      "param": "param value"
    }
  }
}
```
More details on setting up fulfillment using Firebase, Google cloud, or using NodeJS libraries can be found in 
[the dialogflow docs](https://dialogflow.com/docs/fulfillment/configure).

### Putting it to the test

## Creating a custom fulfillment server application

We now know the request format, and how our response should look like. In order to be able to anwer, we will use the [trafiklab](https://trafiklab.se)
APIs. Depending on which features we want to implement, there are two or three types of API requests which we will use:
- Station lookups: The station name, passed as a parameter by DialogFlow, needs to be converted to an ID so we can specify which station we mean
when talking to other APIs.
- Departure boards: A departures API gives information about the next departures from a given stop location.
- Routeplanning: If you want to create an extra intent that handles routeplanning, you will need an API like this to quickly provide you with a response.

The following table shows which API can be used for each purpose, depending on the region for which you want to get information.

|                  | Sweden                                                                                     | Stockholm                                                                                  |
|------------------|--------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
| Station lookup   | [ResRobot Reseplanerare](https://www.trafiklab.se/api/resrobot-reseplanerare/platsuppslag) | [SL Platsuppslag](https://www.trafiklab.se/api/sl-platsuppslag)                            |    
| Departure boards | [ResRobot Stolptidstabeller 2](https://www.trafiklab.se/api/resrobot-stolptidtabeller-2)   | [SL Realtidsinformation 4](https://www.trafiklab.se/api/sl-realtidsinformation-4)          |    
| Routeplanning    | [ResRobot Reseplanerare](https://www.trafiklab.se/api/resrobot-reseplanerare/sok-resa)     | [SL Reseplanerare 3.1](https://www.trafiklab.se/api/sl-reseplanerare-31)                   |    
  
\* ResRobot also works in Stockholm, but the SL APIs might offer better accuracy and realtime data.

\** API versions are the latest at time of writing. Future versions will be just as suited for this. 

### The easy way
If you're using PHP, you're lucky! We developed open-source SDKs which offer those 3 features, both for the ResRobot and SL APIs.
They are interchangeable, meaning your code doesn't need any modification to change the data source. They are also updated when APIs
are updated, meaning you don't get breaking changes in your code.

The SDKs can be found at Github:
- ResRobot: https://github.com/trafiklab/resrobot-php-sdk
- SL: https://github.com/trafiklab/sl-php-sdk

Installation is as easy as `composer require trafiklab/resrobot-php-sdk` or `composer require trafiklab/sl-php-sdk`.

For an example on how these are used, you can look at the readme docs, or in [our Google Assistant demo project](https://github.com/trafiklab/google-assistant-demo/blob/master/app/Http/Controllers/NextDepartureController.php#L62).

### Consuming the APIs in your own programming language
Unfortunately our SDKs are only available in PHP right now, but let us know if you want to see them in an other language! 
Luckily, using the APIs isn't too hard. All the documentation can be found on [trafiklab](https://trafiklab.se). The 
table above contains direct links to the specific API documentation. You can still use the source code of our SDKs to see 
which parameters we send to the APIs and which fields we parse. If you get stuck, you can always reach out to Trafiklab [using
the Kundo forum](https://kundo.se/org/trafiklabse/).