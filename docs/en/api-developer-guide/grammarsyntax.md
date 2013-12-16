﻿# Request grammars

Grammars (modules) describe the rules used by your program to process user phrases, namely to define the executable commands and the execution context at any specific time.

## General

Grammars are normal XML files stored in your project directories.

### Where are grammars created

Grammars are created in your project's `xml` directory. When you need to describe various grammars for several languages, please use the standard Android resources localization procedure, i.e. create the grammar files in the directories called `xml-<language code>`.

### What do grammars contain

Grammars contain a list of commands and [patterns](patternsyntax.html) used by your app. They don't include any code executing any logic. Grammars can use patterns, defined in other grammars (e.g., globally accessible date or time patterns).

## Grammar structure

    <?xml version="1.0" encoding="utf-8"?>
    <module xmlns:android="http://schemas.android.com/apk/res/android">
		<uses-pattern name="..."/>

		<pattern name="..." value="..."/>
		<pattern name="..." uri="..."/>

		<pattern name="...">
			<pattern value="..."/>
		</pattern>

		<command android:id="@+id/...">
			<pattern value="..."/>

			<command android:id="@+id/...">
				<pattern value="..."/>
			</command>
		</command>
	</module>

`module` tag is the root element here. It specifies the `android` scheme.

A module can contain an unlimited number of patterns and commands.

### pattern
---
Describes a pattern for the user phrase. For detailed information about patterns syntax, please see [this section](patternsyntax.html).

Patterns listed directly under the `module` tag are common patterns used by other patterns. Patterns listed under `command` tags are the very patterns used while processing requests for a specific command.

#### `name` attribute

Every pattern can have a name (`name` attribute) which enables you to unambiguously identify the pattern in the module; it is also used for references from other patterns. This name will also be specified in the semantic tree included in the request sent to your app.

#### `value` attribute

A pattern can be specified explicitly using the [patterns syntax](patternsyntax.html) in `value` attribute, or implicitly, using `uri` attribute.

#### `uri` attribute

The pattern will be generated based on the URI defined here. This is either a local [content provider](contentproviders.html) URI featuring the `content://` scheme or a remote pattern (supports `http://` и `ws://` schemes). 

***For the moment, the use of remote patterns is not documented here.***


*A pattern can be specified explicitly via `value` or implicitly via `uri`. You can't use both variants simultaneously for the same pattern!*

#### Patterns nesting

Patterns can be nested. This allows you to specify the alternatives more clearly. E.g., the following pattern

		<pattern name="Pattern">
			<pattern value="one"/>
			<pattern value="two"/>
		</pattern>

is equal to the shorter pattern

		<pattern name="Pattern" value="(one|two)"/>

Another advantage of such nesting is the possibility to specify a name for every nested pattern.

### uses-pattern
---

This tag imports a pattern defined in another module.

#### `name` attribute

The unique name of the pattern to be imported is specified here.

#### List of global patterns

There is a set of global (common) patterns accessible for all apps.

***Number*** for numbers in various formats. This pattern is associated with *NumberConverter* converter that helps to extract integers.

***Date*** for dates. "DateConverter" returns the Date wrapper.

***Time*** for time. "TimeConverter" returns the Time wrapper.

***DateTime*** for date and time. This one is used, when a phrase must contain a nonseparable sequence of date and time. "DateTimeConverter" returns the DateTime wrapper.

***Location*** for geographic location. Includes both absolute (cities) and relative (here, there, etc.) locations. "LocationConverter" returns the location in the shape of Location wrapper.

***Contact*** for contacts from user contact list. "ContactConverter" returns the Contact wrapper.

*For more information about wrappers, please see [this section](converters.html).*

### command
---
Under this tag, you can specify the ID of the command sent to the [agent](agents.html) that processes the requests to your app. At least one pattern is nested in the command; another command can also be nested.

#### `android:id` attribute

Command ID is specified here. It is written either as `@+id/<ID>`, or as `@id/<ID>`. As all XML files are compiled, these IDs will be later accessible via `R.id`.

#### Nested patterns

At least one pattern must be nested in every command. This pattern contains the phrase triggering the command execution. In other words, while processing the text received from ASR the assistant chooses the pattern, which matches this text best, and then delegates the control to the corresponding agent, specifying the command that contains the used pattern.

#### Nested commands

You can nest several child commands into an upper level command (that is, the one described directly in the `module` tag). These commands will be available to the user after the upper level nesting command will work (for more information, please see the [Scopes](scopes.html) section).

E.g., the initial phrase *"Tell me the weather"* can be followed by such phrases as *For tomorrow"* or *"In New York"*, which are meaningless if the initial command is omitted, so they are not available to the user.

Nested commands form automatically expanded areas (scopes), thus creating the dialog context. Certainly, there are means for flexible process control. You should use corresponding API methods and specify the modules (grammars) that will process current user requests. For more information, please see the [Advanced Features](advanced.html) section.

### sample
---

This tag is only used inside the `command` tag. The only `value` attribute must contain a specified user request example to be used for the command.

>Later this value can be used in *Everyday Assistant* UI to display sample phrases that can be said a  given moment.