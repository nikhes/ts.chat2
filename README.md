# Chat widget - version 2
*Inspired from [ts.chat](https://github.com/TheSmiths-Widgets/ts.chat).*

This is a small chat view (list of messages and text area to send new ones). The widget is being given a collection. Its only job is to display the collection. The collection order is preserved so you have to sort it manually, on your side, if you intend so. When the user writes and sends a message from the text field, an event is fired ('*newmessage*'). Another kind of event is fired when the user scrolls to the top ('*moremessages*').

![Demo](https://raw.githubusercontent.com/rpellerin/ts.chat2/develop/demo.gif)

*You have a full demo example project in the sample branch*

##Manifest
* Version: 1.0
* License: not specified yet
* Author: rpellerin
* Supported Platforms: Android, iOS

## Adding to Your Alloy Project
* In your application's config.json file you need to include the following line in your dependencies:

```
"dependencies": {
    "ts.chat2": "*"
}
```

*  Create a widgets directory in your app directory if it doesn't already exist.
*  Copy the ts.chat2 folder into your app/widgets directory.

Opposed to version one of ts.chat, we need no more dependencies :)

## Basic model info
It uses Alloy models and collections ([http://docs.appcelerator.com/titanium/3.0/#!/guide/Alloy_Collection_and_Model_Objects](http://docs.appcelerator.com/titanium/3.0/#!/guide/Alloy_Collection_and_Model_Objects)) which are based on [Backbone.js collections](http://backbonejs.org/).

The model must have, at least, those 3 properties:

- **content**: the actual message
- **emitter**: either a string or an object, it's the sender
- **created_at**: the date of the message

Example:

```javascript
var msg = Alloy.createModel('Message', {
    content: "Hello world",
    emitter: Alloy.User.objectId, // Alloy.User is an object we created in alloy.js, for example
    created_at: new Date(2015, 12, 31, 0, 0, 0)
});
```

## Create the chat in the View
You will want to use the chat in the whole window so you can do this in your view: 

```xml
<Alloy>
	<Window id="chatWin" title="My Chat">
		<Widget id="chat" src="ts.chat2" />
	</Window>
</Alloy>
```

Assign it an ID that you can use in your controller. E.g. `id="chat"` You can now access the Calendar via `$.chat` in your controller.

## Create the Model in the project
Following the example from the *sample* branch, you need to define your Model in the *app/models* folder (create the folder if it doesn't exists). You can have more than the basic fields, but at least the 3 properties described in "Basic model info" must be defined.

## Define the Collection in the project
Go to *app/alloy.js* file and define the Collection you will attach to the chat there (see *sample* brach)

```
Alloy.Collections.discussion = Alloy.createCollection('Message');
```

## Initialize the chat
You are ready to use the data. You can read the current messages and add to the Collection from any source you want or start empty.

Anyway, once your Collection is ready, you must initialize the chat:

```javascript
var validateSender = function(model) {
    return model.get('emitter') == 'Current user ID';
}

$.chat.init({
    messages: Alloy.Collections.discussion,
    validateSender: validateSender
});
```

This will show all your messages and the chat is ready for interaction.
**validateSender** is a function in where you say to the chat who is the user that "emits" the messages (the other will be the receiver).
You will assign here the user "identifier" to be on the right side of the conversation.

## Send a new message
As easy as:

```javascript
$.chat.on('newMessage', function (newMessageEvent) {
    var message = Alloy.createModel('Message', {
         content: newMessageEvent.message,
         emitter: 'Current user ID',
         created_at: newMessageEvent.created_at
     });
    Alloy.Collections.discussion.add(message);
    newMessageEvent.success(); // Mandatory, to acknowledge sending the message successfully
});
```

## Adding messages from "server"
```javascript
$.chat.on('moremessages', function () {
    // Fetch a remote server and add data into Alloy.Collections.discussion
});
```

## Closing the chat
If you will reuse the Collection for other chats, remember to reset it when closing the view. Also free resources:

```javascript
function close() {
	Alloy.Collections.discussion.reset();
	$.destroy();
    $.chatWin.close();
}
```

## Styling a bit:

Feel free to enclose the chat in a view and style it a little bit in your tss file:

```xml
<Alloy>
    <View class="container">
        <View id="chatWrapper">
            <Widget id="chat" src="ts.chat2"></Widget>
        </View>
    </View>
</Alloy>
```

```css
".container" : {
    width: Ti.UI.FILL,
    height: Ti.UI.FILL
}

"#chatWrapper" : {
    height: Ti.UI.FILL,
    width: Ti.UI.FILL
}
```

## Initialization Parameters

| Parameter | Type | Description |
| --------- | ---- | ----------- |
| messages | *Collection* | Initial set of messages. |
| validateSender | *function* | Function that takes one argument, a model. Must returns TRUE if the message is from you and then has to be displayed on the right side, otherwise it returns FALSE. |
| maxTypingHeight (optional) | *Number or decimal* | The max size of the typing area (it can grow automatically). If decimal less than 1, a corresponding percentage of the screen size will be used (default 0.25). |
| batchSize (optional) | *Number* | How many message should be ask for each load (default 10). |

## Accessible Methods
All internal methods are private, but you can listen to two triggers to do some actions

| Trigger | Description | Example |
| ------- | ----------- | ------- |
| newMessage | Allows you to send a new message. Triggered when user press "send" button. | `$.chat.on("newMessage", function (newMessageEvent) {...});` |
| moremessages | Allows you to add a bunch of messages to the Collection. Triggered when user scrolls to top of the calendar | `$.chat.on("moremessages", function () {...});` |

## TODO (from the most important to the least)

- Add some customization
    - Allow to add more buttons at the bottom (like in the Hangout app from Google)
        - Each button would raise its own event when pressed
    - Allow to change the send button (image or text)
    - Enable i18n
- Generate the documentation into ```gh-pages```

[![wearesmiths](http://wearesmiths.com/media/logoGitHub.png)](http://wearesmiths.com)
