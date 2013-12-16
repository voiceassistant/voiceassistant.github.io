﻿# Content providers

This is an instrument for creating dynamic patterns featuring a fuzzy matching option. It proves necessary, when you can't describe the pattern statically or when the pattern data may change over time. We can mention the contacts from your phone book or the app names as an example of such content.

In the `pattern` tag of `uri` attribute you must specify the Uri with `content://` scheme. It will define the content provider to be used for pattern creation.

> You can only create upper level patterns in this way, that is, patterns described in the module itself, and not in the module commands.

For more information about content providers in Android OS, please see corresponding [documentation](http://developer.android.com/guide/topics/providers/content-providers.html).

## How to implement a content provider

As in standard case, you must describe your content provider in your app manifest (AndroidManifest.xml):

	<provider android:name=".MyProvider"
              android:authorities=".content"
              android:exported="false"/>

In this example, `MyProvider` class must implement an abstract class called `android.content.ContentProvider`. You can omit implementing such methods as `insert`, `delete` and `update`, as the assistant will use the provider solely to load the data for pattern creation.

Below is an example of a pattern using such content provider:

	<pattern name="MyPattern" uri="content://com.example.content/mypattern"/>

> It is loaded every time a user request is received, which corresponds to the command pattern that uses this dynamic pattern.

### Pattern type

`getType` method must return the required pattern type.

- [`mobi.voiceassistant.base.AgentContract.Content.KEY_VALUE_TYPE`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/AgentContract.Content.html#KEY_VALUE_TYPE) means that all elements are simple ones and must be matched "as is"
- [`mobi.voiceassistant.base.AgentContract.Content.FUZZY_KEY_VALUE_TYPE`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/AgentContract.Content.html#FUZZY_KEY_VALUE_TYPE) means that fuzzy search must be used for this pattern matching (e.g. word endings may change)

### query method

This method must return a [`Cursor`](http://developer.android.com/reference/android/database/Cursor.html) that contains key-value pairs for all pattern elements. Usually the [`MatrixCursor`](http://developer.android.com/reference/android/database/MatrixCursor.html) is used for this purpose. Every cursor string must contain two columns only:

- [`mobi.voiceassistant.base.AgentContract.Content.COLUMN_ID`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/AgentContract.Content.html#COLUMN_ID) is the key for a pattern element; it will be returned to `value` field in the [token](token.html)
- [`mobi.voiceassistant.base.AgentContract.Content.COLUMN_VALUE`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/AgentContract.Content.html#COLUMN_VALUE) is the value itself, being matched according to the rule described by the [`getType`](http://developer.android.com/reference/android/content/ContentProvider.html#getType(android.net.Uri)) method of your content provider.

## When to use content providers

You should remember that content providers are loading data every time when the pattern is being matched; it means that only a small amount of data must be loaded, and it shouldn't take much time.

Usually content providers are used to retrieve data stored either locally (databases, [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) etc.), or on some devices in a high-speed intranet network.

## Annotators

In order to use big amounts of data stored on remote servers, you must use an *annotator*; you can specify its URL in the same `uri` attribute. While matching the text, the matcher sends a request to such server, which returns supplementary data concerning the entities found in the string. Therefore, the annotator server is used as a search engine able to process big data volumes and language entities. 

That's how "Cinema" or "Sport" services work in "Everyday Assistant". The servers store all movie titles, theater names or names of sports clubs and athletes, required for these services.

> This documentation version doesn't describe the use of annotators while creating a third-party app. The description of this function will be included in the next versions of API documentation.