﻿# Bots in Channels

Microsoft Teams allows users to bring bots into their channel conversations.  By adding a bot as a team member, all users of the team can take advantage of the bot functionality right in the channel conversation.  You can also access Teams-specific functionality within your bot like querying team information and @mentioning users.

To add a bot to a team, you'll need to follow the [packaging](createpackage.md) and [sideloading](sideload.md) instructions.

## Designing a great bot for channels

Bots added to a team become another team member, who can be @mentioned as part of the conversation.  In fact, bots only receive messages when they are @mentioned, so other conversations on the channel are not sent to the bot.

A bot in a channel should provide information relevant and appropriate for all members of the team.  While your bot can certainly provide any information relevant to the experience, keep in mind conversations with it are visible to all members of the channel.  Therefore, a great bot in a channel should add value to all users on the channel, and certainly not inadvertantly share information that would otherwise be more relevant in a personal context. 

Note that depending on your experience, the bot might be entirely relevant in both scopes (personal and team) as is, and in fact, no significant extra work is required to allow your bot to work in both.  In Microsoft Teams, there is no expectation that your bot function in all contexts, but you should ensure your bot makes sense, and provides user value, in whichever scope you choose to support.  For more information on scopes, see [Context in Teams](teamsapps.md).

## Building your bot

Developing a bot that works in channels uses much of the same functionality from 1:1 conversation.  There are additional events and data in the payload that provide Teams channel information.  Those differences, as well as key differences in common functionality are enumerated below.

### Receiving messages

