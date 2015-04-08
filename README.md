pyslate - WORK IN PROGRESS, NOT TO BE USED YET
==============================================

A Python library for maintaining grammatically correct translations for multiple
languages.

What is it for?
---------------

As you probably know, there already are quite many i18n libraries for Python, mostly based on Gettext.
The reason I decided to prepare my own library was because I wasn't satisfied with any of them.
I needed full-features library, having similar capabilities as [Rails i18n](http://guides.rubyonrails.org/i18n.html). But it's not just a port.
I included all features I found important, but also many more:
 - i18n of text (tag values) based on their unique names (tag keys)
 - possibility to use different backends where translations are stored
 - support for special structures to use by translator directly in translation text
 - powerful fallback abilities in case that some variant of tag is missing
 - possibility injecting Python code into translations using decorators and custom functions

Simple example
--------------

Define a translation file 'translations.json':
```json
{
    "hello_world": {
        "en": "Hello world!",
        "pl": "Witaj świecie!"
    }
}
```

Then you can check that it works in an interactive Python session:
```
>>> from pyslate import Pyslate, PyslateJsonBackend
>>> pys_en = Pyslate("en", backend=PyslateJsonBackend("translations.json"))
>>> pys_en.translate("hello_world")
Hello world!
>>> pys_pl = Pyslate("pl", backend=PyslateJsonBackend("translations.json"))
>>> pys_pl.translate("hello_world")
Witaj świecie!
```

It works!

So the most basic use it to create a pyslate object for a selected language and then request translation
of a specific tag using a `Pyslate.translate()` method. To make it more handy you can use `Pyslate.t` abbreviation. The JSON backend is used as an example, there are other storage options available.

More complicated example
------------------------

Change translation file into:
```json
{
    "introduction": {
        "en": "Hello! %{m?His|f?Her} name is %{name}."
    }
}
```
Then in your Python file you can write:
```
>>> pys.t("introduction", name="John", variant="m")
Hello! His name is John.
>>> pys.t("introduction", name="Judy", variant="f")
Hello! Her name is Judy.
```

There are two new things here: `%{name}` is a variable field where actual name (specified as a kwarg for `t()` method) is interpolated.
The second is `%{m?His|f?Her}` structure, called a switch field, which means:
if "variant" kwarg is "m", then print "His", if variant kwarg is "f" then print "Her". If none of these is true, then the first one is used as fallback.
It's easily possible to change pieces of translation based on context variables. That's great for English,
but it's often even more important for [fusional languages](https://en.wikipedia.org/wiki/Fusional_language) (like Polish) where word suffixes can vary in different forms.

Even more complicated example
-----------------------------

Change translation file into:
```json
{
    "show_off": {
        "en": "Hello! I'd like to show you ${toy@article}"
    },
    "toy": {
        "en": "wooden toy"
    }
}
```
Then you can write:
```
>>> pys.t("show_off")
Hello! I'd like to show you a wooden toy.
```

Two new things here: `${}` specifies an inner tag field. It means evaluating a "toy" tag and interpolating the contents directly into the main tag value.
At the end of the inner tag key there's a `@article`. It's a decorator, which means "take the tag value of tag it's used in, and then transform the string into something else".
Decorator "article" is included as specific for English and simply adds a/an article.
There also "upper" "lower" and "capitalize" decorators included right away. In addition, you can define any new decorator as you like.

Combo
-----

```json
{
    "show_off": {
        "en": "Hello! I'd like to show you ${%{toy_name}@article}"
    },
    "horse": {
        "en": "rocking horse"
    }
}
```
Then you can write:
```
>>> pys.t("show_off", toy_name="horse")
Hello! I'd like to show you a rocking horse.
```

How does it work? It's simply evaluating `%{toy_name}` variable field into "horse", which produces `%{horse@article}` inner tag field,
which is evaluated to "rocking horse" which is decorated using `article`, and in the end we get "a rocking horse".

Grammatical forms
-----------------

```json
{
    "announcement": {
        "en": "Hello! ${pol:%{policeperson}@article@capitalize} is here. %{pol:m?He|f?She} is going to help us.",
    },
    "john": {
        "en": ["policeman", "m"]
    },
    "judy": {
        "en": ["policewoman", "f"]
    }
}
```
Then you can write:
```
>>> pys.t("announcement", policeperson="john")
Hello! A policeman is here. He is going to help us.
```

For "john" key for English in specified JSON string there's a list instead of a single string.
The first element of the list is a value used for this key, the second is a grammatical form.

Another new thing is a "pol" identifier followed by a colon - both in an inner tag and a switch field.
The first is tag's ID, which then can be used to specify some special tag options (which will be explained later),
but it can also be used as identifier of grammatical form which can be used in switch field.
So, in short, "m" form is taken from an inner tag and used in switch field to print "He".
The use-case for such mechanism look quite slim for English, however it's very important in many languages,
where every noun has a grammatical form which can, for example, affect form of adjectives.

