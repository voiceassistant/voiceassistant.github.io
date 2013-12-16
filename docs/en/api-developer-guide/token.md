﻿# Token

A token is a container of data obtained during user phrase parsing according to the rules described in a pattern.

In other words, a token can be conceived of as a semantic parsing tree. It is a representation of user phrase as an abstraction independent of any specific language.

Token are nested into one another, that's why we call it a tree. When a pattern is actuated, one token is sent to the agent, the root element that contains the whole phrase as well as (normally) the nested tokens.

## Token attributes

Every token must contain in the `source`field a substring of text extracted from the phrase.

There are also optional attributes:

`name` - token name if it is specified in the `name` attribute of the pattern or as a synonym

`value` - displayed value if it is defined in the pattern using `:`

`tokens` - child tokens

## How to use tokens

[`Token`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html) class provides auxiliary methods for finding child tokens using their name or index.

[`findTokenByName`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#findTokenByName(java.lang.String))

[`findTokensByName`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#findTokensByName(java.lang.String))
	
[`getTokensCount`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#getTokensCount())
	
[`getTokenByIndex`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#getTokenByIndex(int))
	
[`isEmpty`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#isEmpty())
	
[`hasToken`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#hasToken(java.lang.String))

 token may contain more than one token with similar names (e.g., if `repeat` structure is used). In this case, [`findTokenByName`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#findTokenByName(java.lang.String)) method will return the first one only, while [`findTokensByName`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#findTokensByName(java.lang.String)) will return all tokens.

All child tokens are numbered from left to right, starting with zero. [`getTokenByIndex`](http://voiceassistant.mobi/reference/mobi/voiceassistant/base/Token.html#getTokenByIndex(int)) method returns tokens based on their index.

## Converters

In fact, a token is a "raw" representation of data retrieved from the text parsing according to a pattern. Sometimes these data are sufficient for token use (`name`, `source` and `value` fields). Still sometimes you may need to convert the token into an instance of a more convenient data type to implement the app logic.

E.g. the phrase *Twenty five* is not suitable for performing any calculations, so you need to convert it into an integer *25*.

To do this, token converters are used. They accept tokens and return instances of the required class.

API contains already necessary converters for the tokens of all [global patterns](grammarsyntax.html). You may need to create new converters for your own tokens.

### Global pattern converters

All converters are implemented as singletons,as a converter is not supposed to store the status. Every converter has a `getInstance` class method that returns a converter instance. This is the recommended way for the implementation of your own converters.

*NumberConverter* accepts the token from the *Number* global pattern as input and returns an *Integer*.

*DateConverter* accepts a token from the *Date* global pattern as input and returns a *Date* wrapper representing the date.

*TimeConverter* accepts a token from the *Time* global pattern as input and returns a *Time* wrapper representing the time.

*DateTimeConverter* accepts a token from the *DateTime* global pattern as input and returns a *DateTime* wrapper representing date and time.

*LocationConverter* accepts a token from the *Location* global pattern as input and returns a *Location* wrapper representing geographical location (either the coordinates or a specific city).

*ContactConverter* accepts a token from the *Contact* global pattern as input and returns a *Contact* wrapper representing a contact from user contact list.

### How to implement your own converter

A converter is just an implementation of the [`TokenConverter`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/converter/TokenConverter.html) parameterized interface. The [`convert`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/converter/TokenConverter.html#convert(mobi.voiceassistant.base.Token)) method of this interface accepts a token as a parameter and returns an object belonging to the corresponding class.

### How to use converters

Generally speaking, before using a converter you must find the required token in the parent token or in the cookies and then send it to the converter's `convert` method as input. Below is an example of this operation:

	Token timeToken = request.getToken().findTokenByName(AssistantAgentContract.Tokens.TOKEN_TIME);
	Time time = TimeConverter.getInstance().convert(timeToken);

In this example we first look for the *Time* token (`AssistantAgentContract.Tokens.TOKEN_TIME`constant), and then send it to `convert` method of [`TimeConverter`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/converter/TimeConverter.html). The output is an object belonging to [`Time`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/model/Time.html) class, providing time information. Next, you can use the required methods of this class in order to use time data in your app logic (e.g., the [`getCalendar`](http://voiceassistant.mobi/reference/mobi/voiceassistant/client/model/Time.html#getCalendar()) method).
