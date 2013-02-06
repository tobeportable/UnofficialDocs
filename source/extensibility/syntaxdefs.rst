Syntax Definitions
==================

Syntax definitions make Sublime Text aware of programming and markup languages.
Most noticeably, they work together with colors to provide syntax highlighting.
Syntax definitions define *scopes* in a buffer that divide the text in named
regions. Several editing features in Sublime Text make extensive use of
this fine-grained contextual information.

Essentially, syntax definitions consist of regular expressions used to find
text, and more or less arbitrary, dot-separated strings called *scopes* or *scope
names*. For every occurrence of a given regular expression, Sublime Text gives
the matching text its corresponding *scope name*.

Prerequisites
*************

In order to follow this tutorial, you will need to install AAAPackageDev_, a package
intended to ease the creation of new syntax definitions for Sublime Text. AAAPackageDev
lives on a public Mercurial_ repository at Bitbucket_.

.. _AAAPackageDev: https://bitbucket.org/guillermooo/aaapackagedev
.. _Mercurial: http://mercurial.selenic.com/
.. _Bitbucket: http://bitbucket.org

Download the latest ``.sublime-package`` file and install it as described in
:ref:`installation-of-sublime-packages`.

.. sidebar:: Mercurial and Bitbucket

  Mercurial is a distributed version control system (DVCS). Bitbucket is an
  online service that provides hosting for Mercurial repositories. If you want
  to install Mercurial, there are freely available command-line and graphical
  `clients`_.

  .. _`clients`: http://mercurial.selenic.com/downloads/

File format
***********

Sublime uses `property list`_ files (Plist) to store syntax definitions. Because
editing XML files is a cumbersome task, though, we'll be using JSON_ instead and
converting it to Plist afterwards. This is where the AAAPackageDev package mentioned
above comes in.

.. _property list: http://en.wikipedia.org/wiki/Property_list
.. _JSON: http://en.wikipedia.org/wiki/JSON

.. note::
    If you experience unexpected errors during this tutorial, chances are
    AAAPackageDev is to blame. Don't immediately think your problem is due to a
    bug in Sublime Text.

By all means, do edit the Plist files by hand if you prefer to work in XML, but
keep always in mind the differing needs with regards to escape sequences, etc.

.. _scopes-and-scope-selectors:

Scopes
******

Scopes are a key concept in Sublime Text. Essentially, they are named text
regions in a buffer. They don't do anything by themselves, but Sublime Text peeks
at them when it needs contextual information.

For instance, when you trigger a snippet, Sublime Text checks the scope the snippet's
bound to and looks at the caret's position in the file. If the caret's current
scope matches the snippet's scope selector, Sublime Text fires the snippet off.
Otherwise, nothing happens.

.. sidebar:: Scopes vs Scope Selectors

  There's a slight difference between *scopes* and *scope selectors*: scopes are
  the names defined in a syntax definition, whilst scope selectors are used in
  items like snippets and key bindings to target scopes. When creating a new syntax
  definition, you care about scopes; when you want to constrain a snippet to a
  certain scope, you use a scope selector.

Scopes can be nested to allow for a high degree of granularity. You can drill down
the hierarchy very much like with CSS selectors. For instance, thanks to scope
selectors, you could have a key binding activated only within single quoted strings
in python source code, but not inside single quoted strings in any other language.

Sublime Text implements the idea of scopes from Texmate, a text editor for Mac.
`Textmate's online manual`_ contains further information about scope selectors
that's useful for Sublime Text users too.

.. _`Textmate's online manual`: http://manual.macromates.com/en/


How Syntax Definitions Work
***************************

At their core, syntax definitions are arrays of regular expressions paired with
scope names. Sublime Text will try to match these patterns against a buffer's text
and attach the corresponding scope name to all occurrences. These pairs of regular
expressions and scope names are known as *rules*.

.. XXX: What are those exceptions mentioned below?

Rules are applied in order, one line at a time. Each rule consumes the matched
text region, which will therefore be excluded from the next rule's matching attempt
(save for a few exceptions). In practical terms, this means that you should take
care to go from more specific rules to more general ones when you create a new
syntax definition. Otherwise, a greedy regular expression might swallow parts
you'd like to have styled differently.

Syntax definitions from separate files can be combined, and they can be recursively
applied too.

