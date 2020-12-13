# rpabotsworldbot
rpabotsworldbot - uipath + telegram bot
Telegram Bot Integration With UiPath
Tags: uipath, uipath examples, uipath tutorial, captcha, telegram, bot.
 
Telegram Bot needs no introduction today and with the growing usage of chat bot in RPA industry it is obvious choice to integrate your workflow with telegram bot.
In this tutorial, we are going to write a small UiPath process which interacts with Telegram Bot chat: send to user a screenshot with Captcha image (from website https://captcha.com/demos/features/captcha-demo.aspx), wait for message with recognized Captcha text or command to change Captcha or command to speak Captcha out.
Project .xaml you can get via GitHub – You’ll just need to add your Telegram Bot token and ChatId (id of chat of bot with you)  https://github.com/mflow42/rpabotsworldbot 
Action plan
1.	Add UiPath.Web.Activities package in Manage Packages
2.	Create Telegram Bot
3.	Create initial UiPath process.
4.	Get target ChatId to connect bot with you.
5.	Finalize UiPath with Telegram process.

Add UiPath.Web.Activities package in Manage Packages
To work with “HTTP request” and “Deserialize JSON” activity you need to first add UiPath Web Activities Pack as dependencies in your project.
You can enable and install it from Manage Packages window as shown below.
  Select UiPath Web Activities Pack and click on install to add required dependency. As soon as they are added you will be able to see web activities from activities panel as below.
 
So far so good. Let’s move to steps for creating a new Telegram Bot.
Create new Telegram Bot
Open telegram app: mobile, desktop or web (https://web.telegram.org/). Register if you not did it before. Then follow instructions:
1.	Search “botfather”
2.	Open proper chat
3.	Create new bot by typing “/newbot”
4.	Give bot name and username
5.	Copy bot token – you’ll need it later.
6.	Send any test message – to get ChatId in future steps.
My example below:
 
Create initial UiPath process.
We will use only following types of Telegram methods:
1.	getUpdates – to get messages and callbacks from user.
2.	sendPhoto – to send screenshot.
3.	sendMessage – to send message user.
Read More at Telegram Documentation - https://core.telegram.org/bots/api#authorizing-your-bot 

First of all you need to add “Use Application\Browser” activity and assign it to the web site https://captcha.com/demos/features/captcha-demo.aspx.
 

Then add Flowchart and:
1.	Assign string Token to remembered before telegram bot token.
2.	Assign string UpdateId to “0”.
3.	Assign string Offset to ((CInt(UpdateId)+1).ToString. It needs to get not all but only last update from chat. By default, it is incremented value of UpdateId by 1.
4.	Add HTTP request activity and fill in it:
Assign EndPoint to "https://api.telegram.org/bot"+Token+"/getUpdates"
Add Parameters:
	String offset = Offset
	String limit = “1” 
 
 
Our Workflow will look like this
 
Then add Deserialize JSON activity to deserialize response from server.
Then add assign to UpdateId = If(ResultJObj("result") IsNot Nothing AndAlso ResultJObj("result")(0)("update_id") IsNot Nothing, ResultJObj("result")(0)("update_id").ToString, UpdateId)
Then add assign ChatId and make Breakpoint at this activity to get ChatId from response in Local Panel via Debug mode.
 
Get target ChatId to connect bot with you
Run Project in debug mode and see what stored in Result variable to get ChatId.

 
Then assign valid ChatId. My example below:
 
Finalize UiPath with Telegram process.
Now we know all necessary items: Token and ChatId. So we can move on to the most interesting part of this guide.
Making keyboard layout for telegram bot
More at https://core.telegram.org/bots/api#inlinekeyboardmarkup 
I chose this type of markup and it has following JSON form – there are two buttons “change” and “speak” which contains appropriate “callback_data”.
Assign KeyboardMarkup = "{""inline_keyboard"": [[{""text"": ""change"",""callback_data"": ""change""},{""text"": ""speak"",""callback_data"": ""speak""}]]}"
It will be like this in chat:
 
Send screenshot with Captcha.
Add “Take screenshot” activity, set it to Captcha image and save image file path to variable CaptchaImagePath:
 
 
Then send it to telegram chat via HTTP Request activity:
 
 
Fill in path to screenshot with Captcha image:
 
Fill in Parameters:
 
In chat it will be like this:
 
Listen answer from user
Add another HTTP Request Get Updates activity.
Add another Deserialize JSON activity.
Add Flow Decisions activity to check whether answer is callback or message: get action if it is a callback and Captcha text if it is a message.
  
Answer is Callback?
ResultJObj("result").Count > 0 AndAlso ResultJObj("result")(0)("callback_query") IsNot Nothing

If it is a callback then save Action:
ResultJObj("result")(0)("callback_query")("data").ToString

Answer is a Message?
ResultJObj("result").Count > 0 AndAlso ResultJObj("result")(0)("message") IsNot Nothing

If it is a message then save AnswerText:
ResultJObj("result")(0)("message")("text").ToString
React on actions
We have 2 actions: change and speak. So every action is a click to appropriate button:
 
 
 
Then we need to send a message to user to give a feedback:
 
 

 

Then update UpdateId and loop to “Take Screenshot 'Captcha'”. Workflow will look like this:
 

React on message
Then we need to process getting and typing into Captcha text. If answer is message with Captcha text then we need to save AnswerText from ResultJObj:

ResultJObj("result")(0)("message")("text").ToString

Then add “Type Into” activity to type this answer to Captcha validation input:
 
Then add “Click” activity to button “Validate”:
 
Then we need to get result text of validation:
 
Then direct to “HTTP Request sendMessage” activity. If answer is not a message then we need to direct flow to “HTTP Request getUpdates” activity.
Workflow will look like this:
 
Overall workflow from “Take Screenshot 'Captcha' activity will look like this:
 
Final words
Please feel free to reach me in case you are still facing any issues with your Telegram integration with Uipath.
Happy Automation!
