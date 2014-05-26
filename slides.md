<!------------------------------------------------------------>
<!-- Topic: xxx -->

# Lord Parsnip

<img src="images/lordparsnip.png" style="border:0px;">

A Combinatoric Parser Library

--SLIDE--

# Background

* me: [github.com/krig][k], [SUSE][s], [kodsnack.se][ks]

* Inspired by Parsec from Haskell

* Core idea: Combine small parsers to build bigger ones

* "Regular expressions meet Lego"

[k]: https://github.com/krig
[s]: https://www.suse.com/
[ks]: http://kodsnack.se

--SLIDE--
## Get and use

```
git clone https://github.com/krig/parsnip

from parsnip import ...
```

--SLIDE--
## Example parser

```
foo = text('foo')
foo(tokens('foo'))
# => 'foo'

foo(tokens('bar'))
# => NoMatch: Input: bar, Expected: foo
```

* `foo()`: A *parser function*.
* `text()`: Function that **returns** a *parser function*.

--SLIDE--
## Combiners

```
foo = many(choice(text('foo'), text('bar')))
foo(tokens('foo bar foo foo bar'))
# => ['foo', 'bar', 'foo', 'foo', 'bar']

foo(tokens('foo bar wiz'))
# => ['foo', 'bar']

foo = seq(foo, end())
foo(tokens('foo bar foo foo bar'))
# NoMatch: Input: foo wiz, Expected: foo <END>
#	Caused by: Input: wiz, Expected: <END>
```

* Simple pieces **combine**
* Re-use smaller parts

--SLIDE--
## Lifting

```
word_colons = many(lift(regex(r'(\w+):')))
# word_colons is a parser function
word_colons(tokens("a: b: c:"))
# => ['a', 'b', 'c']
```

* `regex()` returns a list of match **groups** (parentheses)
* **Lifting**: return one of the matched groups
  * `lift()`, `lift2()`, ...

--SLIDE--
## Error messages
```
malformed = tokens("foo bar wiz bang")
word_colons = many(lift(regex(r'(\w+):', '<word>:')))
word_colons(malformed)
# =>
#   NoMatch: Input: foo, Expected: [<word>: ...]
```

* Second argument to `text()` is like a doc-string

--SLIDE--
## regex lexer

#### Building more complex lexers

```
lexer = regex_lexer(r'\(', r'\)',
                     '{',   '}',
                     ',',
                     '[a-zA-Z_][a-zA-Z0-9_]*',
                    skip=r'[\s\n]+')
```

* More lexers: `tokens_shlex()`, `tokens_gen()`, ...
* `tokens()` tries to guess based on argument

--SLIDE--
## Tagging output

```
name = regex('\w+', '<name>')
arg = regex('[a-zA-z][a-zA-Z0-9_]*', '<arg>')
arglist = lift2(seq(text('('), sep(arg, text(',')), text(')')))
body = lift2(seq(text('{'), text('pass'), text('}')))
fundef = seq(text('def'), tag(name, 'name'),
                          tag(arglist, 'args'),
                          tag(body, 'body'))
funmap = maptags(fundef, lambda tags: tags)
```

--SLIDE--
## Tagging output, continued

```
print fundef.__doc__
# => def <name> ( [<arg> [, <arg>] ...] ) { pass }
print funmap(tokens(lexer("""
def foo(x, y, z) {
    pass
}
""")))
# => {'name': 'foo',
#     'args': ['x', 'y', 'z'],
#     'body': 'pass'}
```

--SLIDE--
## Basic Parsers

* `text(str)` - matches _str_ (case-insensitive)
* `regex(txt, [doc], [flags])` - matches the regular expression _txt_
* `anything()` - match anything
* `end()` - match the end of input
* `doc(parser, text)` - rename _parser_ as _text_

--SLIDE--
## Basic Combinators

* `option(parser)` - optionally matches _parser_
* `seq(*parsers)` - matches an exact sequence of _parsers_
* `many(parser)` - matches _parser_ zero or more times
* `choice(*parsers)` - matches one of _parsers_

--SLIDE--
## Tagging and mapping

* `tag(parser, name)` - store output of _parser_ in tag _name_
* `tagfn(parser, name, fn)` - pass output of _parser_ through _fn_ and store in tag _name_
* `maptags(parser, fn)` - when matched, _fn_ is called with a `dict` of tags as parameter
* `mapfn(parser, fn)` - when matched, _fn_ is called with output of _parser_ as parameter

--SLIDE--

https://github.com/krig/parsnip
