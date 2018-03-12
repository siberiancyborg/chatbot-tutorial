
Step by step guide: how to make your own Facebook messenger  chatbot using node.js and ngrok.
 <img src="https://s9.postimg.org/ay4jf6z1b/Screen_Shot_2018-03-12_at_20.31.48.png" class="centerImage" width="580" height="300"/>
## What will you need?
1. Atom https://atom.io/
2. node.js >= 4.0.0 (https://nodejs.org/en/download/)
3. Messenger-bot from github (https://github.com/remixz/messenger-bot)
4. Ngrok https://ngrok.com/
5. Facebook documentation (https://developers.facebook.com/docs/messenger-platform)

## Create a FB page (if you know it already go to the next step)
At least for now you can’t to separate communication between your bot and for example a manager for your page. So it makes sense to create a new page special for your bot. 
How to do that?
Eaaaasy 👌

1. Go the this page https://www.facebook.com/pages/create/

 <img src="https://cdn-images-1.medium.com/max/1600/1*rE_pg264YFwr8Uked7PyXg.png" class="centerImage" width="480" height="345"/>

2. I picked Entertainment option. In next steps it asks different options which you can always skip.

3. Great! My page is ready and something like should be visible for you as well. I added the avatar and background photo (optional).

 <img src="https://cdn-images-1.medium.com/max/1600/1*ioU6dIS6AWfipvY0YuPTzA.png" class="centerImage" width="480" height="345"/>

## Setup the Facebook App
1. Register as a Facebook developer developers.facebook.com/apps
2. Choose “basic setup”

 <img src="https://cdn-images-1.medium.com/max/1600/1*RMHojq2KMP_sRvelpRZjGQ.png" width="480" height="345"/>

3. Fill the form, click “Create APP id”.

 <img src="https://cdn-images-1.medium.com/max/1600/1*_Fevz5jB7hx7VDkxmCxj5g.png" width="480" height="345"/>

4. Next screen, in the left side check “messenger” and click “start”
5. Then choose a FB page for a bot (that you created before) and copy the token. Save somewhere this token, because you will need it again quite soon.

<img src="https://cdn-images-1.medium.com/max/1600/1*nrLT3Xti1-MxAGxKrQNxDw.png" width="480" height="245"/>

## Install messenger-bot, a node client for sending and receiving messages
First, make sure you have node.js >= 4.0.0 installed. Then, open up terminal, navigate to the directory you want to host your project in, and run:

     npm install messenger-bot

messenger-bot is an excellent, lightweight node client that will save you the hassle of setting up a node server from scratch. Even so, the code for messenger-bot is only 131 lines, making it extremely easy to understand and further customize, if you wish.

Create a new file called app.js and copy-paste in this code, originally taken and modified from messenger-bot's examples directory (Note: if you copy-paste directly from github, it won't work!).

      'use strict'
      const http = require('http')
      const Bot = require('messenger-bot')

      let bot = new Bot({
        token: 'PAGE_TOKEN',
        verify: 'VERIFY_TOKEN'
      })

      bot.on('error', (err) => {
        console.log(err.message)
      })

      bot.on('message', (payload, reply) => {
        let text = payload.message.text

        bot.getProfile(payload.sender.id, (err, profile) => {
          if (err) throw err

          reply({ text }, (err) => {
            if (err) throw err

            console.log(`Echoed back to ${profile.first_name} ${profile.last_name}: ${text}`)
          })
        })
      })

    http.createServer(bot.middleware()).listen(3000)

Go back to your Facebook app page and replace PAGE_TOKEN in your app.js file with your actual token. Feel free to leave VERIFY_TOKEN as is; if you change it, be sure to use the same token in the next step, when you set up your bot's webhook subscription. To run your server, type node app.js in your terminal. If there are no error messages, that means your server is working!


## Setup webhooks

To receive messages and other events sent by Messenger users, the app should enable webhooks integration. 
There are different options how to do that, as I mentioned before I will use ngrok. 

1. Setup ngrock

Use ngrok to set up a secure tunnel to your localhost server. ngrok is a tool that allows you to easily expose your localhost server to the outside world, and I'm including it in this tutorial because it's the fastest way to get your bot up and running.
If you don't already have ngrok installed, download it here (https://ngrok.com/).

Then open up a new tab in terminal and run (note: unable port, ask Matthias)
      ./ngrok http 3000.
You should see a screen displaying information about the tunnel; for this tutorial, you'll need the https url forwarding to your localhost, which in my case is https://944e4d4b.ngrok.io (second link, every important to have it with a certificate. Copy this link to your clipboard, as Facebook will ask for it in the next step.

<img src="https://s9.postimg.org/4kfgbv9jz/Screen_Shot_2018-03-12_at_20.32.04.png" width="480" height="245"/>

##### Note: Of course, if you want your bot to be available 24/7 even when your computer's not running, you'll eventually want to deploy your server to a third-party hosting service like Heroku, AWS, or Azure. Just don't forget to update webhooks and run the curl command (steps 4 and 5) from that server as well!

2. Set up your Facebook Messenger webhooks.
Return to your app page and click the "Setup Webhooks" button in the "Webhooks" section. Paste your ngrok url into the callback url, use "VERIFY_TOKEN" as your verify token (it's the default in your app.js file; feel free to change both if you wish), select all the subscription fields, and press "Verify and Save."
<img src="https://s9.postimg.org/97lie54rz/Screen_Shot_2018-03-12_at_21.43.42.png" width="480" height="245"/>

If you done everything right, you should see this screen:

<img src="https://cdn-images-1.medium.com/max/1600/1*h5Vp2Dq0XvusLU0W9U7sLw.png" width="480" height="245"/>

3. Also, please don't forget to choose your page. 

<img src="https://s9.postimg.org/4zvbioalb/Screen_Shot_2018-03-12_at_21.43.53.png" width="480" height="245"/>


## Subscribe your app to the Facebook page

 Go again to Terminal and type in this command to trigger the Facebook app to send messages. Remember to use the token you requested earlier? We need it now. Just don’t forget instead “<PAGE_ACCESS_TOKEN>” post your own token. 

     url -X POST "https://graph.facebook.com/v2.6/me/subscribed_apps?access_token=<PAGE_ACCESS_TOKEN>"


Now let’s see what our bot actually can do.

## Making easy echo-bot🚀

### Setup the bot

Add an API endpoint to index.js to process messages. Remember to also include the token we got earlier (<PAGE_ACCESS_TOKEN>) .

      app.post('/webhook/', function (req, res) {
          let messaging_events = req.body.entry[0].messaging
          for (let i = 0; i < messaging_events.length; i++) {
              let event = req.body.entry[0].messaging[i]
              let sender = event.sender.id
              if (event.message && event.message.text) {
                  let text = event.message.text
                  sendTextMessage(sender, "Text received, echo: " + text.substring(0, 200))
              }
          }
          res.sendStatus(200)
      })

      const token = "<PAGE_ACCESS_TOKEN>"

### 2. Add a function to echo back messages. 
      function sendTextMessage(sender, text) {
          let messageData = { text:text }
          request({
              url: 'https://graph.facebook.com/v2.6/me/messages',
              qs: {access_token:token},
              method: 'POST',
              json: {
                  recipient: {id:sender},
                  message: messageData,
              }
          }, function(error, response, body) {
              if (error) {
                  console.log('Error sending messages: ', error)
              } else if (response.body.error) {
                  console.log('Error: ', response.body.error)
              }
          })
      }
3. Now we need again to deploy it to our server. 

4. Go to the Facebook Page and click on Message to start chatting! Now your bot will repeat all the words you type. Still not an AI, but we are on the way! 

<img src="https://cdn-images-1.medium.com/max/1600/1*36fSOIPmm7NVN4PL4hxLFg.png" width="480" height="345"/>


Now let’s check the most interesting part — how we can customise our bot.

### Chatbot customisation  🎨
I’ll go through all the important parts of creating your chatbot like: greeting, get started button, structed messages etc. 
Let’s start with the entry points!

#### Making Getting Started button
Copy the code below to your Terminal and run it. You could check this code in the Facebook documentation. Don’t forget to change “payload” and of course “PAGE_ACCESS_TOKEN”. Let’s use payload “NEWS” that I have already (you will see it later in the code).

   curl -X POST -H "Content-Type: application/json" -d '{
     "setting_type":"call_to_actions",
     "thread_state":"new_thread",
     "call_to_actions":[
       {
         "payload":"NEWS"
       }
     ]
   }' "https://graph.facebook.com/v2.6/me/thread_settings?access_token=PAGE_ACCESS_TOKEN"

2. If everything is okay you will see something like that in your Terminal.

<img src="https://cdn-images-1.medium.com/max/1600/1*qwnuOXITjRyaQ2ViHpfbBw.png" width="480" height="345"/>

3. Let’s check how it works. Delete your bot from the messenger so you start conversation again. You should see “Get started button”.

<img src="https://cdn-images-1.medium.com/max/1600/1*0Mh_6VG7Vi3ASrzdQSAuaA.png" width="480" height="345"/>

#### Making the greeting text 
Okay, besides “Get Started” button you could create a welcome message. This can be used obviously only once. 
The same algorithm. 
Copy this code to your terminal and run it. You could customise the message ( but it has 160 character limit). And… Change access_code to your token.

   curl -X POST -H "Content-Type: application/json" -d '{
     "setting_type":"greeting",
     "greeting":{
       "text":"Welcome to BotSpot Vienna!"
     }
   }' "https://graph.facebook.com/v2.6/me/thread_settings?access_token=PAGE_ACCESS_TOKEN"

#### Send a Structured Message
There are different types of messages you could see make on Facebook: the Button and Generic Template can render buttons that open a URL or make a back-end call to your webhook. The Receipt Template can be used to send a receipt.

  <img src="https://cdn-images-1.medium.com/max/1600/1*iu-LKrjgeA-OYgFP5HK50g.jpeg" width="480" height="345"/>

 How we could implement it? 
Copy the code below to index.js to send an test message back as a menu I use for my bot.

As you could see with Node.js you can use emojis, so let’s make our Generic more user friendly. 😌
2. Update the webhook API to look for special messages to trigger the menu and to send back a postback function.

      app.post('/webhook/', function (req, res) {
          let messaging_events = req.body.entry[0].messaging
          for (let i = 0; i < messaging_events.length; i++) {
            let event = req.body.entry[0].messaging[i]
            let sender = event.sender.id
            if (event.message && event.message.text) {
              let text = event.message.text
              if (text === 'Menu') {
                  sendGenericMessage(sender)
                  continue
              }
              sendTextMessage(sender, "Text received, echo: " + text.substring(0, 200))
            }
            if (event.postback) {
              let text = JSON.stringify(event.postback)
              sendTextMessage(sender, "Postback received: "+text.substring(0, 200), token)
              continue
            }
          }
          res.sendStatus(200)
        })

3. Open up Terminal again and deploy it.

4. Let’s try it! Type “Menu” and see what’s happens.

  <img src="https://cdn-images-1.medium.com/max/1600/1*mGoT-5pqdLChM1TH-7aqMg.png" width="480" height="345"/>

  For further customisation please check the Facebook documentation: https://developers.facebook.com/docs/messenger-platform

  Best of luck with your bots, thanks for reading! :)

# Rene is awesome!!
