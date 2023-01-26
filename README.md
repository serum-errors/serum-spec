Serum Errors
============

"Serum" is a simple standard for errors and error serialization.

Serum aims to be a common-sense, "just enough" standard -- easy to adopt, easy to extend, easy to describe.
It specifies enough to be meaningful, but not so much that it becomes complicated.

The goals are:
- to have observable, serializable, cross-language errors;
- to have human readable, and also search-engine friendly errors;
- and to promote rigorous error handling, and be amenable to static analysis.

Serum describes a simple format (canonically, in JSON), and a couple of conventions for using it.
(It can be other serial formats too.  JSON is just clear and widely-known.)

In short:

```json
{
  "code": "error-codes-matter",
  "message": "messages matter too",
  "details": {"extensible": "yet simple"}
}
```

You don't need any particular library to use Serum, and you don't need to be in any particular programming language.
It's easy to conform to Serum, with or without library support.

Keep reading for more details, below.



How Errors Should Be
--------------------

Errors should be both minimal in their essense, and able to carry rich data as necessary.

Serum starts with a very minimal set of principles.
From these, we derive the serial specification;
and then with that in mind, can begin to recommend library designs and usage conventions.

You can continue scrolling down to go the "principles" route,
or if you're hands-on type, [jump to serialization examples](#serum-serialization)
or jump to [information about libraries you can use to easily adopt Serum](#working-with-serum)


Principles
----------

### Part I

Errors should have a string "code".

This derives from the first three principles:

- 0. Errors are Values.
- 1. It is important to be able to handle errors programmatically, and exhaustively: for this we will need errors to have some kind of "code" or "tag".
- 2. It is important for the error "code" to be textual, so that it can be serialized, and so that programmers can make them rich and unique (as opposed to integers, which would be prone to colision).

(This is an admonition not confuse the typesystem in one language with a general, cross-program way of reasoning about errors, if you're working in a strongly typed language.
You can use both; but the presence of a typesystem does not absolve the need to specify how these values become serialized.)

### Part II

Errors should be serializable -- and deserializable.

This (and the specific schema we will recommend, later) derives from the next seven principles:

- 3. For serialization and deserialization to be easy and realiable, it is important to specify a minimal but standard structure.
- 4. Most critically, the code must be serializable and deserializable, without loss.  (This is easy, because it is already specified to be a simple string.)
- 5. It is useful for errors to describe themselves in a human-readable way.  Therefore, errors should be able to store a freetext message, and if present this should contain prose that is reasonable to show to a user.
- 6. It is useful for errors to be able to attach details: key-value freetext pairs are sufficient for this.
- 7. It is too much to expect that string templating will be available in the same way in every language, so message text is free to repeat parts of the details entries.  (The message is for humans; the details entries are for machines.)
- 8. It is too much to expect that anything at all can be standardized except codes.  For example, it is not reasonable to expect stack traces can be standardized, or even that they are always desirable.  If data such as this are present, they should be placed in a detail entry as freetext, like anything else -- not blessed.
- 9. It is typical for systems and developers to consider an error to have a "cause"; therefore, it should be possible to attach another errors as the "cause" of any error.

Jump down to the [Serum Serialization](#serum-serialization) section to see how these principles become applied.

### Part III

Error handling should be easy to model and analyze.

- 10. It should be possible to model what errors have been handled or not handled with basic set operations.  It should be simple and logical to determine how many distinct error codes need to be considered at any point in a program.
- 11. Error handling should have logical branches *only* determined their Serum code.  Any other advisory information attached may be used as part of responses, but should not determine the logical flow of any response.
- 12. In keeping with the goal of set logic: It is *not* advisable to use the concept of inheritance hierarchies for errors.  Simple sets of codes (and handling more complex error paths by re-tagging) are sufficient, and easier to reason about as sets, and so should be preferred.

These principles do not have considerable effect on the serial format of Serum errors,
but add further support the thesis of simple string codes (e.g. inheritance or more complex models not necessary),
and will also influence how libraries tend to support operations.

### "Unix Style"

Holistically speaking: If you're a fan of "the unix philosophy" of small programs communicating using textual APIs, you should find Serum errors appealing.



Working With Serum
------------------

The Serum errors specification is meant to be so easy that you can easily work with it in any language,
even without special support or complicated libraries.

However, some langauges do have some libraries you can use as a starting point.
There are also some analysis tools that can be put to work to ensure good error handling.

For languages that don't yet have premade libraries, see the ["In General" section](#in-general).

### Golang

- Libraries:
	- You can use https://github.com/serum-errors/go-serum for premade types that comply nicely with the Serum spec.
	- You can also write your own!
- Tools:
	- See https://github.com/serum-errors/go-serum-analyzer for a static analyzer which verifies Serum errors are propagated, handled, and documented accurately.
		- Analysis uses a taint model: all possible codes that a function can return are determined; then the analyzer requires that the documentation of the function states the same set, or emits warnings about the difference.
		- Analysis is mostly automatic and you control it by structuring your code flow to handle errors.  In cases where your code is more complex than the analyzer can understand, you can manually declare changes to the expected error set on return sites, if necessary.
		- Works with any `error` type that has a `Code() string` method -- use of the official go-serum library is in no way required!

### In General

#### In Languages With Error Inheritance

Serum disavows error inheritance.

Inheritance heirarchies do not make code easier to reason about;
tend to result in APIs which throw the broadest types of errors possible (which does not make good handling easy);
and requires coordination to agree on the hierarchy (which is a nonstarter for Serum's cross-program and cross-language intent).
For all of these reasons, Serum errors explicitly do not use or allude to inheritance or hierachies.

This may seem to run against how errors work in some langauges (like Java or C#, for example) which have built in error checking, but base it around inheritance hierarchies.
This doesn't mean you can't adopt Serum error conventions in those languages, though.
You can still create a 1:1 mapping of types to Serum errors, if desired, and continue to use the language's built in error checking facilities.
Or, one can build a Serum error catch-all type (like we did in Golang), and build static analysis tools which understand that instead.

The approach you use internal to a program is up to you; it's mostly a matter of how it will compose with other libraries in your ecosystem,
and whether it helps you when it comes to documenting the behaviors of any serial APIs which other folks may want to expect Serum conventions from.



Serum Codes
-----------

Serum error codes should be reasonably human readable, reasonably unique,
and should not use whitespace or other characters that could make them require quoting or escaping.

We further recommend that conventional Serum error codes should be defined in ASCII text (`[0-9a-zA-Z]`),
and typically use several hunks of text which are joined by dash characters ("`-`").
By convention, lowercase should be preferred.

Tools working with Serum error codes may rely on whitespace or other punctuation characters to delimiters,
and so using characters outside of the recommended ranges make create considerable problems and should be avoided.

Additional typical conventions include:

- The first hunk of an error code (e.g., before the first "-") should be a package name or application name,
  or other reasonably unique and contextually relevant string.
- It is typical, but not required, to put the word "error" as a second hunk,
  especially if the other hunks in the code would not make it clear that string is describing a problematic situation.
- The last hunk(s) should describe the specific error.
- More intermediate hunks can exist as desired.

(This concept of "hunks" is meant to to encourage uniqueness,
while also acknowledging typical logical grouping patterns that are likely to emerge,
but are recommendations only.)

Here are some example error code strings which follow the convention:

```text
libzow-error-needconfig
libzow-error-frobnozfailed
wowapp-error-unknown
wowapp-error-subsys-bonked
wowapp-error-subsys-zonked
wowapp-error-othersys-storagecorrupted
```



Serum Serialization
-------------------

Serum errors, at the minimal form, should be serialized according to this schema:

```ipldsch
type Error struct {
	code    String
} representation map
```

In other words, we would serialize this in JSON as:

``` json
{"code": "your-error-code-here"}
```

At the most rich, using all of the possible data, Serum errors should be serialized according to this schema:

```ipldsch
type Error struct {
	code    String
	message optional String
	details optional {String:String}
	cause   optional [Error]
} representation map
```

A very rich error using all of these fields would serialize in JSON as:

``` json
{
	"code": "your-error-code-here",
	"message": "this is the full error code including all of its details, such as foo=bar and baz=quux",
	"details": {
		"foo": "bar",
		"baz": "quux"
	},
	"cause": [
		{"code": "some-nested-error"}
	]
}
```

Jump back up to the [Serum Principles: Part II](#part-ii) section to see how this structure was designed.


Recommendations for Printing
----------------------------

Errors often need to be printed in a human-readable way.

(Printing JSON is fine and dandy, and is certainly _complete_,
but can be excessively verbose and somewhat user-hostile,
so printing JSON shouldn't be our only option.)

Serum doens't formally specify a printing format,
and programs are free to render errors however they see fit when presenting errors to humans.
(Serum only cares about APIs.)

However, for the common scenario where a program will render an error as simple printed text,
the form that we recommend for Serum, simply for the sake of consistency, is:

- if only `code` is present: `{{code}}`.
- if `message` is present: `{{code}}: {{message}}`.
- if `cause` is present, and there's one of them: `{{code}}: {{cause}}` (apply the printing function recursively).
- if `message` and `cause` are both present: `{{code}}: {{message}}: {{cause}}`
- if multiple values are in `cause`: this is undefined.  Consider emitting a list of codes, or just prose about details being elided.  Avoid overwhelming the user (or consider just showing JSON at this point).

Why like this?

- the code is always present, and is the clearest thing (think: end users should definitely be copy the error code into search queries!), so it absolutely must be visible.
- it is not necessary to say "error " as a prefix, because codes will usually say that word already (and it would create stuttering when printing cause chains).
- the message is already prose: there's nothing more that needs to be done with it.
- other errors in the cause can just repeat these patterns.
- the details map is optional to print, and can be ommitted, because the message prose should already contain the same information (in the prose, contextualized).
	- remember: the goal when rendering is to produce something human readable, not to emphasize machine parsing, because we still have JSON for when we want machine parsing!

