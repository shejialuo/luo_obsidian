# Type System

## Numbers

For perl, all numbers have the same format internally. It relies on the C libraries for its numbers and uses a double-precision floating-point value to store the numbers.

### Integer Literals

Perl allows you to add underscores for clarity within integer literals.

```perl
0
2001
-40
137
61298040283768
```

### Nondecimal Integer Literals

Perl allows you to specify numbers in other ways than base 10 (decimal). Octal (base 8) literals start with a leading 0 and use the digits from 0 to 7. Hexadecimal (base 16) literals start with a leading `0x`. Binary (base 2) literals start with a leading `0b`.

```perl
0377 # same as 255 decimal
0xff # FF hex, also 255 decimal
0b11111111 # also 255 decimal
```

### Floating-Point Literals

Numbers with and without decimal points are allowed, as well as tacking on a power-of-10 indicator with E notation (also known as *exponential notation*).

```perl
1.25
255.000
255.0
7.25e45
-6.5e24
-1.2E-23
```

### Numeric Operators

Perl provides the typical ordinary addition, subtraction, multiplication, and division operators. These numeric operators always treat their operands as numbers, and are denoted by symbolic characters:

```perl
2 + 3
5.1 - 2.4
3 * 12
14 / 2
10.2 / 0.3
```

Perl only has single values and it doesn't distinguish numbers that are integers, fractions, or floating-point numbers. Perl also supports a *modulus* operator (`%`). Both values are first reduced to their integer values, so `10.5 % 3.2` is computed as `10 % 3`.

## Strings

It's a good practice to always include this in your program unless you know why you wouldn't want to:

```perl
use utf8;
```

Literal strings come in two different flavors: *single-quoted string literals* and *double-quoted string literals*.

### Single-Quoted String Literals

A *single-quoted string literal* is a sequence of characters enclosed in single quotes. Any character other than a single quote or a backslash between the quote marks stands **for itself**. If you want a literal single quote or backslash inside your string, you need to *escape* it with a backslash.

```perl
'Don\'t let it happen'
'the last character is a backslash: \\'
'\'\\' # single quote followed by backslash
```

You can spread your string out over two (or more) lines. A new line between the single quotes is a newline in your string:

```perl
'hello
there'
```

Note that Perl does not interpret the `\n` within a single-quoted string as a newline, but as the two characters backslash and `n`:

```perl
'hello\nthere'    # hellonthere
```

### Double-Quoted String Literals

A *double-quoted string literal* is a sequence of characters, although this time enclosed in double quotes. But now the backslash takes on its full power to specify certain control characters, or even any character at all through otcal and hex representations.

```perl
"barney" # just the same as 'barney'
"hello world\n" # hello world, and a newline
"The last character of this string is a quote mark: \""
"coke\tsprite" #coke, a tab and sprite
"\x{2668}" # Unicode HOT SPRINGS character code point
```

### String Operators

You can concatenate, or join, string values with the `.` operator. This does not alter either string. The resulkting string is then available for further computation or assignment to a variable.

```perl
"hello" . "world"
"hello" . ' ' . "world"
```

A special string operator is the *string repetition* operator, consisting of the single lowercase letter `x`. This operator takes its left operand and makes as many concatenated copies of that string as indicated by its right operand (a number).

```perl
"fred" x 3
"barney" x (4 + 1)
5 x 4.8 # is really "5" x 4, which is "5555"
```

### Automatic Conversion Between Numbers and Strings

Perl automatically converts between numbers and strings as needed. How does it know which it should use? It all depends upon the operator that you apply to the scalar value. If an operator expects a number (like `+` does), Perl will see the value as a number. If an operator expects a string (like `.` does), Perl will see the value as a string

## Lists and Arrays

A *list* is an ordered collection of scalars. An *array* is a variable that contains a list. The list is the data and the array is the variable that stores the data. You can have a list value that isn't in an array, but every array variable holds a list (although that list may be empty).

Each *element* of an array of list is a separate scalar value. These values are ordered. The elements of an array or a list are *indexed* by integers starting at zero and counting by ones.

### Accessing Elements of an Array

The array elements are numbered using sequential integers, beginning at 0 and increasing by 1 for each element, like this:

```perl
$fred[0] = "yabba";
$fred[1] = "dabba";
$fred[2] = "doo";
```

### Special Array Indices

Sometimes, you need to find out the last element index in an array. For the array of `rocks`, the last element index is `$#rocks`.

### List Literals

A *list literal* is a list of comma-separated values enclosed in parentheses. These values form the elements of the list.

```perl
(1, 2, 3)
(1, 2, 3,)
("fred", 4.5)
()
```

You don't have to type out every value if they are in sequence. The `..` *range operator* creates a list of values by counting from the left scalar up to the right scalar by ones.

```perl
(1..100) # list of 100 integers
(1..5) # same as (1, 2, 3, 4, 5)
(1.7..5.7) # same thing; both values are truncated
(0, 2..6, 10, 12)
```

#### The qw Shortcut

A list may have any scalar values, like this typical list of strings:

```perl
("fred", "barney", "betty", "wilma", "dino")
```

It turns out that lists of simple words are frequently in Perl programs. The `qw` shortcut makes it easy to generate them without typing a lof of extra quote marks:

```perl
qw(fred barney betty wilma dino)
```

Perl treats `qw` like a single-quoted string. The whitespace (characters like spaces, tabs, and newlines) disappear and whatever is left becomes the list of items. Since whitespace is insignificant, here's another (but unusual) way to write that same list:

```perl
qw(fred
  barney     betty
wilma dino)
```

Perl lets you choose any punctuation character as the delimiter.

```perl
qw! fred barney betty wilma dino !
```
