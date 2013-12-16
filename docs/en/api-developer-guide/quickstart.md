﻿# Quick start

This section describes in detail the creation of a simple Android app featuring voice control using *Everyday Assistant* API.

## Required qualification

In order to perform the operations below, you need to have Java programming skills, to know how to use [Android SDK](http://developer.android.com/develop/index.html); an IDE for developing Android apps is also required.

## What does this example offer

In this example, we will develop an app that can make acquaintance with the user, memorize his/her name and greet him every time using the stored name.

These functions can be implemented as a separate Android app or built in any existing app.

## Creating a new project

Create a new Android app project in your IDE or use an existing project. We will have to create several source code files and request syntax description files.

We suggest that you use the last [Android Studio](http://developer.android.com/sdk/installing/studio.html) and [Gradle](http://www.gradle.org/) version. For the new project you have to write the following in the `build.gradle` file of the project:

	repositories {
	    maven {
	        url 'http://voiceassistant.mobi/m2/repository'
	    }
	}

	dependencies {
	    compile 'mobi.voiceassistant:client:0.1.0-SNAPSHOT'
	    compile 'mobi.voiceassistant:base:0.1.0-SNAPSHOT'
	}

This will set the dependencies on two assistant libraries that your app will use.

*Please mind the SNAPSHOT postfix, it indicates that this is not the final version of the library. There will be many updates in the near term*

### Creating request syntax

Create a hello.xml file in the XML directory of your project. In this file we will describe the [request syntax](grammarsyntax.html), which allows to convert user speech into commands that your app understands.

> #### How speech is converted into commands
> *Everyday assistant* uses speech recognition technology, which converts user speech into written text. Then the assistant has to "understand" (based on the text), to which app does it refer, what command and in what context should be executed, and then he has to extract from the text all data required to execute the command.
> 
> To ensure the foregoing, we have to include in the request syntax the [patterns](patternsyntax.html) describing user sentences, to which the app must respond, and the data that must be extracted from these sentences in order to execute certain commands.

Please writhe following in the hello.xml file:

	<?xml version="1.0" encoding="utf-8"?>

	<module xmlns:android="http://schemas.android.com/apk/res/android">

    	<command android:id="@+id/cmd_hello">
        	<pattern value="hello* *"/>
    	</command>
    
	</module>
	
For the time being, we have described but one command featuring the ID `cmd_hello` and a single pattern. This pattern will be actuated by the phrases such as "Hello", "Hello there" etc., that is, any phrases starting with "hello" regardless of its ending and of the number of subsequent words.

### Creating an agent

[Agent](agents.html) is a program interface between the assistant and your app domain logic. Fundamentally, it is an add-in for standard Android services containing special methods for managing the dialog with the user.

In order to create the agent we have to write a class implementing the [`AssistantAgent`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/AssistantAgent.html) abstraction:

	public class HelloAgent extends AssistantAgent {
    	@Override
    	protected void onCommand(Request request) {
    	}
	}

Every agent must redefine at least one method, [`onCommand`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onCommand(mobi.voiceassistant.base.Request)), which is the entry point for the assistant. The assistant sends to this method a special [`Request`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html) data container, which includes all necessary information to process user request.

	@Override
   	protected void onCommand(Request request) {
        switch (request.getDispatchId()) {
            case R.id.cmd_hello:
                onHello(request);
                break;
        }
    }

    private void onHello(Request request) {
    }

Here we schedule our only command with `cmd_hello` ID to `onHello` method; later it will implement the logic of greeting and getting to know the user. For this purpose we will use the [`getDispatchId`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html#getDispatchId()) method of the [`Request`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html) class, which returns the command ID as an integer.

> All XML files under Android are compiled, so we can use the `R` class to store unique integer command IDs.

### Generating an answer

The agent must return at least one answer to every user request. To ensure this, we have to create the answer content and add it to the request using [`addResponse`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html#addResponse(mobi.voiceassistant.base.Response)) or [`addQuickResponse`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html#addQuickResponse(java.lang.Object)) methods.

> The assistant logs user requests. So every answer should be connected to a request, which determines the use of [`Request`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html) class methods for the generation of answer to the user.

Now let's create a basic answer just to greet the user. It will be a plain text string containing one word - "Hello". It's better to write it in the `values/strings.xml` file.

	<resources>
    	<string name="hello_hello">Hello</string>
	</resources>

Then the `onHello` method can be implemented as follows^

	private void onHello(Request request) {
		request.addQuickResponse(getString(R.string.hello_hello));
    }
    
Here we use the [`addQuickResponse`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Request.html#addQuickResponse(java.lang.Object)) method to generate a quick answer by sending a plain text string as answer content. This text will be displayed in the assistant interface as a text "[bubble](bubbles.html)"; if the user has activated a TTS service, the assistant will articulate it.

### Registering the agent and launching the app

In order to make the assistant detect the agent and load its main module, we have to describe it in your app's manifest AndroidManifest.xml

	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    	package="com.example.quickstart"
    	android:versionCode="1"
    	android:versionName="1.0">

    	<uses-sdk android:minSdkVersion="9" android:targetSdkVersion="18" />

    	<application android:label="@string/app_name"
        	android:icon="@drawable/ic_launcher">

        	<service android:name=".HelloAgent">
            
            	<intent-filter>
                	<action android:name="mobi.voiceassistant.intent.action.COMMAND"/>
                	<data android:scheme="assist" android:host="mobi.voiceassistant.en"/>
            	</intent-filter>
            
            	<meta-data android:name="mobi.voiceassistant.MODULE" android:resource="@xml/hello"/>
            
        	</service>
    	</application>

	</manifest>

As you can see, the agent is a normal service. In `mobi.voiceassistant.MODULE` metadata we define the main module to be loaded automatically by the assistant. We must also describe the `mobi.voiceassistant.intent.action.COMMAND` action in the agent's filter and to specify under `data` tag the particular "Everyday Assistant" app that your agent will call. In this case, it goes about the English assistant version (`mobi.voiceassistant.en` package). For more information about the agent registration, see [special section](agents.html).

Now we can compile our app using standard build methods for Android apps and launch it on any device where "Everyday Assistant" is installed.

When the app is launched, "Everyday Assistant" will detect it and load its main module. Now any relevant phrases will be scheduled to our `HelloAgent` agent, and the user will see an answer in the shape of a featuring the text "Hello".

> As you can see, our test app doesn't contain any activity and isn't displayed in the menu. This doesn't mean that you may not use activities, it just means that we don't need any in this particular case.

After the first request will be completed, you can connect a standard debugger to your app process.

## Managing the dialog

Now we will implement a more complex logic, which includes switching the dialog context, storing the user name and greeting him by name.

When the user says "Hello" while the app doesn't know his name yet, we should ask the user to give his name. Anything that the user answers to this question will be interpreted as a name. After the name is received, the app can store it in some data storage (in this case, we will use SharedPreferences).

### Modal mode

To implement this scenario, we need to manage the dialog, namely to enter the modal mode. To learn more about the dialog modes and their use, see [special section](scopes.html). For the moment we just have to know that the modal mode lets us restrict the number of commands available to the user by temporarily hiding all the other commands. This is necessary to let our agent intercept any string and interpret it as user name (there are all sorts of names, so it is impossible to describe all names in a pattern).

Create one more XML file in the XML directory of your app. The file called `name.xml` should contain the following:

	<?xml version="1.0" encoding="utf-8"?>

	<module xmlns:android="http://schemas.android.com/apk/res/android">

    	<pattern name="UserName" value="*"/>

    	<command android:id="@+id/cmd_name">
        	<pattern value="[my name is] $UserName"/>
    	</command>
    
	</module>

Here we have described a simple pattern called `UserName`, which will intercept any string of any length, and one command with a pattern for phrases that can start with "My name is" and end with the user name.

> Square brackets mean that "My name is" string is not necessary, i.e. may be missing in the phrase. In other words, the user can just say his name directly, but the pattern will still work.

Now we can switch the dialog context in `HelloAgent` agent to modal mode using this only module, if need to know the user name. The user will have to say his name if he doesn't use "Cancel" to avoid answering this question.

	private void onHello(Request request) {
        final SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(this);
        final String userName = preferences.getString(PREF_NAME, null);
        
        if(userName == null) {
            final Response response = request.createResponse();
            response.setContent(getString(R.string.hello_say_name));
            response.enterModalQuestionScope(R.xml.name);
            request.addResponse(response);
        } else {
            request.addQuickResponse(getString(R.string.hello_hello, userName));
        }
    }

Now while greeting, the app first looks into SharedPreferences and tries to get the stored user name. If it is found there, the app just greets 
the user by name. If not, the dialog is switched to modal mode, and the app asks the user a question.

The `values/strings.xml` file will now look as follows:

	<resources>
    	<string name="app_name">AssistantQuickStart</string>
    	<string name="hello_hello">Hello, %1$s!</string>
    	<string name="hello_say_name">Hello. What's your name?</string>
	</resources>

> The `enterModalQuestionScope` method won't just switch the assistant to modal mode but also switches the microphone on automatically after the assistant asks his question.

Now the user has to tell his name or say "Cancel" to leave the modal mode. Otherwise, our agent will interpret any user phrase as a name, as the `cmd_name` ID command will be activated.

### Processing the request in modal mode

An answer in modal mode doesn't differ from a normal answer, except for the fact that the user can leave the modal mode using "Cancel" command or answer something that cannot be processed based on the defined module patterns. In this particular case the second option is not relevant, as any phrase will be interpreted as a user name; still we need to respond to the cancellation in some way.

To do this, the agent has to redefine the [`onModalCancel`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onModalCancel(mobi.voiceassistant.base.Request)) method, which must include some suitable content to respond to user command. In our case we will return a plain string containing "Bye" text.

	@Override
    protected void onModalCancel(Request request) {
        request.addQuickResponse(getString(R.string.hello_cancel));
    }

In case of a normal user answer, we will get a normal command that can be processed using [`onCommand`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onCommand(mobi.voiceassistant.base.Request)) method

	@Override
    protected void onCommand(Request request) {
        switch (request.getDispatchId()) {
            case R.id.cmd_hello:
                onHello(request);
                break;
            case R.id.cmd_name:
                onName(request);
                break;
        }
    }
    
    private void onName(Request request) {
    }

Using `onName` method, we can get user name from the request and save it in SharedPreferences; later we will be able to greet the user using this name

	private void onName(Request request) {
        final Token token = request.getContent();
        final Token nameToken = token.findTokenByName(TOKEN_NAME);
        final String userName = nameToken.getSource();

        if(userName.length() == 0) {
            final Response response = request.createResponse();
            response.setContent(getString(R.string.hello_say_again));
            response.enterModalQuestionScope(R.xml.name);
            request.addResponse(response);
            return;
        }

        final StringBuilder sb = new StringBuilder(userName);
        sb.setCharAt(0, Character.toUpperCase(userName.charAt(0)));
        final String displayName = sb.toString();

        final SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(this);
        preferences.edit().putString(PREF_NAME, displayName).commit();
        
        onHello(request);
    }
    
In this case, a voice interaction with the user is concerned, so the request content is represented by a [token](token.html), i.e., the semantic tree of phrase parsing. This token contains a child token called `UserName` that we can get using [`findTokenByName`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#findTokenByName(java.lang.String)) method.

`UserName` token contains plain text in `source` field that we can interpret as a name. As we have already defined the `UserName` pattern as `*`, it means that the text string can also be of zero length (e.g., if the user says just "My name is" and nothing more). To process this case, we have to check the token string length and then return the assistant into modal mode if the string is empty - then the assistant will ask the user to repeat the name.

If the user has told his name, we can store it in SharedPreferences and generate an answer.

> It is very important to remember that the assistant won't switch to modal mode again after the command or the [`onModalFail`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onModalFail(mobi.voiceassistant.base.Request)) method is processed.

`values/strings.xml` file will contain following strings:

	<resources>
    	<string name="app_name">AssistantQuickStart</string>
    	<string name="hello_hello">Hello, %1$s!</string>
    	<string name="hello_say_name">Hello. What's your name?</string>
    	<string name="hello_say_again">Sorry, I don't understand. What's your name?</string>
    	<string name="hello_cancel">Bye</string>
	</resources>

Now you can compile the app again and install it on your device. Then "Everyday Assistant" will reload the module, and you can use it straight off.

## Unconventional articulation of the answer

As you may have remarked, assistant TTS tries to use the tone suitable for the text returned as answer. Still the tone may not be suitable in some cases. E.g., it would be correct syntactically to separate the word "Hello" from the user name with a comma. On the other hand, in normal speech we don't make a pause between "Hello" and the name. 

So you should define another articulation for the greeting, which doesn't feature a pause. Let's include in the `values/strings.xml` file one more string for articulation.

	<string name="speech_hello">hello %1$s</string>
	
In this string, we have omitted the comma. The assistant will articulate this string in a more correct way.

To use this pronunciation, we must use the [`SpeechTextUtils`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/content/SpeechTextUtils.html) utility class of the API:

	private void onHello(Request request) {
        final SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(this);
        final String userName = preferences.getString(PREF_NAME, null);

        if(userName == null) {
            final Response response = request.createResponse();
            response.setContent(getString(R.string.hello_say_name));
            response.enterModalQuestionScope(R.xml.name);
            request.addResponse(response);
        } else {
            final CharSequence content = SpeechTextUtils.textWithSpeech(getString(R.string.hello_hello, userName), getString(R.string.speech_hello, userName));
            request.addQuickResponse(content);
        }
    }

As you see, we generate the answer as a [CharSequence](http://developer.android.com/reference/java/lang/CharSequence.html), where the bubble text differs from the articulated text.

## Changing your name

As our app stores the user name in [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html), you can just clear the app data through the app manager to change your name. But this is not a fair way. Let's try to change our module syntax, so that the user is able to reintroduce himself to the assistant. In this case you can change the main hello.xml module as follows:

	<?xml version="1.0" encoding="utf-8"?>

	<module xmlns:android="http://schemas.android.com/apk/res/android">

    	<pattern name="UserName" value="*" />

    	<command android:id="@+id/cmd_hello">
        	<pattern value="hello* *"/>
    	</command>

    	<command android:id="@+id/cmd_name">
        	<pattern value="* my name is $UserName"/>
    	</command>

	</module>
	
We have just added the `cmd_name` command and the `UserName` pattern into our main module. Now the agent will be able to process such phrases as "Hello, my name is John" without switching to modal mode and asking additional questions. The agent will just store the new name instead of the old one and greet the user. There's no need to change the code.

> As you may have remarked, the command pattern in the main module differs from the one in the name.xml module. In the new pattern, "My name is" string is required; otherwise, the agent would respond to any phrase that doesn't match any pattern from other agents.

## Conclusion

In this example, we have developed a simple app that can connect to the assistant and establish voice contact with the user. We have learned to change dialog context, generate answers (also with a different articulation) and extract data from user speech.

As one can see from this example, the app doesn't differ from normal Android apps. It can use activities, services, manage data storage and perform any other operations accessible to any Android app.

> In this example, we have used the simplest user communication way and the simplest GUI to display the data. For more information about the development of more complex GUI, please go to [bubbles](bubbles.html) section.

Now we suggest that you read the ["Architecture"](architecture.html) section, which describes all assistant components and user communication ways in detail. When you master these aspects, you will be able to develop much more complex apps for use in real life.