Your First Syntax Definition
****************************

By way of example, let's create a syntax definition for Sublime Text snippets.
We'll be styling the actual snippet content, not the ``.sublime-snippet`` file.

.. note::
  Since syntax definitions are primarily used to enable syntax highlighting,
  we'll use *to style* as in *to break down a source code file into
  scopes*. Keep in mind, however, that colors are a different thing to syntax
  definitions and that scopes have many more uses besides syntax highlighting.

These are the elements we want to style in a snippet:

    - Variables (``$PARAM1``, ``$USER_NAME``\ …)
    - Simple fields (``$0``, ``$1``\ …)
    - Complex fields with place holders (``${1:Hello}``)
    - Nested fields (``${1:Hello ${2:World}!}``)
    - Escape sequences (``\\$``, ``\\<``\ …)
    - Illegal sequences (``$``, ``<``\ …)

.. note::
    Before continuing, make sure you've installed the AAAPackageDev package
    as explained further above.

Creating A New Syntax Definition
--------------------------------

To create a new syntax definition, follow these steps:

  - Go to **Tools | Packages | Package Development | New Syntax Definition**
  - Save the new file to your ``Packages/User`` folder as ``Sublime Snippets (Raw).JSON-tmLanguage``.

You should now see a file like this::

  { "name": "Syntax Name",
    "scopeName": "source.syntax_name",
    "fileTypes": [""],
    "patterns": [
    ],
    "uuid": "ca03e751-04ef-4330-9a6b-9b99aae1c418"
  }

Let's examine now the key elements.

``uuid``
    Located at the end, this is a unique identifier for this syntax definition.
    Each new syntax definition gets its own uuid. Don't modify them.

``name``
    The name that Sublime Text will display in the syntax definition drop-down list
    Use a short, descriptive name. Typically, you will be using the programming
    language's name you are creating the syntax definition for.

``scopeName``
    The top level scope for this syntax definition. It takes the form
    ``source.<lang_name>`` or ``text.<lang_name>``. For programming languages,
    use ``source``. For markup and everything else, ``text``.

``fileTypes``
    This is a list of file extensions. When opening files of these types,
    Sublime Text will automatically activate this syntax definition for them.

``patterns``
    Container for your patterns.

For our example, fill in the template with the following information::

    {   "name": "Sublime Snippet (Raw)",
        "scopeName": "source.ssraw",
        "fileTypes": ["ssraw"],
        "patterns": [
        ],
        "uuid": "ca03e751-04ef-4330-9a6b-9b99aae1c418"
    }

.. note::
    JSON is a very strict format, so make sure to get all the commas and quotes right.
    If the conversion to Plist fails, take a look at the output panel for more
    information on the error. We'll explain later how to convert a syntax
    definition in JSON to Plist.

Analyzing Patterns
******************

The ``patterns`` array can contain several types of elements. We'll look at some
of them in the following sections. If you want to learn more about patterns,
refer to Textmate's online manual.


.. sidebar:: Regular Expressions' Syntax In Syntax Definitions

  Sublime Text uses Oniguruma_'s syntax for regular expressions in syntax definitions.
  Several existing syntax definitions make use of features supported by this regular
  expression engine that aren't part of perl-style regular expressions, hence the
  requirement for Oniguruma.

  .. _Oniguruma: http://www.geocities.jp/kosako3/oniguruma/doc/RE.txt

Matches
-------

They take this form:

.. code-block:: js

    { "match": "[Mm]y \s+[Rr]egex",
      "name": "string.ssraw",
      "comment": "This comment is optional."
    }

``match``
    A regular expression Sublime Text will use to try and find matches.

``name``
    Name of the scope that should be applied to the occurrences of ``match``.

``comment``
    An optional comment about this pattern.

Let's go back to our example. Make it look like this:

.. code-block:: js

    { "name": "Sublime Snippet (Raw)",
      "scopeName": "source.ssraw",
      "fileTypes": ["ssraw"],
      "patterns": [
      ],
      "uuid": "ca03e751-04ef-4330-9a6b-9b99aae1c418"
    }

That is, make sure the ``patterns`` array is empty.

Now we can begin to add our rules for Sublime snippets. Let's start with simple
fields. These could be matched with a regex like so:

.. code-block:: perl

    \$[0-9]+
    # or...
    \$\d+

