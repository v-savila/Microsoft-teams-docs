﻿# Getting Started with Bots for Microsoft Teams

Build and connect intelligent bots to interact with Microsoft Teams users naturally through chat.  Or provide a simple commands-based bot, to used as your "command-line" interface for your broader Teams app experience.  You can even bring your existing bot from another channel, and add Teams-specific support to make your experience shine. 

If you are just looking for a way to simply extend your team by integrating with custom tools and services in a secure manner, check out our new [Custom Bots](custombot.md) feature. 

## Bots in Microsoft Teams

> At this time, Microsoft Teams bots support 1:1 chats and channel conversations. They do not yet support group chats. 

Other than their hexagonal avatar icon, bots appear just like any other team member you interact with, in a team or in 1:1 conversation.  They are always online, and do not have a mood message.

Bots in Teams can surface in a 1:1 context ("personal scope"), as a member of a team ("team scope"), or both.  For the latter, they only take part in a conversation when you @mention them.  For the former, you can address them via the conversation interface, or access them in the apps personal experience from the app bar flyout.

With Microsoft Teams apps, you can choose make the bot the star of your experience, or just a helper.  They will be distributed as part of your broader app package, which may include other capabilities such as tabs or compose extensions.

If you bot is the star, be sure to take advantage of the [tabs](tabs.md) capability as well. Use this rich web view to surface accompanying experiences and information that helps your users best interact with your service.


## Overview of building a Microsoft Teams bot

Microsoft Teams supports much of the common [Bot Framework](https://dev.botframework.com/) functionality.  Follow these steps to build a great Teams bot:

- [Design a great bot](design.md#building-a-great-bot): Microsoft Teams is the place for group and team collaboration.  Consider what functionality your bot can bring to this collaboration environment, via both 1:1 conversations, or when added as part of a channel conversation.  A great bot on Teams will also find ways to leverage the unique Tabs feature, via a [configurable channel tab](tabs.md), or a [static bot tab](statictab.md).
- [Create and register your bot in the Bot Framework](botscreate.md):  Take advantage of the great tools, documentation, and community provided by the Bot Framework team.
- [Develop your bot](botsconversation.md): Add basic conversation flow and leverage channel-specific functionality. 
- Test your bot:  [Add your bot](botsadd.md) for 1:1 or team conversations to see it in action.
- Publish your bot: Create your Teams package, adding other capabilities, and submit it to the Office Store.

## Overview of building a Custom bot

Custom bots allow you to create a simple bot for basic interaction, like kicking off a workflow or other simple commands you may need.  These Custom Bots live only in the team in which you create them, and are intended for simple processes specific to your and your company's workflow.  See [here](custombot.md) for more information.