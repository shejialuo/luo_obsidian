# EH Forwarder Bot

EH Forwarder Bot is an extensible framework that allows user to control and manage accounts from different chat platforms in a unified interface. It consists 4 parts:

1. A Master Channel. The channel that directly interact with the User. It is guaranteed to have one and only one master channel in an EFB session. ^2b0b56
2. Some Slave Channels. The channel that delivers messages to and from their relative platform. There is at lease one slave channel in an EFB session. ^d26159
3. Some Middlewares. Module that processes messages and statuses delivered between channels, and make modifications where needed.
4. A Coordinator. Component of the framework that maintains the instances of channels, and delivers messages between channels. ^95df96

![EH Forwarder Bot](https://ehforwarderbot.readthedocs.io/en/latest/_images/EFB-docs-0.png)

## Chat

There are three kinds of chat in EFB:

+ Private chat. A conversation with a single person on the IM platform. Messages from a private conversation shall has an author of the user, the other person, or a "system member".
+ Group chat. A chat that involves more than two members. A group chat MUST provide a list of members that is involved in the conversation.
+ System chat. A chat that is a part of the system. Usually used for chats that are either a part of the IM platform.

## Chat Member

Chat member is a participant of a chat. It can be the user, another person or bot in the chat, or a virtual one created by the IM platform.

## Message

Messages are delivered strictly between the master channel and a slave channel. It usually carries an information of a certain type.

Each message should at least have a unique ID that is distinct within the slave channel related to it. Any edited message should be able to be identified with the same unique ID.

## Status

Status is an information that is not formatted into a message. It usually includes updates of chats and members of chats, and removal of messages.

## Slave Channels

The job of slave channels is relatively simple.

1. Deliver messages to and from the master channel.
2. Maintain a list of all available chats, and group members.
3. Monitor changes of chats and notify the master channel.

Features that does not fit into the standard EFB Slave Channel model can be offered as additional features.

## Master Channel

Master channel is relatively more complicated and also more flexible. As it directly faces the User, its user interface should be user-friendly, or at least friendly to the targeted users.

The job of the master channel includes:

1. Receive, process and display messages from slave channels.
2. Display a full list of chats from all slave channels.
3. Offer an interface for the User to use “extra functions” from slave channels.
4. Process updates from slave channels.
5. Provide a user-friendly interface as far as possible.

## Middlewares

Middlewares can monitor and make changes to or nullify messages and statuses delivered between channels. Middlewares are executed in order of registration, one after another. A middleware will always receive the messages processed by the preceding middleware if available. Once a middleware nullify a message or status, the message will not be processed and delivered any further.