However, because we're writing our regex in JSON, we need to factor in JSON's
own escaping rules. Thus, our previous example becomes:

.. code-block:: js

    \\$\\d+

With escaping out of the way, we can build our pattern like this:

.. code-block:: js

    { "match": "\\$\\d+",
      "name": "keyword.source.ssraw",
      "comment": "Tab stops like $1, $2..."
    }

.. sidebar:: Choosing the Right Scope Name

    Naming scopes isn't obvious sometimes. Check the Textmate online manual
    for guidance on scope names. It is important to re-use the basic categories
    outlined there if you want to achieve the highest compatibility with existing
    colors.

    Colors have hardcoded scope names in them. They could not possibly include
    every scope name you can think of, so they target the standard ones plus some
    rarer ones on occasion. This means that two colors using the same syntax
    definition may render the text differently!

    Bear in mind too that you should use the scope name that best suits your
    needs or preferences. It'd be perfectly fine to assign a scope like
    ``constant.numeric`` to anything other than a number if you have a good
    reason to do so.

And we can add it to our syntax definition too:

.. code-block:: js

    {   "name": "Sublime Snippet (Raw)",
        "scopeName": "source.ssraw",
        "fileTypes": ["ssraw"],
        "patterns": [
            { "match": "\\$\\d+",
              "name": "keyword.source.ssraw",
              "comment": "Tab stops like $1, $2..."
            }
        ],
        "uuid": "ca03e751-04ef-4330-9a6b-9b99aae1c418"
    }

We're now ready to convert our file to ``.tmLanguage``. Syntax definitions use
Textmate's ``.tmLanguage`` extension for compatibility reasons. As explained further
above, they are simply XML files in the Plist format.

Follow these steps to perform the conversion:

    - Save file as a ``.JSON-tmLanguage`` file
    - Select ``Automatic`` in **Tools | Build System**
    - Press :kbd:`F7`
    - A ``.tmLanguage`` file will be generated for you in the same folder as your
      ``.JSON-tmLanguage`` file
    - Sublime Text will reload the changes to the syntax definition

You have now created your first syntax definition. Next, open a new file and save
it with the extension ``.ssraw``. The buffer's syntax name should switch to
"Sublime Snippet (Raw)" automatically, and you should get syntax highlighting if
you type ``$1`` or any other simple snippet field.

Let's proceed to creating another rule for environment variables.

.. code-block:: js

    { "match": "\\$[A-Za-z][A-Za-z0-9_]+",
      "name": "keyword.source.ssraw",
      "comment": "Variables like $PARAM1, $TM_SELECTION..."
    }

Repeat the steps above to update the ``.tmLanguage`` file and restart Sublime Text.

Fine Tuning Matches
-------------------

You might have noticed that the entire text in ``$PARAM1``, for instance, is styled
the same way. Depending on your needs or your personal preferences, you may want
the ``$`` to stand out. That's where ``captures`` come in. Using captures,
you can break a pattern down into components to target them individually.

Let's rewrite one of our previous patterns to use ``captures``:

.. code-block:: js

    { "match": "\\$([A-Za-z][A-Za-z0-9_]+)",
      "name": "keyword.ssraw",
      "captures": {
          "1": { "name": "constant.numeric.ssraw" }
       },
      "comment": "Variables like $PARAM1, $TM_SELECTION..."
    }

Captures introduce complexity to your rule, but they are pretty straightforward.
Notice how numbers refer to parenthesized groups left to right. Of course, you can
have as many capture groups as you want.

Arguably, you'd want the other scope to be visually consistent with this one.
Go ahead and change it too.

Begin-End Rules
----------------

Up to now we've been using a simple rule. Although we've seen how to dissect patterns
into smaller components, sometimes you'll want to target a larger portion of your
source code clearly delimited by start and end marks.

Literal strings enclosed in quotation marks and other delimited constructs are
better dealt with with begin-end rules. This is a skeleton for one of these rules::

      { "name": "",
        "begin": "",
        "end": ""
      }

Well, at least in their simplest version. Let's take a look at one including all
available options::

       { "name": "",
         "begin": "",
         "beginCaptures": {
           "0": { "name": "" }
         },
         "end": "",
         "endCaptures": {
           "0": { "name": "" }
         },
         "patterns": [
            {  "name": "",
               "match": ""
                         }
         ],
         "contentName": ""
       }

