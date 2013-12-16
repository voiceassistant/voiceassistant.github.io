﻿Request-response
===

After the agent receives a request it is time to formulate a response and send it to the assistant.

Let's start from the end ‧— from the sending. Suppose you have already the response content in the `content` variable. There are 2 ways to send this content to the assistant.

Wordy:

    Response response = request.createResponse();
    response.setContent(content);
    request.addResponse(response);

One string:

    request.addQuickResponse(content);

A one string response won't let you enter the `Scope`, set a `cookie`, but if the response is final without appeal, one string is quite enough.

Response content
===
In order to make the assistant show the content to the user, it must be first delivered to and understood by the assistant. Despite the fact that [`Request.addQuickResponse()`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html#addQuickResponse(java.lang.Object)) and [`Response.setContent()`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html#setContent(java.lang.Object)) methods accept the `Object`, it is impossible to send an arbitrary object. First of all, not all objects can be processed with [`Parcel`](http://developer.android.com/reference/android/os/Parcel.html) (see [`Parcel.writeValue()](http://developer.android.com/reference/android/os/Parcel.html#writeValue(java.lang.Object))). Next, some objects cannot be displayed.

The assistant supports 3 main content types as well as several auxiliary types.

Main:

- [`CharSequence`](http://developer.android.com/reference/java/lang/CharSequence.html)
- [`RemoteViews`](http://developer.android.com/reference/android/widget/RemoteViews.html)
- [`Bubble`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/content/Bubble.html)

Auxiliary:

- [`Utterance`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/content/Utterance.html)
- Massive or ArrayList from any supported types

CharSequence
---
There are but a few things to say about the [`CharSequence`](http://developer.android.com/reference/java/lang/CharSequence.html). It supports styles (see [`TextUtils.writeToParcel`](http://developer.android.com/reference/android/text/TextUtils.html#writeToParcel(java.lang.CharSequence,android.os.Parcel,int)).

RemoteViews
---
There isn't much to say about [`RemoteViews`](http://developer.android.com/reference/android/widget/RemoteViews.html) either. There is only one important difference from appwidgets - it doesn't support lists.

Bubble
---
The most flexible way to formalize the response is to use the [`Bubble`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/content/Bubble.html) class. It is nearly similar to [`RemoteViews`](http://developer.android.com/reference/android/widget/RemoteViews.html), but also includes a number of convenient methods to process button taps.

Below is the example of button tap processing:

    Bubble bubble = new Bubble(R.layout.bubble_mine);
    bubble.makeOnClickRequest(R.id.btn1);

If the user taps a button with the ID equal to `R.id.btn1`, your agent's [`onBubbleClick(Request)`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/AssistantAgent.html#onBubbleClick(mobi.voiceassistant.base.Request)) method вашего will be called, and the [Request.getDispatchId()](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html#getDispatchId()) will be equal to `R.id.btn1`.

It is important to realize that when the tap is processed in this way, a new request will be send, and the agent will have to formulate a new response.

To process the button tap without sending a new request, you must use something like this:

    PendingIntent template = PendingIntent.getActivity(MyAgent.this, 0, new Intent(), 0);
    bubble.setPendingIntentTemplate(template);
    Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://example.com")
    bubble.setOnClickFillInIntent(R.id.btn1, intent);

The browser will be launched.

You can also process the taps on the whole bubble using [`makeOnBubbleClickRequest()`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/content/Bubble.html#makeOnBubbleClickRequest()), [`setOnBubbleClickRequest(PendingRequest.Builder)`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/content/Bubble.html#setOnBubbleClickRequest(mobi.voiceassistant.base.PendingRequest.Builder)) and [`setOnBubbleClickFillInIntent(Intent)`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/content/Bubble.html#setOnBubbleClickFillInIntent(android.content.Intent)) methods.

Utterance
---
The main way of articulating any content is to include it in [`Utterance`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/content/Utterance.html). One and only constructor [`Utterance(Object, CharSequence)`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/content/Utterance.html#Utterance(java.lang.Object,java.lang.CharSequence)) accepts the content and its articulation. No need to say that the restrictions imposed on the response content are also imposed on `Utterance` content.

Massives and lists
---
If the response contains a massive or a list, all of its elements will be arranged vertically while displayed.
