# Regex Cheat Sheet
I will only include types that are mostly used.

identifier|name|example
----------|----|-------
\d | digit from 0 to 9 |
\w | ascii letter, digit, underscore|
\s | space, tab, newline, etc. |
\D | nondigit char|
\W | not-a-word char |
\S | not-a-whitespace char|
||
+ | one or more |
{3} | exactly 3 times |
{2,4} | 2 to 4 times|
{3,} | 3 or more times|
* | zero or more |
? | once or more |
||
. | any char except line break|
\ | escapes a special char|
||
\| | alternation |
() | capturing group | A(nt|pple)
\1 | contents of group 1 | 	r(\w)g\1x	-> regex
||
\t | tab |
\r | carriage return char |
\n | line break |
\h \H \v \V \R||
||
[abc] | one of the char in brackets | T[ao]p -> Tap
[x-y] | once of the char between x and y| [A-Z]+ -> HAHA
[\^x] | the char is not x | [\^a-z]{3} -> 123
[\^x-y] | the char is not x to y |
||
\^|start of a string or start of line| ^abc.\* -> abcdefg
$|end of a string or line | .\*abc$ -> efgabc