Some elements may look familiar, but their combination might be daunting. Let's
see them individually.

``begin``
    Regex for the opening mark for this scope.

``end``
    Regex for the end mark for this scope.

``beginCaptures``
    Captures for the ``begin`` marker. They work like captures for simple matches. Optional.

``endCaptures``
    Same as ``beginCaptures`` but for the ``end`` marker. Optional.

``contentName``
    Scope for the whole matched region, from the begin marker to the end marker,
    inclusive. This will effectively create nested scopes for ``beginCaptures``,
    ``endCaptures`` and ``patterns`` defined within this rule. Optional.

``patterns``
    An array of patterns to match against the begin-end content **only** ---they are not
    matched against the text consumed by ``begin`` or ``end``.

We'll use this rule to style nested complex fields in snippets::

    { "name": "variable.complex.ssraw",
       "begin": "(\\$)(\\{)([0-9]+):",
       "beginCaptures": {
           "1": { "name": "keyword.ssraw" },
           "3": { "name": "constant.numeric.ssraw" }
       },
       "patterns": [
           { "include": "$self" },
           {  "name": "string.ssraw",
              "match": "."
           }
       ],
       "end": "\\}"
    }

This is the most complex pattern we'll see in this tutorial. The ``begin`` and ``end``
keys are self-explanatory: they define a region enclosed between ``${<NUMBER>:`` and ``}``.
``beginCaptures`` further divides the begin mark into smaller scopes.

The most interesting part, however, is ``patterns``. Recursion and the
importance of ordering have finally made an appearance here.

We've seen further above that fields can be nested. In order to account for
this, we need to recursively style nested fields. That's what the ``include``
rule does when furnished the ``$self`` value: it recursively applies our entire
syntax definition to the portion of text contained in our begin-end rule, excluding
the text consumed by both ``begin`` and ``end``.

Remember that matched text is consumed and is excluded from the next match
attempt.

To finish off complex fields, we'll style place holders as strings. Since
we've already matched all possible tokens inside a complex field, we can
safely tell Sublime Text to give any remaining text (``.``) a literal string scope.

Final Touches
-------------

Lastly, let's style escape sequences and illegal sequences, and wrap up.

::

        {  "name": "constant.character.escape.ssraw",
           "match": "\\\\(\\$|\\>|\\<)"
        },

        {  "name": "invalid.ssraw",
           "match": "(\\$|\\<|\\>)"
        }

The only hard thing here is getting the number of escape characters right. Other
than that, the rules are pretty straightforward if you're familiar with
regular expressions.

However, you must take care to put the second rule after any others matching
the ``$`` character, since otherwise you may not get the desired result.

Also, note that after adding these two additional rules, our recursive begin-end
rule above keeps working as expected.

At long last, here's the final syntax definition::

  {   "name": "Sublime Snippet (Raw)",
      "scopeName": "source.ssraw",
      "fileTypes": ["ssraw"],
      "patterns": [
          { "match": "\\$(\\d+)",
            "name": "keyword.ssraw",
            "captures": {
                "1": { "name": "constant.numeric.ssraw" }
             },
            "comment": "Tab stops like $1, $2..."
          },

          { "match": "\\$([A-Za-z][A-Za-z0-9_]+)",
            "name": "keyword.ssraw",
            "captures": {
                "1": { "name": "constant.numeric.ssraw" }
             },
            "comment": "Variables like $PARAM1, $TM_SELECTION..."
          },

          { "name": "variable.complex.ssraw",
            "begin": "(\\$)(\\{)([0-9]+):",
            "beginCaptures": {
                "1": { "name": "keyword.ssraw" },
                "3": { "name": "constant.numeric.ssraw" }
             },
             "patterns": [
                { "include": "$self" },
                { "name": "string.ssraw",
                  "match": "."
                }
             ],
             "end": "\\}"
          },

          { "name": "constant.character.escape.ssraw",
            "match": "\\\\(\\$|\\>|\\<)"
          },

          { "name": "invalid.ssraw",
            "match": "(\\$|\\>|\\<)"
          }
      ],
      "uuid": "ca03e751-04ef-4330-9a6b-9b99aae1c418"
  }

There are more available constructs and code reuse techniques, but the above
explanations should get you started with the creation of syntax definitions.
