


# (APPENDIX) Appendix {-}   

# Reviews on regular expressions


References:

- [Sams Teach Yourself Regular Expressions in 10 Minutes](https://www.amazon.com/Teach-Yourself-Regular-Expressions-Minutes-ebook/dp/B0027KRPHM/ref=sr_1_2?dchild=1&keywords=Sams+Teach+Yourself+Regular+Expressions+in+10+Minutes&qid=1585917048&sr=8-2)  
- https://regexr.com/

## Metacharacters and POSIX character classes  

- `\w` matches any word character (alphabet or number, or alphanumeric) and underscore, equivalent to `[A-Za-z0-9_]`.  
- `\W` is the opposite of `\w` that matches non-word character, or `[^A-Za-z0-9_]`
- `\d` matches any single digit number  
- `.` matches any character **except** linebreaks, equivalent to `[^\r\n]` (Windows) or `[\n]` (Mac)  
- `\s` matches any white space, including spaces, tabs and vertical tab, return and line breaks, equivalent to `[:space:]` in the following table.  
- `\S` is the opposite of `\s` that matches any non-white character. `[\s\S]` is a common shorthand for **matching everything**, since `.` does not match linebreak. 


And there are POSIX character classes.  

|class|description|
|:-:|:-:|
|`[:alnum:]`|alphabets or numbers, equivalent to `[A-Za-z0-9]`|
|`[:alpha:]`|alphabets, equivalent to `[A-Za-z]`|
|`[:punct:]`|punctuation|
|`[:blank:]`|space or tab, equivalent to `[\t ]`|  
|`[:space:]`|any whitespace character including space `[\f\n\r\t\v ]`| 
|`[:print:]`|any printable character, a similar expression is `[:graph:]` which excludes space  
|`[:xdigit:]`|any hexadecimal digit, equivalent to  `[F-Aa-f0-9]`|  

## Unicode Code Points, Categories, Blocks, and Scripts  

https://www.regular-expressions.info/unicode.html

Unicode is a character set that aims to define all characters and glyphs from all human languages, living and dead. The Unicode Standard defines a codespace of numerical values ranging from 0 through 10FFFF16, called code points and denoted as U+0000 through U+10FFFF ("U+" plus the code point value in hexadecimal, prepended with leading zeros as necessary to result in a minimum of four digits^[Not all codepoint correspond to a character since there are many reserved positions, the first meaningful character in current Unicode is `U+0020`, which is the blank space as word divider]. This section introduces regular expressions that leverage this powerful mapping. 

We have stated that `.` matches any single character, in Unicode parlance this means “the dot matches any single **Unicode code point**”. For instance, à can be encoded as two code points: U+0061 (a) followed by U+0300 (grave accent)^[In R, this can be printed with `print("\u0061\u0300")`]. In this situation, . applied to à will match the first code point without the accent. `^.$` will fail to match, and `^..$` matches `à`.  

The Unicode code point U+0300 (grave accent) is a **combining mark**. Any code point that is not a combining mark can be followed by any number of combining marks. This sequence, like U+0061 U+0300 above, is displayed as a single grapheme^[The smallest meaningful contrastive unit in a writing system.] on the screen.  

To match a single grapheme, whether it be just one codepoint all a codepoint followed by multiple combining marks, we can use metacharacter `\X`, which is rougly the Unicode version of `.`. One difference is that `\X` always matches line break characters.   

### Unicode categories

In the case of regualr expressions, Unicode also brings new possibilites of matching. One notion is that each Unicode character belongs to a certain category. For example, `\p{L}` will match a single character belonging to the *letter* category, and `\p{P}` means the *punctuation category*. There are also subcategories such as `\p{LI}` will match the lowercase letter subcategory,  children of `\p{L}`. A list of all Unicode categories and subcategories can be found at https://en.wikipedia.org/wiki/Unicode_character_property#General_Category.

### Unicode scripts

The Unicode standard places each assigned code point (character) into one script. A script is a group of code points used by a particular human writing system. There are scripts like `Thai` Thai correspond to a single human language (denoted by `\p{Common}`), and scripts like `Latin` spanning multiple languages (denoted by `\p{Latin}`, including basic ASCII characters, latin supplements, latin extended and more).  

`\p{Han}` is the script for Chinese.


```r
str_view_all("彷佛全世界的细雨下在全世界的grassland上", "\\p{Han}")
```

<!--html_preserve--><div id="htmlwidget-d87e2579ce8ae6f76324" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-d87e2579ce8ae6f76324">{"x":{"html":"<ul>\n  <li><span class='match'>彷<\/span><span class='match'>佛<\/span><span class='match'>全<\/span><span class='match'>世<\/span><span class='match'>界<\/span><span class='match'>的<\/span><span class='match'>细<\/span><span class='match'>雨<\/span><span class='match'>下<\/span><span class='match'>在<\/span><span class='match'>全<\/span><span class='match'>世<\/span><span class='match'>界<\/span><span class='match'>的<\/span>grassland<span class='match'>上<\/span><\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

A special script is the `Common` script. This script contains all sorts of characters that are common to a wide range of scripts. It includes all sorts of punctuation, whitespace and miscellaneous symbols.  

### Unicode blocks

A Unicode block is a certain range of code points. An essential difference between blocks and scripts is that a block is a single contiguous range of code points, and blocks do not correspond 100% with scripts. 

For ASCII characters, the block is `[\u0000–\u007F ]`, and for Chinese `[\u4E00-\u9FA5]`







## Greedy and lazy quantifiers 

|Greedy|Lazy|
|:-:|:-:|
|`*`|`*?`|
|`+`|`+?`|
|`{n, }`|`{n, }?`|  

A common use case of lazy quantifiers is when we need to strip from html form text all its tags: 


```r
text <- "This offer is not available to customers living in <B>AK</B> and <B>HI</B>"

# lazy 
str_extract_all(text, "<[Bb]>.+?</[Bb]>")
#> [[1]]
#> [1] "<B>AK</B>" "<B>HI</B>"
# greedy
str_extract_all(text, "<[Bb]>.+</[Bb]>")
#> [[1]]
#> [1] "<B>AK</B> and <B>HI</B>"
```





## Looking ahead and back  

Lookahead specifies a pattern to be matched but not returned. A *lookahead* is actually a subexpression and is formatted as such. The syntax for a lookahead pattern is a **subexpression** preceded by `?=`, and the text to match follows the `=` sign.  Some refer to this behaviour as "match but not consume", in the sense that lookhead and lookahead match a pattern after/before what we actually want to extract, but do not return it.   

In the following example, we only want to matcch "my homepage" that followed by a `</title>`, and we do not want `</title>` in the results  

```r
text <- c("<title>my homepage</title>", "<p>my homepage</p>")
str_extract(text, "my homepage(?=</title>)")
#> [1] "my homepage" NA
# looking ahead (and back) must be used in subexpressions 
str_extract(text, "my homepage?=</title>")
#> [1] NA NA
```


Similarly, `?<=` is interpreted as the *lookback* operator, which specifies a pattern before the text we actually want to extract. Following is an example. A database search lists products, and you need only the prices.  

Following is an example. A database search lists products, and you need only the prices.  


```r
text <- c("ABC01: $23.45", 
          "HGG42: $5.31", 
          "CFMX1: $899.00", 
          "XTC99: $69.96", 
          "Total items found: 4")

str_extract(text, "(?<=\\$)[0-9]+")
#> [1] "23"  "5"   "899" "69"  NA
```

ookahead and lookbehind operations may be combined, as in the following example  


```r
str_extract("<title>my homepage</title>", "(?<=<title>)my homepage(?=</title>)")
#> [1] "my homepage"
```


Additionally, `(?=)` and `(?<=)` are known as **positive** lookahead and lookback. A lesser used version is the **negative** form of those two operators, looking for text that does not match the specified pattern.  

class | description |  
:-:|:-:|
`(?=)` | positive lookahead |
`(?!)`| negative lookahead |
`(?<=)` | positive lookbehind | 
`(?<!)`| negative lookbehind|


Suppose we want to extract just the quantities but not the prices in the followin text: 

```r
text <- c("I paid $30 for 100 apples, 50 oranges, and 60 pears. I saved $5 on this order.")
# without word boundary, 0 after 3 as in $30 will be included
str_view_all(text, "\\b(?<!\\$)\\d+\\b")
```

<!--html_preserve--><div id="htmlwidget-8d217188afc5c82e30eb" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-8d217188afc5c82e30eb">{"x":{"html":"<ul>\n  <li>I paid $30 for <span class='match'>100<\/span> apples, <span class='match'>50<\/span> oranges, and <span class='match'>60<\/span> pears. I saved $5 on this order.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


## Backreferences 


Backreferences are used to overcome the problem that one match has no knowledge of its previous match, appearing as a pair of a subexpression and a `\number` referencing to that subexpression.  

Find all repeated words (often typos):  


```r
text <- "This is a block of of text, several words here are are repeated, and and they should not be."
str_view_all(text, "(\\w+) \\1")
```

<!--html_preserve--><div id="htmlwidget-10a657b698926e14e0a1" style="width:960px;height:100%;" class="str_view html-widget"></div>
<script type="application/json" data-for="htmlwidget-10a657b698926e14e0a1">{"x":{"html":"<ul>\n  <li>Th<span class='match'>is is<\/span> a block <span class='match'>of of<\/span> text, several words here <span class='match'>are are<\/span> repeated, <span class='match'>and and<\/span> they should not be.<\/li>\n<\/ul>"},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

Another example with html data where we want to match all normal header tags, note that the last pair `<h2>...<h3>` is invalid:  


```r
text <- "<BODY>
<H1>Welcome to my Homepage</H1>
Content is divided into two sections:<BR>
<H2>ColdFusion</H2>
Information about Macromedia ColdFusion.
<H2>Wireless</H2>
Information about Bluetooth, 802.11, and more.
<H2>This is not valid HTML</H3>
</BODY>"

str_extract_all(text, "<[Hh](\\d)>.+</[Hh]\\1>")
#> [[1]]
#> [1] "<H1>Welcome to my Homepage</H1>" "<H2>ColdFusion</H2>"            
#> [3] "<H2>Wireless</H2>"
```

**Backreferences is particularly useful when performing replace operations**.  


```r
text <- "user@gmail.com is my email address"
str_replace(text, "(.+@.+\\.com)", "<a href: \\1>\\1<a>")
#> [1] "<a href: user@gmail.com>user@gmail.com<a> is my email address"
```




