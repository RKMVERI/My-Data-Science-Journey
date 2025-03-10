R String Basic
================
Kasper Welbers & Wouter van Atteveldt
2022-01

-   [Introduction](#introduction)
-   [A note on characters and
    factors](#a-note-on-characters-and-factors)
-   [String basics](#string-basics)
    -   [Combining strings](#combining-strings)
    -   [Subsetting strings](#subsetting-strings)
-   [Regular expressions](#regular-expressions)
    -   [Finding patterns](#finding-patterns)
    -   [Replacing patterns](#replacing-patterns)
    -   [Extracting patterns](#extracting-patterns)
-   [Encoding](#encoding)
    -   [Background](#background)
    -   [Text encoding in R](#text-encoding-in-r)
    -   [Dealing with encodings](#dealing-with-encodings)

# Introduction

The goal of this tutorial is to get you acquainted with basic string
handling in R. A large part of this uses the `stringr` included in the
[Tidyverse](https://www.tidyverse.org/). See also chapter 14 of [R for
Data Science](http://r4ds.had.co.nz/) and the [stringr cheat
sheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/strings.pdf)

Note that ‘string’ is not an official word in R (which uses `character`
to denote textual data), but since it’s the word used in most
documentations I will also use `strings` to refer to objects containing
textual data. (AFAIK, the name originates from seeing a text as a
`string` or sequence of characters)

# A note on characters and factors

In R, every vector (column) has a data type, which you can inspect using
`class(x)`. Two relevant data types are `character` for actual texts,
and `factor` for labeled nominal values (groups).

In most cases, you want you texts to be stored as `character`, but many
default functions (like `data.frame` and `read.csv`) store text colunms
as `factor` – while the tidyverse equivalents of `tibble` and `read_csv`
use `character` by default.

In most cases, in R you can convert object types using `as.character`:

``` r
df = data.frame(name=c("john", "mary"))
class(df$name)
library(tidyverse)
tib = tibble(name=c("john", "mary"))
class(tib$name)
```

Before R version 4.0, textual columns in data frames were automatically
converted to a `factor`. A `factor` looks like a `character` column, but
it is intended for groups or nominal values. Internally it is stored as
a number, with a label attached to each possible value.

This makes storing large amounts of data more efficient (numbers are
smaller than texts), but sometimes, you get unexpected results when you
apply a text-based function to a factor. For example, normally
`as.numeric` can be used to convert strings to numbers, setting
non-numeric strings to NA:

``` r
as.numeric(c("22", "16", "x"))
```

However, if we have this data encoded as a factor, it gives a completely
different result:

``` r
survey = data.frame(age=c("22", "16", "1"), stringsAsFactors = TRUE)
as.numeric(survey$age)
```

What happened here? The trick is to remember that factors are intended
as labeled groups, and internally R stores the group index and has a
separate list of labels (by default sorted alphabetically). Thus, “22”
was the second element in the labels, “16” the first, etc. To convert
the labels into numbers, convert to character first (which uses the
labels) before converting to numeric:

``` r
as.numeric(as.character(survey$age))
```

Since R version 4.0, these problems are much less common, but it’s good
to be aware of the difference between factors and strings. Also, when
using `ggplot` it’s important to set factor data types and levels
correctly for nominal values as they control the ordering and
visualization of nominal data. For more information on dealing with
factors, see the [forcats package](https://forcats.tidyverse.org/) and
[cheat
sheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/factors.pdf)

# String basics

The package `stringr` has a number of functions for dealing with
strings. For example, `str_length` gives the length of the string:

``` r
str_length("john")
```

As usual, these functions can be applied to a vector or column of
strings directly:

``` r
str_length(c("john", "mary"))
str_length(df$name)
mutate(df, len=str_length(name))
```

Other useful functions are `str_to_lower`, `str_to_upper` (which mostly
mimic built-in `tolower` and `toupper`), and `str_to_title`.

## Combining strings

To combine two strings, you can use `str_c` (which is equivalent to
built-in `paste0`):

``` r
str_c("john", "mary")
str_c("john", "mary", sep = " & ")
```

It can also work of longer vectors, where shorter vectors are repeated
as needed:

``` r
names = c("john", "mary")
str_c("Hello, ", names)
```

Finally, you can also ask it to *collapse* longer vectors after the
initial pasting:

``` r
str_c(names, collapse=" & ")
str_c("Hello, ", names, collapse=" and ")
```

## Subsetting strings

To take a fixed subset of a string, you can use str\_sub. This can be
useful, for example, to strip the time part off dates:

``` r
dates = c("2019-04-01T12:00", "2012-07-29 01:12")
str_sub(dates, start = 1, end = 10)
```

You can also replace a substring, for example to make sure the ‘T’
notation is used in the dates:

``` r
str_sub(dates, start=11, end=11) = "T"
dates
```

# Regular expressions

The example above showed how to extract or replace a fixed part of a
string. In many cases, however, we want to find, replace, or extract
certain patterns in a string (for example, dates, email addresses, or
html tags).

For this purpose, R (like most other languages) use *regular
expressions*, a very powerful way to write text patterns. Although a
full overview of regular expressions is beyond the scope of this handout
(there’s full books written on the subject!), below are some examples of
what you can do.

Note: To find a regular expression in a text, I use the `str_view`
command, which is quite useful for designing/debugging expressions:

``` r
txt = c("Hi, I'm Bob", "my email address  is  Bob@example.com", "A #hashtag for the #millenials")
str_view(txt, "my") # literal text
str_view(txt, "m.") # . matches any character 
str_view(txt, "\\.") # \\. matches an actual period (.)  
str_view(txt, "\\w") # \w matches any alphanumeric character (including numbers)
str_view(txt, "\\W") # \W matches any alphanumeric character 
str_view(txt, "[a-z]") # [..] are character ranges, in this case, all lower caps letters 
str_view(txt, "[^a-z]") # [^..] is a negated character range, in this case, all except lower caps letters 
str_view(txt, "\\ba") # \b matches word boundaries,this matches an a at the beginning of a word
```

The different options above all match (a sequence of) individual
characters. You can also specify multiples of a character:

``` r
str_view(txt, "ad*")   # * means an a followed by zero or more (d's, in this case)
                       # and as many as possible (greedy)
str_view(txt, "ad*?")  # *? means zero or more (d's, in this case), but as few as needed (non-greedy)
str_view(txt, "ad+")   # + means one or more d's
str_view(txt, "ad+?")  # + means one or more d's, but as few as needed
str_view(txt, "ad?")   # ? means zero or one, i.e. an 'optional' match
str_view(txt, "add?")  # a single d, optionally followed by another d
str_view(txt, "B.*m")  # a B, followed by zero or more of any character 
                       # (and as many as possible), followed by an m
str_view(txt, "B.*?m") # a B, followed by zero or more of any character 
                       # (but as few as needed), followed by an m
```

These elements can be combined to make fairly powerful patterns, such as
for emails or introductions:

``` r
str_view(txt, "\\w+@\\w+\\.\\w+")  # matches (some) email addresses
str_view(txt, "I'm \\w+\\b")       # matches "I'm XXX" phrases
str_view(txt,  "#\\w+")            # matches #hashtags
```

Note, the email address pattern is far from complete, and will match
addresses with subdomains, numbers, and many other possibilities. It
turns out emails are surprisingly complex to match - but the pattern
below should do pretty well for all but the most arcane addresses:

``` r
regex_email = regex("[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+")
str_view(txt, regex_email)
```

For more information, see the [relevant section of
R4DS](https://r4ds.had.co.nz/strings.html#matching-patterns-with-regular-expressions),
or one of the many available resources on regular expressions.

## Finding patterns

Regular expressions can be used e.g. to find rows containing a specific
pattern. For example, if we had a data frame containing the earlier
texts, we can filter for rows containing an email address:

``` r
t = tibble(id=1:3, text=txt)
t
t %>% filter(str_detect(text, regex_email))
```

You can also `str_count` to count how many matches of a pattern are
found in each text:

``` r
t %>% mutate(n_hashtags = str_count(text, "#\\w+"))
```

## Replacing patterns

You can also use regular expressions to do find-an-replace. For example,
you can remove all punctionation, normalize whitespace, or redact email
addresses:

``` r
t %>% mutate(nopunct = str_replace_all(text, "[^\\w ]", ""),
             normalized = str_replace_all(text, "\\s+", " "),
             redacted = str_replace_all(text, "\\w+@", "****@"),
             text = NULL)
```

Note the use of setting text to NULL as an alternative way to drop a
column. In `textr`, most functions have a `_all` variant which
replaces/finds/extracts all matches, rather than just the first.

## Extracting patterns

Besides replacing patterns, it can also be useful to extract elements
from a string, for example the email or hashtag:

``` r
t %>% mutate(email = str_extract(text, regex_email),
             hashtag = str_extract(text, "#\\w+"))
```

Note that for the hashtag, it only extracted the first hit. You can use
the `str_extract_all` function, but since it can match zero, once, or
more often per text, it returns a list containing all matches per row
(or more correctly, per element of the input vector):

``` r
str_extract_all(t$text, "#\\w+")
```

The best way to deal with this output might be to turn it into ‘tidy’
data, by using `simplify=T` (which turns the matches into columns),
converting to a tibble, adding the identifier, and gathering and
filtering the results:

``` r
matches =str_extract_all(t$text, "#\\w+", simplify = T)
matches = matches %>% as_tibble %>% mutate(id = t$id) %>%
  gather(var, hashtag, -id) %>% filter(hashtag != "") %>% select(-var)
matches 
```

(if you know a more elegant way, let me know <sub>:-)</sub>)

# Encoding

*NOTE*: this code is not tested on non-western/non-linux computers, so
I’m really curious if/how it will work on other computers!

Finally, a short note about unicode and character encodings. As with
regular expressions, a full explanation of this topic is (well) beyond
the scope of this tutorial. See this [guide on unicode in
R](https://kevinushey.github.io/blog/2018/02/21/string-encoding-and-r/)
and the classic [What Every Programmer .. Needs To Know About Encodings
..](http://kunststube.net/encoding/) for some very useful information if
you need to know more.

## Background

A fairly short version of the story is as follows: when computers were
mostly dealing with English text, life was easy, as there are not a lot
of different letters and they could easily assign each letter and some
punctuation marks to a number below 128, so it could be stored as 7
bits. For example, A is number 65. This encoding was called ‘ASCII’.

It turned out, however, that many people needed more than 26 letters,
for example to write accented letters. For this reason, the 7 bits were
expanded to 8, and many accented latin letters were added. This
representation is called latin-1, also known as ISO-8859-1.

Of course, many languages don’t use the latin script, so other 8-bit
encodings were invented to deal with Cyrillic, Arabic, and other
scripts. Most of these are based on ASCII, meaning that 65 still refers
to ‘A’ in e.g. the Hebrew encoding. However, character 228 could refer
to greek δ, cyrillic ф, or hebrew ה. Things get even more complex if you
consider Chinese, where you can’t fit all characters in 256 numbers, so
several larger (multi-byte) encodings were used.

This can cause a lot of confusion if you read a text that was encoding
in e.g. greek as if it were encoded in Hebrew. A famous example of this
confusion is that Microsoft Exchange used the WingDings font and
encoding for rendering symbols in emails, amongst others using character
74 as a smiley. For non-exchange users (who didn’t have that font),
however, it renders as the ASCII character nr 74: “J”. So, if you see an
email from a microsoft user with a J where you expected a smiley, now
you know `:)`.

To end this confusion, *unicode* was invented, which assigns a unique
number (called a code point) to each letter. A is still 65 (or “” in
hexadecimal R notataion), but δ is now uniquely “”, and ф is uniquely
“”. There are over 1M possible unicode characters, of which about 100
thousand have been currently assigned. This gives enough room for
Chinese, old Nordic runes, and even Klingon to be encoded.

You can directly use these in an R string:

``` r
"Some Unicode letters: \u41 \u03B4 \u0444"
```

Now, to be able to write all 1M characters to string, one would need
almost 24 bits per character, tripling the storage and memory needed to
handle most text. So, more efficient encodings were invented that would
normally take only 8 or 16 bits per character, but can take more bits if
needed. So, while the problem of defining characters is solved,
unfortunately you still need to know the actual encoding of a text.
Fortunately, UTF-8 (which uses 1 byte for latin characters, but more for
non-western text) is emerging as a de facto standard for most texts.
This is a compromise which is most efficient for latin alphabeters, but
is still able to unambiguously express all languages.

It is still quite common, however, to encounter text in other encodings,
so it can be good to understand what problems you can face and how to
deal with them

## Text encoding in R

To show how this works in R, we can use the charToRaw function to see
how a character is encoded in R:

``` r
charToRaw('A')
```

    ## [1] 41

Note that the output of this function depends on your regional settings
(called ‘locale’). On most computers, this should produce 41 however, as
most encodings are based on ASCII.

For other alphabets it can be more tricky. The Chinese character “蘭”
(unicode “”) on my computer is expressed in UTF-8, where it takes 3
bytes:

## Dealing with encodings

To convert between encodings, you can use the iconv function. For
example, to express the Chinese character above in GB2312 (Chinese
national standard) encoding:

``` r
charToRaw(iconv('蘭', to='GB2312'))
```

The most common way of dealing with encodings is to ignore the problem
and hope it goes away. However, outside the English world this is often
not an option. Also, due to general unicode ignorance many people will
use the wrong encoding, and you will even see things like
double-utf8-encoded text.

The *sane* way to deal with encodings is to make sure that all text
stored inside your program is encoded in a standard encoding, presumably
UTF-8. This means that whenever you read text from an external source,
you need to convert it to UTF-8 if it isn’t already in that form.

This means that when you use `read_csv` (on text data) or `readtext`,
you should ideally *always* specify which encoding the text is encoded
in:

``` r
readtext::readtext("file.txt", encoding = "utf-8")
read_csv("file.csv", locale=locale(encoding='utf-8'))
```

If you don’t know what encoding a text is in, you can try utf-8 and the
most common local encodings (e.g. latin-1 in many western countries),
you can inspect the raw bytes, or you can use the guessEncoding function
from readr:

``` r
guess_encoding("file.txt")
```
