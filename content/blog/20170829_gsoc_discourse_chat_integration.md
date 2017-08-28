+++
title = "Summer of Code with Discourse"
slug = "gsoc_with_discourse"
date = 2017-08-29T12:04:19+01:00
thumbnail = "images/gsoc_17.jpg"
tags = ["discourse"]
description = "Creating a chatroom integration plugin for Discourse, the open source discussion software"
+++

I have spent this summer working on the [Discourse](https://discourse.org) project as part of [Google Summer of Code](https://summerofcode.withgoogle.com/). I created the [discourse-chat-integration](https://meta.discourse.org/t/chatroom-integration-plugin-discourse-chat-integration/66522) plugin.

The Discourse chatroom integration plugin allows sending notifications about new Discourse posts to ‘group chats’ on a large number of instant messaging platforms including Slack, Telegram, Discord, HipChat, Mattermost, Matrix, Zulip and Rocket Chat. Notifications can be triggered by new topics, new replies, messages to a group, or mentions of a group. 

This post will provide a brief summary of my work, and the new functionality it provides.

## The Application
I have been involved in the Discourse project since October 2016, so when their participation in Google Summer of Code was announced early this year, I was keen to get involved. After looking through the list of ideas, I settled on the "Common Event System for Chatrooms" project. I had previous relevent experience in this area, having created the [discourse-telegram-notifications](https://meta.discourse.org/t/telegram-notifications-plugin/60483) plugin.

I started by expanding the idea into a formal specification, and [shared it with the community](https://meta.discourse.org/t/common-event-system-for-chatrooms-a-specification/59245) on Discourse Meta. Following comments from the community and the core team, this was adapted (a lot!) into the specification for a new plugin: "discourse-chat-integration". Once discussion on the specification had settled, I submitted my GSOC application for the project.

## The Project

The first thing I did was create a detailed description of my plan, along with some user interface mockups. This was great, as it prompted some discussion about the best way to structure the code, before I'd even opened the editor! The core team, and the community on Meta, were definitely the most useful resources throughout the project.

Communication about the GSOC projects was mostly handled in a dedicated GSOC category on the Discourse Meta forum. This was a great way to share progress and get help from the core team and other students. Weekly progress updates helped to give some structure to the work. I formatted my updates so that I could use them as a to-do list as well:

>
### Week # Update
- What I did this week
- Tasks for next week
- Other to-do items
- Questions for the team

Discourse already had a Slack integration plugin, which provided a starting point for my work. Thanks to this, within the first week I had a working prototype of my plugin. Initially it only supported Slack, but was structured in a way that new providers could be "bolted on" easily. 

Throughout the next 3 months, I continued to adapt and improve the plugin, adding support for new providers throughout. The plugin has a full suite of backend (rspec) tests, frontend (qunit) tests, and passes all of Discourse's source code linting. 

The project required me to work with APIs from many other organisations. After reading their API documentation, I contacted some provider's development communities for more information. It was great to get a peek into the communities of each of these systems, and in one case to submit a [pull request](https://github.com/zulip/zulip/pull/6201) to their repository.

Throughout the project I took some time to work on smaller, but relevent, tasks for Discourse. These included:

- [Adding QUnit tests for core plugins to Discourse's Travis build](https://github.com/discourse/discourse/pull/4971)
- [Add functionality for Discourse plugins to run their own continuous integration](https://github.com/discourse/discourse/pull/4973)
- [Using headless Chrome for integration tests](https://dtaylor.uk/blog/2017/08/chrome-qunit/)


## The Result

Commits to the plugin repository made during the Google Summer of Code period can be found [here](https://github.com/discourse/discourse-chat-integration/commits/master?until=20170829). 

In addition to the code, creating a usable and maintainable product requires good documentation. The [main plugin page](https://meta.discourse.org/t/chatroom-integration-plugin-discourse-chat-integration/66522) lists the key features of the plugin, as well as including links to setup instructions for every provider that is supported. 

[Documentation for contributers](https://meta.discourse.org/t/adding-a-new-provider-to-discourse-chat-integration/68156?u=david_taylor) was also written to make it easier for others to improve the plugin in future.

## Conclusion

I have enjoyed being a part of the Discourse community this summer, and hope to continue my involvement into the future. The key things I will take away are:

- The best resource for an open source project is its community
- Don't be afraid to rewrite sections of code from scratch - it's the best way to improve
- Communicate with developers of external tools - they'll probably love to help out

