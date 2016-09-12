# TWINECERY
twinecery combines the grammar-expansion library [Tracery] with the interactive fiction tool [Twine] _(version 1.x)_. If you're not familiar with either one, then you might check out this [Tracery tutorial] and/or this [Twine tutorial] for a crash course.

twinecery has two primary uses.

1. Make authoring JSON files for a Tracery grammar easier and more organized, regardless of where they'll be used.
2. Generate text from Tracery grammars within Twine stories.

If you run into any problems with twinecery, or this guide, or if you just want to share something neat you did with it, feel free to ping me @[mrfb]!

## Table of Contents
1. [Getting Started](#1-getting-started)
2. [Authoring a Grammar](#2-authoring-a-grammar)
    1. [The Anatomy of a Tracery Grammar](#2i-the-anatomy-of-a-tracery-grammar)
    2. [The Anatomy of a Twine Passage](#2ii-the-anatomy-of-a-twine-passage)
    3. [How twinecery Encodes Tracery Grammars](#2iii-how-twinecery-encodes-tracery-grammars)
    4. [Twine Links as Replacement Rules](#2iv-twine-links-as-replacement-rules)
        1. [Mixing Twine Links and Tracery Symbols](#2iva-mixing-twine-links-and-tracery-symbols)
        2. [Modifiers](#2ivb-modifiers)
        3. [Hiding a Link](#2ivc-hiding-a-link)
3. [Using Grammars in Twine](#3-using-grammars-in-twine)
    1. [With a Macro](#3i-with-a-macro)
    2. [With a Function](#3ii-with-a-function)
    3. [With a Link to Another Passage](#3iii-with-a-link-to-another-passage)
    4. [With an Internally-Acting Link](#3iv-with-an-internally-acting-link)
4. [Importing and Exporting Grammars](#4-importing-and-exporting-grammars)
    1. [Importing JSON](#4i-importing-json)
    2. [Importing Corpora](#4ii-importing-corpora)
    3. [Exporting JSON](#4iii-exporting-json)

## 1. GETTING STARTED
First, create a new story in Twine or open an existing .tws file. In the application menu, go to **Story > Special Passages > StoryIncludes**. Add the following to that passage:
```
https://cdn.rawgit.com/mrfb/twinecery/master/twinecery.txt
```

*n.b.* — This will goof things up if you attempt to build your story without an internet connection. To get around this, download twinecery.txt from this repository and include it locally. If you put that file in the same folder as your .tws file, you can change the above line to `./twinecery.txt`.

Second, go to **Story > Special Passages > StoryInit** and add the following:
```
<<traceryInit>>
```

Third, go to **Story > Special Passages > StorySettings** and enable jQuery.
## 2. AUTHORING A GRAMMAR
### 2.i. The Anatomy of a Tracery Grammar
Normally when authoring a grammar to be used in a Tracery application, you write in JSON. A grammar might look like this:
```
{
    "origin": ["I saw a #color# #animal# today."],
    "color": ["#adjective# #primary#", "#primary#"],
    "adjective": ["dark", "bright", "pretty"],
    "primary": ["red", "blue", "pink"],
    "animal": ["cat", "dog", "porpoise"]
}
```

Which outputs things like:
```
I saw a dark red cat today.
I saw a pink dog today.
I saw a pretty blue cat today.
I saw a blue porpoise today.
```

The whole thing constitutes a **grammar**. Each item in the grammar (`origin`, `color`, `adjective`, &c.) is a **symbol**. Each item in a symbol (`red`, `porpoise`, `I saw a #color# #animal# today.`, &c.) is a **rule**.

### 2.ii. The Anatomy of a Twine Passage
Twine passages have three parts: a **title**, a list of **tags**, and **text**. This guide assumes that you'll be using the graphical interface of the Twine application when you're authoring, but for my own convenience I'll be using the plaintext Twee notation to represent passages, which looks like this:
```
:: title [tag1, tag2, ..., tagN]
text

:: teatime at the witch's cottage [swamp]
I went to the grocery store today and got some apples. Would you like a [[snack|killed by a poisoned apple]]?
```

### 2.iii. How twinecery Encodes Tracery Grammars
In twinecery, a passage gets slurped up into a Tracery grammar when you add `grammar` to that passage's list of tags in Twine. The title of that passage becomes the name of the symbol, and each separate line of the text of that passage becomes the set of rules associated with that symbol. Like so:
```
:: adjective [grammar]
dark
bright
pretty
```

### 2.iv. Twine Links as Replacement Rules
In twinecery, when you link to a Twine passage tagged with `grammar`, twinecery will replace that link syntax with Tracery's replacement syntax. So `[[adjective]]` becomes `#adjective#` in the final grammar.

Our example grammar from earlier would look like this:
```
:: origin [grammar]
I saw a [[color]] [[animal]] today.

:: color [grammar]
[[adjective]] [[primary]]
[[primary]]

:: adjective [grammar]
dark
bright
pretty

:: primary [grammar]
red
blue
pink

:: animal [grammar]
cat
dog
porpoise
```

![twinecery example screenshot](http://mrfb.github.com/mrfb.github.io/twinecery-example.png)

Use this feature to organize your symbols/passages in the Twine interface, and understand where links are going and how your grammar is structured.

#### 2.iv.a. Mixing Twine Links and Tracery Symbols
Replacements only happen on passages tagged with `grammar`, so a link to a standard Twine passage that might be in your story like `[[killed by a poisoned apple]]` is unchanged. In standard Tracery applications, this has no meaning, but if you're writing a grammar to be used in a Twine game, you can use this to put links into your grammar when they print out.

#### 2.iv.b. Modifiers
You can use Twine's [setter links] to add modifiers to replacements, so `[[adjective][capitalize]]` becomes `#adjective.capitalize#`.

#### 2.iv.c. Hiding a Link
If you want to call a symbol in your grammar without creating a link to it in Twine (e.g., for a symbol that gets used so often that it would convolute the graph structure if you included it.) you can still just use the standard Tracery syntax. It will still be replaced as per normal, but won't display the connection.

## 3. USING GRAMMARS IN TWINE

### 3.i. With A Macro
If you want to include text from your grammar in a Twine passage, you'll want to use the ``<<trace>>`` macro. (Well, technically it's a [pseudomacro].)

Pass a single argument in quotes to expand a specific symbol, or call it without any arguments to default to the `origin` symbol.
```
<<trace "symbol">>
<<trace>>
```

**Example:**
```
:: the living room [inside]
There's a worn-out couch and a small [[television]] in here. In the window outside, <<trace "window view">>. The hallway leads to [[the kitchen]].

<<if $television eq "on">>The television lights up the room. You watch for a second, and see <<trace "tv">><<endif>>
```

### 3.ii. With a Function
If you need a grammar expansion in an expression of some kind, the trace() function is where it's at. You can use it to save expansions to variables, or as part of print statements, or wherever else you might need it. You can also use it in the JavaScript console for testing and development purposes.

Pass it a string as an argument to expand a particular symbol, or call it without any arguments to default to `origin`.

```
<<set $variable to trace("symbol")>>
<<set $variable to trace()>>
```

This offers more flexibility than Tracery's built-in method of saving values, since you can also use it in tandem with Twine's conditional statements to branch the story text. For example:

```
:: dinner []
<<silently>>
<<if !$fav_meal>><<set $fav_meal to trace("foods")>><<endif>>
<<set $tonights_meal to trace("foods")>>
<<endsilently>>Tonight, dad made <<$tonights_meal>>. <<if $tonights_meal eq $fav_meal>>Yum!<<else>>Yuck!<<endif>>

You can't wait for tomorrow's [[dinner]].
```

### 3.iii. With a Link to Another Passage
You can also use the ``trace`` passage to create a link to a passage in Twine and have the text of that passage be entirely produced by your grammar. You can specify a symbol using a setter link and adjusting the `$symbol` variable, or just link to it normally to default to `origin`.
```
[[trace][$symbol = "some symbol"]]
[[trace]]
```
_n.b.:_ Linebreaks are normally used to indicate a separation in rules, but if you need to include one in a rule you can use `\n` (in something like [Cheap Bots Done Quick]) or `<br>` (in Twine). For example:
```
:: trapped hallway []
To your left is [[a door with a skull painted on it|trace][$symbol = "game over"]]. To your right is [[a door with a happy face painted on it|the boba place]].

:: game over [grammar]
As you grab the handle of the door, it transforms into an awful hellbeast, who devours you in a way which is surprisingly swift yet excruciatingly painful.<br><br>[[epilogue]]<br><br><b>GAME OVER</b>

:: epilogue [grammar]
Your name and legacy are lost to the sands of time.
It's a pity you never obtained the mythical <<$playerQuest>>.
You are briefly mourned by several passersby.
You are mourned for decades by your lover, [[name]].
```
### 3.iv. With an Internally-Acting Link
`<<tracelink>>` is a modification of Leon Arnott's `<<cyclinglink>>` macro. As with the other methods, if you give it a single string it will expand that symbol, or default to the `origin` symbol if no argument is provided.
```
<<tracelink "an arbitrary symbol">>
<<tracelink>>
```

This is very useful for quickly evaluating the output of a symbol, as clicking rerolls the grammar. The macro will attempt to make sure that each time the link is clicked, the content of the link changes, so it always feels to the user like something is changing when they click the link.

As with the original `<<cyclinglink>>`, you can also pass a variable name as an argument, and it will save the currently displayed output of the link to the provided variable. For example:
```
:: wardrobe []
<<set $player to {}>>You sleepily throw some clothes on and look at yourself in the mirror. You're wearing <<tracelink $player.outfit "clothes">>, a <<tracelink $player.head "flamboyant hat">>, and <<tracelink $player.feet "shoes">>.

[[Okay, you look great!|off to school]]

:: off to school []
<<if $player.head = "baby blue bolero">>Someone bursts out laughing as you pass them.<<else>>Several people gasp as you walk by. "Wow, I love that <<$player.head>>!"<<endif>>

You have a feeling today is going to be a [[good day|pop quiz in first period]].
```

When the user clicks on this link, it's kind of like "swiping left" to get a new result and seeing if they're happy with that one.

## 4. IMPORTING AND EXPORTING GRAMMARS
### 4.i. Importing JSON
If you include a JSON file in a passage and tag that passage with `JSON`, it will be appended to the grammar that twinecery builds. If there are any name duplications between an included JSON file and the twinecery grammar, the symbol from the included JSON file will be used.

If you want to bring that JSON into Twine in order to edit it more, you can use the following macro in a passage:
```
<<print tale.JSONtoTwee()>>
```

...and then you can save that to a plaintext file and import it into your .tws file via **File > Import > Twee Source Code** for further editing.

### 4.ii. Importing Corpora
There are lots of cool sources of data out there. For example, [dariusk/corpora]. If you create a passage and tag it `corpus`, twinecery will do some importing for you.

On each line of the corpus passage, include a URL to a data source and path of properties. The goal is to get an array of strings. The URL comes first, and the path is separated by `#` symbols.
```
:: dog [corpus]
https://rawgit.com/dariusk/corpora/master/data/animals/dogs.json#dogs

:: dinosaur [corpus]
https://rawgit.com/dariusk/corpora/master/data/animals/dinosaurs.json#dinosaurs
```
If there are multiple corpuses, they'll be concatenated. The name of the symbol is the name of the passage.
```
:: animal [corpus]
https://rawgit.com/dariusk/corpora/master/data/animals/dogs.json#dogs
https://rawgit.com/dariusk/corpora/master/data/animals/dinosaurs.json#dinosaurs
```
To help with arrays-of-objects, items in the path that are preceded by a `!` will be skimmed for those properties. In the example case below, each item in `crayola.json["colors"]` has a `color` property and a `hex` property. This will make a symbol named `color` consisting of all the values of `color` in crayola.json.
```
:: color [corpus]
https://rawgit.com/dariusk/corpora/master/data/colors/crayola.json#colors#!color
```
...these two formats won't work for gnarlier corpuses, but they're hopefully helpful enough to work for rapid prototyping.
### 4.iii. Exporting JSON
If the grammar you've authored is destined for a non-Twine application (e.g.: a Twitter bot made with [Cheap Bots Done Quick]), you can print out the JSON using the following macro in a passage:
```
<<print tale.story.toHTML()>>
```

Then, just copy-paste the JSON into wherever it needs to go.

[//]: #
   [Tracery]: <http://tracery.io>
   [Tracery tutorial]: <http://www.crystalcodepalace.com/traceryTut.html>
   [Twine]: <http://twinery.org>
   [Twine tutorial]: <http://www.auntiepixelante.com/twine/>
   [setter links]: <https://twinery.org/wiki/link#setting_variables_with_setter-links>
   [Cheap Bots Done Quick]: <http://cheapbotsdonequick.com>
   [pseudomacro]: <https://twinery.org/wiki/macro#pseudo-macros>
   [mrfb]: <http://twitter.com/mrfb>
   [dariusk/corpora]: <https://github.com/dariusk/corpora>