For a bot in a channel, in addition to the [regular message schema](https://docs.botframework.com/en-us/core-concepts/reference/#activity), your bot will also receive the following properties:

* `channelData` - see [below](#teams-channel-data).
* `conversationData.id` - this value is the reply chain ID, consisting of channel id plus the id of the first message in the reply chain. 
* `conversationData.isGroup` - this value will be set to `true` for bot messages in channels
* `entities` - this object may contain one or more Mention objects (see [below](#mentions))

### Replying to messages

In order to reply to an existing message, call the `ReplyToActivity()` in [C#](https://docs.botframework.com/en-us/csharp/builder/sdkreference/routing.html#replying) or `session.send` in [Node.JS](https://docs.botframework.com/en-us/node/builder/chat/session/#sending-messages).  The BotFramework SDK handles all the details.

If you choose to use the REST API, you can also call the [/conversations/{conversationId}/activities/{activityId}`](https://docs.botframework.com/en-us/restapi/connector/#/Conversations) endpoint.  

Note that replying to a message in a channel shows as a reply to the initiating reply chain.  The `conversationId` contains the channel and the top level message id.  While the Bot Framework takes care of the details, you may cache that `conversationId` for future replies to that conversation thread as needed.

### Creating new channel conversation

Your team-added bot can post into a channel to create a new reply chain.  With the BotBuilder SDK, call  `CreateConversation()` for [C#](https://docs.botframework.com/en-us/csharp/builder/sdkreference/routing.html#conversationmultiple) or utilize Proactive Messaging techniques (`bot.send()` and `bot.beginDialog()`) in [Node.JS](https://docs.botframework.com/en-us/node/builder/chat/UniversalBot/#proactive-messaging).  

Alternatively, you can issue a POST request to the [`/conversations/{conversationId}/activities/`](https://docs.botframework.com/en-us/restapi/connector/#!/Conversations/Conversations_SendToConversation) resource.

Note: at this point, bots in Microsoft Teams cannot initiate 1:many / group conversations.

#### Example (.NET SDK)

```csharp
var channelData = new TeamsChannelData { Channel = new ChannelInfo(yourChannelId) };
IMessageActivity newMessage = Activity.CreateMessageActivity();
newMessage.Type = ActivityTypes.Message;
newMessage.Text = "Hello, on a new thread";
ConversationParameters conversationParams = new ConversationParameters(
    isGroup: true,
    bot: null,
    members: null,
    topicName: "Test Conversation",
    activity: (Activity)newMessage,
    channelData: channelData);
var result = await connector.Conversations.CreateConversationAsync(conversationParams);
```

### Best Practice - Welcome message

When your bot is first added to the team, it is a best practice to send a Welcome Message to the team, to introduce it to all users of the team, and tell a bit about its functionality.  To do this, make sure your bot responds to the `conversationUpdate` message, with the `teamsAddMembers` eventType in the `channelData` object.  Note that you must ensure that the `memberAdded` id is the Bot id itself, as the same event is sent when a new user is added to a team.  See [Team member or Bot addition](botevents.md#team-member-or-bot-addition) for more details.

You may also wish to send a 1:1 message to each member of the team when the bot is added.  To do this, you could [query the team roster](botapis.md#fetching-the-team-roster) and send each user a [direct messages](bots1on1.md#starting-a-11-conversation).

For more best practices, see our [design guidelines](design.md).

## Mentions

Bots in a channel only respond when they themselves are @mentioned in a message.  That means every message received by a bot in a channel contains its own name, and you must make sure your message parsing handles that case (see below for sample).  In addition, bots can parse out other users @mentioned and @mention users as part of their message as well.

### Retrieving mentions

Mentions are returned in the `entities` object in payload, and contain both the unique ID of the user and in most cases, the name of user mentioned.  You can retrieve all mentions in the message by calling the `GetMentions()` function in the BotFramework .NET SDK.  This should return an array of `Mentioned` objects.

Per above, all messages your bot receives will have its name in the channel, and should accomodate that in its message parsing.

#### Sample code - check for and strip @bot mention (.NET SDK)

```csharp
Mention[] m = sourceMessage.GetMentions();
var messageText = sourceMessage.Text;

for (int i = 0;i < m.Length;i++)
{
    if (m[i].Mentioned.Id == sourceMessage.Recipient.Id)
    {
        //Bot is in the @mention list.  
        //The below example will strip the bot name out of the message, so you can parse it as if it wasn't included.  Note that the Text object will contain the full bot name, if applicable.
        if (m[i].Text != null)
            messageText = messageText.Replace(m[i].Text, "");
    }
}
```

>Note: you may also use the Teams Extension function: `GetTextWithoutMentions()` which will strip out all mentions, including the bot.

#### Sample code - check for and strip @bot mention (Node.js)

```javascript

var text = message.text;
if (message.entities) {
    message.entities
        .filter(entity => (entity.type === "mention" && entity.id === yourbotid))
        .forEach(entity => {
            text = text.replace(entity.text, "");
        });
    text = text.trim();
}
```
>Note: you may also use the Teams Extension function: `getTextWithoutMentions()` which will strip out all mentions, including the bot.

### Constructing mentions

Your bot can @mention other users in messages posted into channels. To do this, your message must:
* Include `<at>@username</at>` in the message text.
* Include the `mention` object inside the entities collection

The Teams Extension SDK provides functionality to easily accomodate this.  See below.

>**Note**: At this time, team and channel @mentions are not supported.

#### .NET SDK sample

Note: This sample uses the [new Microsoft Teams .NET SDK](https://www.nuget.org/packages/Microsoft.Bot.Connector.Teams):

```csharp
// Create reply activity
Activity replyActivity = activity.CreateReply();

// Construct text of the form @sender Hello
replyActivity.Text = "Hello ";
replyActivity.AddMentionToText(activity.From, MentionTextLocation.AppendText);

// Send the reply activity
await client.Conversations.ReplyToActivityAsync(replyActivity);
```

#### Node SDK sample

Note: this sample uses [the new Microsoft Teams Node.js SDK](https://www.npmjs.com/package/botbuilder-teams):

```javascript
// User to mention
var toMention: builder.IIdentity = {
    name: 'John Doe',
    id: userId
};

// Create a new message and add mention to it
var msg = new teams.TeamsMessage(session).text(teams.TeamsMessage.getTenantId(session.message));
var mentionedMsg = msg.addMentionToText(toMention);

// Post the message
var generalMessage = mentionedMsg.routeReplyToGeneralChannel();
session.send(generalMessage);
```

#### Schema example - outgoing message with user @mentioned

```json
{ 
    "type": "message", 
    "text": "Hey <at>Larry Jin</at> check out this message", 
    "timestamp": "2016-10-29T00:51:05.9908157Z", 
    "serviceUrl": "https://skype.botframework.com", 
    "channelId": "msteams", 
    "from": { 
        "id": "28:9e52142b-5e5e-4d7b-bb3e- e82dcf620000", 
        "name": "SchemaTestBot" 
    }, 
    "conversation": { 
        "id": "19:aebd0ad4d6ab42c8b9ed19c251c2fc37@thread.skype;messageid=1481567603816" 
    }, 
    "recipient": { 
        "id": "8:orgid:6aebbad0-e5a5-424a-834a-20fb051f3c1a", 
        "name": "stlrgload100" 
    }, 
    "attachments": [ 
        { 
            "contentType": "image/png", 
            "contentUrl": "https://upload.wikimedia.org/wikipedia/en/a/a6/Bender_Rodriguez.png", 
            "name": "Bender_Rodriguez.png" 
        } 
    ], 
    "entities": [ 
        { 
            "type":"mention", 
            "mentioned":{ 
                "id":"29:08q2j2o3jc09au90eucae",
                "name":"Larry Jin" 
            }, 
            "text": "<at>@Larry Jin</at>"
        } 
    ], 
    "replyToId": "3UP4UTkzUk1zzeyW" 
}
```

## Accessing team context

Your bot can do more than send and receive messages in teams. For instance, it can also fetch the list of team members, including their profile information, as well as the list of channels. Visit the section on the [bot APIs](botapis.md) to learn more.