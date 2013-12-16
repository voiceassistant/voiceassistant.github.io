﻿# Patterns syntax

*Everyday Assistant* uses a third-party speech recognition system. After receiving plain text as system output, the assistant has to recognize the meaning of it and to understand what the user wants.

In other words, we need to extract all meaningful parts of the text, as well as the necessary data for the correct request dispatching to one of the apps or assistant's own modules.

To do this, every app must provide at least one *request grammar*, describing user phrases, to which the app shall respond (using *text patterns*), commands to be executed and the execution context, and the data to be extracted from the text and to be used as command input.

Patterns syntax is flexible enough to cover the variations of speech accepted by your app. Patterns are included in the request grammar, described as XML files. For more information about the grammars, please see [specific section](grammarsyntax.html).

## How patterns work

Every pattern is associated with a certain command and contains all possible request wordings in natural language, as well as named placeholders for parameter extraction. These parameters are then sent to the app entry point ([agent](agent.html)) in the shape of a parsing tree (an equivalent of a semantic tree). This tree contains all data defined in the pattern as language-independent entities ([Token](token.html)).

## How to convert a Token into data

To convert the result of phrase parsing ([Token](token.html)) into data of a certain type (e.g., `int` or an instance of some class) we use special [converters](token.html). API features a number of converters that may be used for global patterns, but you must provide new converters for your own patterns.

## Syntax

Patterns syntax is to a large extent similar to the syntax of formal grammars used by speech recognition systems, e.g., [JSGF](http://www.w3.org/TR/jsgf/). Patterns nesting can be as deep as need be.

Following structures are used in the patterns:

### Alternatives

Defines the presence of the text corresponding to one of listed structures or character strings in a given position.

	(first|second|first or second|$Number)

`|` character is used to separate alternative structures. This example states that the text must contain either the word *first*, or the word *second*, or the whole phrase *first or second*, or a number in any format defined by the *Number* pattern.

Alternatives must be put in plain brackets.

### Options

Define an optional presence of some structures on a given position in the text.

	[first|second|first or second|$Number]

Options must be put in square brackets.

### Transpositions

To specify speech variations without writing lots of similar patterns, you can use transpositions in the places where various word order is possible.

	{north direction}

Transpositions must be put in curly brackets. `|` character is not used to separate structures being transposed. You can use plain brackets for this purpose, e.g., when whole phrases including blanks are transposed.

In the above case the pattern will process both *north direction* and *direction north* phrases.

### Garbage

Means that there might be any number of any characters in this position, or it might be empty. `*` character is used for this purpose.

	my name is *

The compulsory *my name is* phrase might be followed by any string of characters, or by nothing at all.

### Stems

`*` character can also be used for changing the word form.

	interest*

This string covers all words beginning with *interest* (interest, interesting, interested, etc.). `*` can be placed in the beginning of the word as well as in the end, or both in the beginning and the end.

### Link to a pattern

You can refer to the patterns that were defined in a grammar or imported. To do this, please specify the pattern name preceded by a `$` character.

	my name is $Name

The compulsory *my name is* phrase must be followed by a compulsory structure defined in *Name* pattern.

### Synonyms

You may define a synonym in the pattern as a link to another pattern. In this case two colon characters are used - `::`.

	$MyNumber::Number

The parsing tree for such pattern will include a *MyNumber* token with a nested *Number* token. It is handy to use such structures to separate patterns by names, when the same patterns are used in the transposition.

### Representation

`:` character means that the structure will be represented by a certain string value

	(one:1|two:2|three:3)

In the parsing results, the defined value will replace the specified word. E.g., if the user says *three*, the pattern tree will contain the *3* value. This allows to abstract the text data from the speech in a specific language. Later these values can be use for conversion in any required format.

*The represented value may not contain any blanks!* 

If you need to define the value of a whole structure instead of a word, you must put the structure in plain brackets. E.g.

	(day after tomorrow):day_after_tomorrow

### Repetitions

A sequence of one or more patterns can be defined using the `repeat` structure

	repeat($Number)

It indicates that one or more numbers must be present in the specified text position. E.g, a phone number will do^ *812 777 12 34*.

## Complex structures

To implement complex structures containing many alternatives, representations and names, it is advisable to decompose a pattern by nesting one pattern into another. For more information about it, please see the description of [request grammar syntax](grammarsyntax.html).