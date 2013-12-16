﻿# Scopes

Scopes are sets of modules currently available to the user. In other words, it's a set of commands that are currently active and can be used.

Switching scopes enables you to manage voice dialog context.

## Scope types

API features 4 scope types covering all required methods of interaction with the user in a voice dialog.

### Root scope

This scope is created by the assistant at the initialization or at the installation of a new app that uses the API. It includes the main modules of all active agents (i.e. the modules specified in `mobi.voiceassistant.MODULE` metadata of the manifest).

Then every agent may extend or restrict this scope by managing the dialog context.

### Automatic

This type is used to extend automatically the root scope. It is described as nested commands in the upper level commands of a module.

	<command android:id="@+id/weather">
		<pattern value="* weather"/>

		<command android:id="@id/weather">
			<pattern value="next week"/>
		</command>
	</command>

In this case, after the user makes a request *"What will be the weather"*, he gets access to the command *"next week"*. It is a kind of context defined question, which is meaningless if the previous one is omitted. It is called automatic because it doesn't require an explicit scope being specified in the code.

> An automatic scope extends the root one. Please note that the commands from the automatic scope have a higher execution priority; that is, they are not compared by score against other root scope commands. This means that when an automatic scope commands with any score is activated, the assistant will delegate the control to the agent, which is currently working with the corresponding module.
>
> After the automatic scope command is executed, it will resume its work.

### Manual

They are similar to automatic scores; the difference is that they are specified directly from the code using [`enterScope`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html#enterScope(int)) and [`enterQuestionScope`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html#enterModalScope(int)) methods of [`Response`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html) class.

The methods accept a module ID instead of a command. This means that such context management will require separate XML definitions for used modules.

> This type of scopes is convenient for dynamic dialog management depending on conditions that arise, when your app works. Note that the activation conditions for manual scope commands are the same as for the automatic scope.

### Modal

This type of scopes is necessary to restrict the current scope to a certain set of commands. Just as the manual scope, the modal one is specified in the software using [`enterModalScope`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html#enterModalScope(int)) and [`enterModalQuestionScope`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html#enterModalQuestionScope(int)) methods of [`Response`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Response.html) class.

Activation of a modal scope means that the user has got access only to the commands included in this scope. At that, the user can at any moment return to the root scope using cancellation commands (such as *"Cancel"* etc.).

If the user chooses a cancellation command, the [`onModalCancel`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onModalCancel(mobi.voiceassistant.base.Request)) method will be called, enabling the generation of a response to cancellation.

If the user gives a command that doesn't correspond to any pattern from the modal scope, the assistant calls the [`onModalFail`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onModalFail(mobi.voiceassistant.base.Request)) agent method.

If a pattern is activated, the standard [`onCommand`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Agent.html#onCommand(mobi.voiceassistant.base.Request)) method is called.

> After executing any of these scenarios, the modal scope stops working, so that you will need to call again `enterModalScope` or `enterModalQuestionScope` method of `Response` class when necessary, in order to enter the modal mode. Therefore the nested commands in the module, which forms the modal scope, are ignored.