+++
date = '2025-03-03T00:26:11+03:30'
draft = false
title = 'Unix for Poets: Basic Text Processing Using Unix Tools'
tags = ["unix", "linux", "tr", "text_processing"]
summary = "Kenneth Ward Church in *Unix™ for Poets* shows how basic text processing tasks like splitting text into words, counting words, sorting words in rhyming order and even computing n-grams can be done by anybody using simple unix tools without any fancy tools or algorithms, even on a modest PC."
+++


Often, we become so captivated by complexity and sophistication that we overlook the profound effectiveness of simple, fundamental methods.

Kenneth Ward Church in wonderful *Unix™ for Poets* [[1]](#references) shows how basic text processing tasks like splitting text into words, counting words, sorting words in rhyming order and even computing n-grams can be done by anybody using simple unix tools without any fancy tools or algorithms, even on a modest PC. In this article, we look at some basic NLP tasks using Unix tools based on Unix™ for Poets with Moby-Dick corpus [[2]](#references). Which is 214,492 words, 19,390 lines and 1.2MB in size.


## 1. Count Word Frequencis In a Text

The problem is to input a text file, and output a list of words in the file along with their frequency counts. To split text into words we can use this simple method:

```bash
> tr -sc 'a-zA-Z' '\n' < moby.txt | tr 'A-Z' 'a-z' > words.txt
> less words.txt
moby
dick
or
the
whale
by
herman
melville
chapter
loomings
call
me
ishmael
some
years
...
```

`tr` can be used to translate or delete characters. Option `-c` means complement. So In this example we use `tr` to convert all non-alphabetical characters to `\n`. By `-s`, we specify to replace each sequence of a repeated character with a single occurrence of that character. Hence every time tr encounters a non-alphabetical character (eg. space, punctuation, numbers) It replaces it with a `\n` character. Finaly we replace all uppercase letters by lowercase ones. We store the result in words.txt file.

To calculate world frequencies, we can simply use sort and uniq commands.

```bash
> sort words.txt | uniq -c | sort -nr | less
  14178 the
   6467 of
   6321 and
   4629 a
   4537 to
   4073 in
   3036 that
   2494 his
   2489 it
   2108 i
   1874 he
   1804 but
   1735 s
   1717 as
   1691 with
   1691 is
   1628 was
   1594 for
   1513 all
   1377 this
...
```

An we have list of words sorted by frequency in the output. Note that there are some entries like a single `s`. It's beacuse we use a very simple word tokenization method, words like It's will break into two parts It and s. We can use more accurate tokenization methods, but in this simple example the simple appraoch serves us well.

Small changes to the tokenizing rule are easy to make, but can have a dramatic impact. For example we can count vowel sequences instead of words by changing the tokenizing rule to emit sequences of vowels rather than sequences of alphabetic characters.

```bash
> tr 'A-Z' 'a-z' < moby.txt | tr -sc 'aeiou' '\n' | sort | uniq -c | sort -nr | less
  94705 e
  63440 a
  53233 i
  50012 o
  14698 u
   7597 ou
   6274 ea
   4197 ee
   3352 ai
   2734 oo
   1895 ie
   1575 io
   1307 ei
   1262 oa
    962 ia
    886 ue
    769 oi
    540 au
...
```

With `tr 'A-Z' 'a-z'` we replace uppercase characters with lowercase ones and then with `tr -sc 'aeiou' '\n'` we replace any consonant (non-vowel) character with `\n`.

## 2. Sort Words By *Rhyming* Order

Altough it may seem like a complex task but we can do a decsent job using a very simple technic. By *rhyming* order, we mean that the words should be sorted from the right rather than the left. So the only thing we should do it to reverse words and then sort them. We can reverse lines characterwise using rev command:

```bash
> tr -sc 'a-zA-Z' '\n' < moby.txt | tr 'A-Z' 'a-z' > words.txt
> rev words.txt | sort | uniq | rev | less
a
caramba
cuba
juba
malacca
mecca
seneca
republica
oceanica
america
africa
da
armada
canada
andromeda
zogranda
anaconda
golconda
...
```

## 3. N-grams

> I have presented this material in quite a number of tutorials over the years. The students almost always come up with a big ''aha'' reaction at this point. Counting words seems easy enough, but for some reason, students are almost always pleasantly surprised to discover that counting ngrams (bigrams, trigrams, 50-grams) is just as easy as counting words. - *Kenneth Ward Church, Unix™ for Poets*

We can easily count bigrams (pairs of words), using this algorithm:


*Step 1*: Tokenize by word

*Step 2*: Print word<sub>i</sub> and word<sub>i+1</sub> on the same line

*Step 3*: Count


We already know how to tokenize words and store the in `words.txt` file:

```bash
> tr -sc 'a-zA-Z' '\n' < moby.txt | tr 'A-Z' 'a-z' > words.txt
```

For the second step we can use the tail command:

```bash
> tail +2 words.txt > next-words.txt
> less next-words.txt
```

With `tail +2 words.txt` we skip the first line and print from line 2 to the end. So `next-words.txt ` file is the `words.txt` with one line (word) shifted up. Finaly we can use paste command to merge lines in `words.txt` and `next-words.txt`:

```bash
> paste words.txt next-words.txt | sort | uniq -c | sort -nr | less
   1840 of      the
   1146 in      the
    717 to      the
    432 from    the
    377 the     whale
    369 of      his
    361 and     the
    351 on      the
    328 at      the
    324 to      be
    318 of      a
    315 by      the
    310 with    the
    304 for     the
    291 it      was
    282 it      is
    258 in      his
    256 in      a
    248 the     ship
...
```

We can count trigrams and more using this simple method. We can even make it a bash script:

```bash
#!/usr/bin/bash
tr -sc 'a-zA-Z' '\n' < moby.txt | tr 'A-Z' 'a-z' > $$words

tail +2 $$words > $$nextwords
tail +3 $$words > $$nextwords2

paste $$words $$nextwords $$nextwords2 | sort | uniq -c | sort -nr

# remove the temporary files
rm $$words $$nextwords $$nextwords2
```

We can then run it and get trigrams sorted by frequency:

```bash
> sh mktrigram.sh < moby.txt | less
    114 the     sperm   whale
     97 of      the     whale
     88 the     white   whale
     64 one     of      the
     58 of      the     sea
     56 out     of      the
     53 the     ship    s
     53 part    of      the
     52 a       sort    of
     50 the     whale   s
     48 the     pequod  s
     42 of      the     sperm
     37 of      the     ship
     36 i       don     t
     35 the     right   whale
     34 the     old     man
     33 sperm   whale   s
```

## 4. Find Palyndrome Words

Palindrome words spelled the same backward and forward, i.e noon. We can find palyndrome with this algorithm:

*Step 1*: Revers each word

*Step 2*: Find words that are identical to their reverse

```bash
> rev words.txt > rev-words.txt 
> paste words.txt rev-words.txt | awk '$1 == $2' | sort | uniq | less
...
peep    peep
pip     pip
poop    poop
pop     pop
p       p
razar   razar
redder  redder
refer   refer
sees    sees
sexes   sexes
...
```

## 5. Conclusion

Thanks to Kenneth Ward Church, we now understand that anyone can perform basic NLP tasks on their own PC using just Unix tools, even without specialized knowledge of NLP or complex algorithms. In today's landscape, dominated by the hype surrounding AI and large language models, we often underestimate the effectiveness and strength of simple, foundational techniques.

## References

[1] https://www.cs.upc.edu/~padro/Unixforpoets.pdf

[2] [Moby Deck Corpus](https://courses.cs.washington.edu/courses/cse390c/22sp/lectures/moby.txt)