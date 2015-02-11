# tush -- a literate testing shell

This is sort of a doctest for shellscripts. There are two major pluses:

  * The examples in your documentation get checked automatically.
  * Tests can be easy to write and to read.

This style of testing has proved itself in language-specific tools
like Python's doctest and E's UpDoc (and FIT for Java, sort of). But
to test command-line programs it seems to be usual to write
shellscripts that each set up some files, etc., call on the subject
program, check the result, and clean up. This isn't fun, and less fun
means fewer tests get written. After I started a project involving
lots of command-line programs I wrote tush instead.

## To install it:

Copy bin/* from this directory to somewhere in your PATH.

## To use it:

Tush looks for transcript-like lines in a file and checks them. For
example:

```sh
$ echo Hello world
| Hello world
```

If you run `tush-check README` (where README is this file), it notices
the above two lines, executes 'echo Hello world', and checks that
'Hello world' comes out on the standard output. Assuming the test
passes, running tush-check succeeds silently. A failing test makes
tush-check fail and output a diff.

You aren't limited to invoking the program under test; setup,
clean-up, checking, etc., work the same way:

```sh
$ echo  >test.in 'here is some test input'
$ echo >>test.in 'and here is some more'
$ sort test.in | wc -l  # Check: sorting should not change the linecount.
|        2
```

We didn't bother to rm test.in afterwards because we have a crude kind
of test isolation already: Tush makes a new temporary directory named
tush-scratch, runs all the commands in the input from within it, then
deletes it.

What about checking commands that should fail? There are two more
special prefixes. For example:

```sh
$ cat nonesuch
@ cat: nonesuch: No such file or directory
? 1
```

The '@ ' line is like '| ', only for standard error instead of standard
output. The '? ' line shows a nonzero exit status.

## Tools:

tush-check was introduced above. It calls tush-run, which runs
tush-run-raw from within a temporary tush-scratch directory. 

tush-run-raw copies its input except for the special-prefixed lines
introduced above: '$ ' lines are copied, too, but also executed, with
their outputs/status codes inserted into the output with appropriate
prefixes, so that for a successful test the output is the same as the
input. Input lines starting with '| ', '@ ', or '? ' are dropped.

The above tools, like 'cat', take any number of files as arguments.

'tush-bless foo' updates foo so that 'tush-check foo' will then pass:
it changes any output/status-code lines to the actual outputs from
tush-run. Use this when your program is correct but your test is
wrong.

## Examples

See the *.tush files in this directory. They're a kind of self-check
of the Tush implementation, though a weak one, since implementing
tush-run as 'cat' would pass it.  [XXX do something about that?]

## Emacs mode

tush.el lets you pass the file you're editing through tush-run with a
single keystroke. This can go nicely with interactive development
combining unit tests with the code under test.

## Alternatives

Why not more like UpDoc or doctest?  [TODO: explain]

## Credits:

Darius Bacon <darius@wry.me>

The 'overwrite' script is from Kernighan and Pike, _The UNIX
Programming Environment_, and adapted by Clyde Ingram.
