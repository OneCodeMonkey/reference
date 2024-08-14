Groovy 语法
===

ref: https://groovy-lang.org/syntax.html

Syntax
--------

This chapter covers the syntax of the Groovy programming language. 
The grammar of the language derives from the Java grammar, but enhances it with specific constructs for Groovy, and allows certain simplifications.

### 1. Comments
#### 1.1. Single-line comment
Single-line comments start with // and can be found at any position in the line. The characters following //, until the end of the line, are considered part of the comment.

// a standalone single line comment
println "hello" // a comment till the end of the line

#### 1.2. Multiline comment
A multiline comment starts with /* and can be found at any position in the line. The characters following /* will be considered part of the comment, including new line characters, up to the first */ closing the comment. Multiline comments can thus be put at the end of a statement, or even inside a statement.

/* a standalone multiline comment
   spanning two lines */
println "hello" /* a multiline comment starting
                   at the end of a statement */
println 1 /* one */ + 2 /* two */

#### 1.3. Groovydoc comment
Similarly to multiline comments, Groovydoc comments are multiline, but start with /** and end with */. Lines following the first Groovydoc comment line can optionally start with a star *. Those comments are associated with:

type definitions (classes, interfaces, enums, annotations),

fields and properties definitions

methods definitions

Although the compiler will not complain about Groovydoc comments not being associated with the above language elements, you should prepend those constructs with the comment right before it.

```java
/**
 * A Class description
 */
class Person {
    /** the name of the person */
    String name

    /**
     * Creates a greeting method for a certain person.
     *
     * @param otherPerson the person to greet
     * @return a greeting message
     */
    String greet(String otherPerson) {
       "Hello ${otherPerson}"
    }
}
```

Groovydoc follows the same conventions as Java’s own Javadoc. So you’ll be able to use the same tags as with Javadoc.

In addition, Groovy supports Runtime Groovydoc since 3.0.0, i.e. Groovydoc can be retained at runtime.

Runtime Groovydoc is disabled by default. It can be enabled by adding JVM option -Dgroovy.attach.runtime.groovydoc=true
The Runtime Groovydoc starts with /**@ and ends with */, for example:

```java
/**@
 * Some class groovydoc for Foo
 */
class Foo {
    /**@
     * Some method groovydoc for bar
     */
    void bar() {
    }
}
```

assert Foo.class.groovydoc.content.contains('Some class groovydoc for Foo') 
assert Foo.class.getMethod('bar', new Class[0]).groovydoc.content.contains('Some method groovydoc for bar') 
Get the runtime groovydoc for class Foo
Get the runtime groovydoc for method bar

#### 1.4. Shebang line
Beside the single-line comment, there is a special line comment, often called the shebang line understood by UNIX systems which allows scripts to be run directly from the command-line, provided you have installed the Groovy distribution and the groovy command is available on the PATH.

```shell
#!/usr/bin/env groovy
println "Hello from the shebang line"
```

The # character must be the first character of the file. Any indentation would yield a compilation error.

### 2. Keywords
Groovy has the following reserved keywords:

Table 1. Reserved Keywords

abstract

assert

break

case

catch

class

const

continue

def

default

do

else

enum

extends

final

finally

for

goto

if

implements

import

instanceof

interface

native

new

null

non-sealed

package

public

protected

private

return

static

strictfp

super

switch

synchronized

this

threadsafe

throw

throws

transient

try

while

Of these, const, goto, strictfp, and threadsafe are not currently in use.

The reserved keywords can’t in general be used for variable, field and method names.

A trick allows methods to be defined having the same name as a keyword by surrounding the name in quotes as shown in the following example:

// reserved keywords can be used for method names if quoted
def "abstract"() { true }
// when calling such methods, the name must be qualified using "this."
this.abstract()
Using such names might be confusing and is often best to avoid. The trick is primarily intended to enable certain Java integration scenarios and certain DSL scenarios where having "verbs" and "nouns" with the same name as keywords may be desirable.

In addition, Groovy has the following contextual keywords:

Table 2. Contextual Keywords

as

in

permits

record

sealed

trait

var

yields

These words are only keywords in certain contexts and can be more freely used in some places, in particular for variables, fields and method names.

This extra lenience allows using method or variable names that were not keywords in earlier versions of Groovy or are not keywords in Java. Examples are shown here:

// contextual keywords can be used for field and variable names
def as = true
assert as

// contextual keywords can be used for method names
def in() { true }
// when calling such methods, the name only needs to be qualified using "this." in scenarios which would be ambiguous
this.in()
Groovy programmers familiar with these contextual keywords may still wish to avoid using those names unless there is a good reason to use such a name.

The restrictions on reserved keywords also apply for the primitive types, the boolean literals and the null literal (all of which are discussed later):

Table 3. Other reserved words

null

true

false

boolean

char

byte

short

int

long

float

double

While not recommended, the same trick as for reserved keywords can be used:

def "null"() { true }  // not recommended; potentially confusing
assert this.null()     // must be qualified
Using such words as method names is potentially confusing and is often best to avoid, however, it might be useful for certain kinds of DSLs.

### 3. Identifiers
#### 3.1. Normal identifiers
Identifiers start with a letter, a dollar or an underscore. They cannot start with a number.

A letter can be in the following ranges:

'a' to 'z' (lowercase ascii letter)

'A' to 'Z' (uppercase ascii letter)

'\u00C0' to '\u00D6'

'\u00D8' to '\u00F6'

'\u00F8' to '\u00FF'

'\u0100' to '\uFFFE'

Then following characters can contain letters and numbers.

Here are a few examples of valid identifiers (here, variable names):

def name
def item3
def with_underscore
def $dollarStart
But the following ones are invalid identifiers:

def 3tier
def a+b
def a#b
All keywords are also valid identifiers when following a dot:

foo.as
foo.assert
foo.break
foo.case
foo.catch

#### 3.2. Quoted identifiers
Quoted identifiers appear after the dot of a dotted expression. For instance, the name part of the person.name expression can be quoted with person."name" or person.'name'. This is particularly interesting when certain identifiers contain illegal characters that are forbidden by the Java Language Specification, but which are allowed by Groovy when quoted. For example, characters like a dash, a space, an exclamation mark, etc.

def map = [:]

map."an identifier with a space and double quotes" = "ALLOWED"
map.'with-dash-signs-and-single-quotes' = "ALLOWED"

assert map."an identifier with a space and double quotes" == "ALLOWED"
assert map.'with-dash-signs-and-single-quotes' == "ALLOWED"
As we shall see in the following section on strings, Groovy provides different string literals. All kind of strings are actually allowed after the dot:

map.'single quote'
map."double quote"
map.'''triple single quote'''
map."""triple double quote"""
map./slashy string/
map.$/dollar slashy string/$
There’s a difference between plain character strings and Groovy’s GStrings (interpolated strings), as in that the latter case, the interpolated values are inserted in the final string for evaluating the whole identifier:

def firstname = "Homer"
map."Simpson-${firstname}" = "Homer Simpson"

assert map.'Simpson-Homer' == "Homer Simpson"

### 4. Strings
Text literals are represented in the form of chain of characters called strings. Groovy lets you instantiate java.lang.String objects, as well as GStrings (groovy.lang.GString) which are also called interpolated strings in other programming languages.

#### 4.1. Single-quoted string
Single-quoted strings are a series of characters surrounded by single quotes:

'a single-quoted string'
Single-quoted strings are plain java.lang.String and don’t support interpolation.

#### 4.2. String concatenation
All the Groovy strings can be concatenated with the + operator:

assert 'ab' == 'a' + 'b'

#### 4.3. Triple-single-quoted string
Triple-single-quoted strings are a series of characters surrounded by triplets of single quotes:

'''a triple-single-quoted string'''
Triple-single-quoted strings are plain java.lang.String and don’t support interpolation.
Triple-single-quoted strings may span multiple lines. The content of the string can cross line boundaries without the need to split the string in several pieces and without concatenation or newline escape characters:

def aMultilineString = '''line one
line two
line three'''
If your code is indented, for example in the body of the method of a class, your string will contain the whitespace of the indentation. The Groovy Development Kit contains methods for stripping out the indentation with the String#stripIndent() method, and with the String#stripMargin() method that takes a delimiter character to identify the text to remove from the beginning of a string.

When creating a string as follows:

def startingAndEndingWithANewline = '''
line one
line two
line three
'''
You will notice that the resulting string contains a newline character as first character. It is possible to strip that character by escaping the newline with a backslash:

def strippedFirstNewline = '''\
line one
line two
line three
'''

assert !strippedFirstNewline.startsWith('\n')

##### 4.3.1. Escaping special characters
You can escape single quotes with the backslash character to avoid terminating the string literal:

'an escaped single quote: \' needs a backslash'
And you can escape the escape character itself with a double backslash:

'an escaped escape character: \\ needs a double backslash'
Some special characters also use the backslash as escape character:

Escape sequence	Character
\b

backspace

\f

formfeed

\n

newline

\r

carriage return

\s

single space

\t

tabulation

\\

backslash

\'

single quote within a single-quoted string (and optional for triple-single-quoted and double-quoted strings)

\"

double quote within a double-quoted string (and optional for triple-double-quoted and single-quoted strings)

We’ll see some more escaping details when it comes to other types of strings discussed later.

##### 4.3.2. Unicode escape sequence
For characters that are not present on your keyboard, you can use unicode escape sequences: a backslash, followed by 'u', then 4 hexadecimal digits.

For example, the Euro currency symbol can be represented with:

'The Euro currency symbol: \u20AC'

#### 4.4. Double-quoted string
Double-quoted strings are a series of characters surrounded by double quotes:

"a double-quoted string"
Double-quoted strings are plain java.lang.String if there’s no interpolated expression, but are groovy.lang.GString instances if interpolation is present.
To escape a double quote, you can use the backslash character: "A double quote: \"".

##### 4.4.1. String interpolation
Any Groovy expression can be interpolated in all string literals, apart from single and triple-single-quoted strings. Interpolation is the act of replacing a placeholder in the string with its value upon evaluation of the string. The placeholder expressions are surrounded by ${}. The curly braces may be omitted for unambiguous dotted expressions, i.e. we can use just a $ prefix in those cases. If the GString is ever passed to a method taking a String, the expression value inside the placeholder is evaluated to its string representation (by calling toString() on that expression) and the resulting String is passed to the method.

Here, we have a string with a placeholder referencing a local variable:

def name = 'Guillaume' // a plain string
def greeting = "Hello ${name}"

assert greeting.toString() == 'Hello Guillaume'
Any Groovy expression is valid, as we can see in this example with an arithmetic expression:

def sum = "The sum of 2 and 3 equals ${2 + 3}"
assert sum.toString() == 'The sum of 2 and 3 equals 5'
Not only are expressions allowed in between the ${} placeholder, but so are statements. However, a statement’s value is just null. So if several statements are inserted in that placeholder, the last one should somehow return a meaningful value to be inserted. For instance, "The sum of 1 and 2 is equal to ${def a = 1; def b = 2; a + b}" is supported and works as expected but a good practice is usually to stick to simple expressions inside GString placeholders.
In addition to ${} placeholders, we can also use a lone $ sign prefixing a dotted expression:

def person = [name: 'Guillaume', age: 36]
assert "$person.name is $person.age years old" == 'Guillaume is 36 years old'
But only dotted expressions of the form a.b, a.b.c, etc, are valid. Expressions containing parentheses like method calls, curly braces for closures, dots which aren’t part of a property expression or arithmetic operators would be invalid. Given the following variable definition of a number:

def number = 3.14
The following statement will throw a groovy.lang.MissingPropertyException because Groovy believes you’re trying to access the toString property of that number, which doesn’t exist:

shouldFail(MissingPropertyException) {
    println "$number.toString()"
}
You can think of "$number.toString()" as being interpreted by the parser as "${number.toString}()".
Similarly, if the expression is ambiguous, you need to keep the curly braces:

String thing = 'treasure'
assert 'The x-coordinate of the treasure is represented by treasure.x' ==
    "The x-coordinate of the $thing is represented by $thing.x"   // <= Not allowed: ambiguous!!
assert 'The x-coordinate of the treasure is represented by treasure.x' ==
        "The x-coordinate of the $thing is represented by ${thing}.x"  // <= Curly braces required
If you need to escape the $ or ${} placeholders in a GString so they appear as is without interpolation, you just need to use a \ backslash character to escape the dollar sign:

assert '$5' == "\$5"
assert '${name}' == "\${name}"

##### 4.4.2. Special case of interpolating closure expressions
So far, we’ve seen we could interpolate arbitrary expressions inside the ${} placeholder, but there is a special case and notation for closure expressions. When the placeholder contains an arrow, ${→}, the expression is actually a closure expression — you can think of it as a closure with a dollar prepended in front of it:

def sParameterLessClosure = "1 + 2 == ${-> 3}" 
assert sParameterLessClosure == '1 + 2 == 3'

def sOneParamClosure = "1 + 2 == ${ w -> w << 3}" 
assert sOneParamClosure == '1 + 2 == 3'
The closure is a parameterless closure which doesn’t take arguments.
Here, the closure takes a single java.io.StringWriter argument, to which you can append content with the << leftShift operator. In either case, both placeholders are embedded closures.
In appearance, it looks like a more verbose way of defining expressions to be interpolated, but closures have an interesting advantage over mere expressions: lazy evaluation.

Let’s consider the following sample:

def number = 1 
def eagerGString = "value == ${number}"
def lazyGString = "value == ${ -> number }"

assert eagerGString == "value == 1" 
assert lazyGString ==  "value == 1" 

number = 2 
assert eagerGString == "value == 1" 
assert lazyGString ==  "value == 2" 
We define a number variable containing 1 that we then interpolate within two GStrings, as an expression in eagerGString and as a closure in lazyGString.
We expect the resulting string to contain the same string value of 1 for eagerGString.
Similarly for lazyGString
Then we change the value of the variable to a new number
With a plain interpolated expression, the value was actually bound at the time of creation of the GString.
But with a closure expression, the closure is called upon each coercion of the GString into String, resulting in an updated string containing the new number value.
An embedded closure expression taking more than one parameter will generate an exception at runtime. Only closures with zero or one parameter are allowed.

##### 4.4.3. Interoperability with Java
When a method (whether implemented in Java or Groovy) expects a java.lang.String, but we pass a groovy.lang.GString instance, the toString() method of the GString is automatically and transparently called.

String takeString(String message) {         
    assert message instanceof String        
    return message
}

def message = "The message is ${'hello'}"   
assert message instanceof GString           

def result = takeString(message)            
assert result instanceof String
assert result == 'The message is hello'
We create a GString variable
We double-check it’s an instance of the GString
We then pass that GString to a method taking a String as parameter
The signature of the takeString() method explicitly says its sole parameter is a String
We also verify that the parameter is indeed a String and not a GString.

##### 4.4.4. GString and String hashCodes
Although interpolated strings can be used in lieu of plain Java strings, they differ with strings in a particular way: their hashCodes are different. Plain Java strings are immutable, whereas the resulting String representation of a GString can vary, depending on its interpolated values. Even for the same resulting string, GStrings and Strings don’t have the same hashCode.

assert "one: ${1}".hashCode() != "one: 1".hashCode()
GString and Strings having different hashCode values, using GString as Map keys should be avoided, especially if we try to retrieve an associated value with a String instead of a GString.

def key = "a"
def m = ["${key}": "letter ${key}"]     

assert m["a"] == null                   
The map is created with an initial pair whose key is a GString
When we try to fetch the value with a String key, we will not find it, as Strings and GString have different hashCode values
4.5. Triple-double-quoted string
Triple-double-quoted strings behave like double-quoted strings, with the addition that they are multiline, like the triple-single-quoted strings.

```groovy
def name = 'Groovy'
def template = """
    Dear Mr ${name},

    You're the winner of the lottery!

    Yours sincerly,

    Dave
"""
```

assert template.toString().contains('Groovy')
Neither double quotes nor single quotes need be escaped in triple-double-quoted strings.

#### 4.6. Slashy string
Beyond the usual quoted strings, Groovy offers slashy strings, which use / as the opening and closing delimiter. Slashy strings are particularly useful for defining regular expressions and patterns, as there is no need to escape backslashes.

Example of a slashy string:

def fooPattern = /.*foo.*/
assert fooPattern == '.*foo.*'
Only forward slashes need to be escaped with a backslash:

def escapeSlash = /The character \/ is a forward slash/
assert escapeSlash == 'The character / is a forward slash'
Slashy strings are multiline:

def multilineSlashy = /one
    two
    three/

assert multilineSlashy.contains('\n')
Slashy strings can be thought of as just another way to define a GString but with different escaping rules. They hence support interpolation:

def color = 'blue'
def interpolatedSlashy = /a ${color} car/

assert interpolatedSlashy == 'a blue car'

##### 4.6.1. Special cases
An empty slashy string cannot be represented with a double forward slash, as it’s understood by the Groovy parser as a line comment. That’s why the following assert would actually not compile as it would look like a non-terminated statement:

assert '' == //
As slashy strings were mostly designed to make regexp easier so a few things that are errors in GStrings like $() or $5 will work with slashy strings.

Remember that escaping backslashes is not required. An alternative way of thinking of this is that in fact escaping is not supported. The slashy string /\t/ won’t contain a tab but instead a backslash followed by the character 't'. Escaping is only allowed for the slash character, i.e. /\/folder/ will be a slashy string containing '/folder'. A consequence of slash escaping is that a slashy string can’t end with a backslash. Otherwise that will escape the slashy string terminator. You can instead use a special trick, /ends with slash ${'\'}/. But best just avoid using a slashy string in such a case.

#### 4.7. Dollar slashy string
Dollar slashy strings are multiline GStrings delimited with an opening $/ and a closing /$. The escaping character is the dollar sign, and it can escape another dollar, or a forward slash. Escaping for the dollar and forward slash characters is only needed where conflicts arise with the special use of those characters. The characters $foo would normally indicate a GString placeholder, so those four characters can be entered into a dollar slashy string by escaping the dollar, i.e. $$foo. Similarly, you will need to escape a dollar slashy closing delimiter if you want it to appear in your string.

Here are a few examples:

def name = "Guillaume"
def date = "April, 1st"

def dollarSlashy = $/
    Hello $name,
    today we're ${date}.

    $ dollar sign
    $$ escaped dollar sign
    \ backslash
    / forward slash
    $/ escaped forward slash
    $$$/ escaped opening dollar slashy
    $/$$ escaped closing dollar slashy
/$

assert [
    'Guillaume',
    'April, 1st',
    '$ dollar sign',
    '$ escaped dollar sign',
    '\\ backslash',
    '/ forward slash',
    '/ escaped forward slash',
    '$/ escaped opening dollar slashy',
    '/$ escaped closing dollar slashy'
].every { dollarSlashy.contains(it) }
It was created to overcome some of the limitations of the slashy string escaping rules. Use it when its escaping rules suit your string contents (typically if it has some slashes you don’t want to escape).

#### 4.8. String summary table
String name

String syntax

Interpolated

Multiline

Escape character

Single-quoted

'…​'



\

Triple-single-quoted

'''…​'''



\

Double-quoted

"…​"



\

Triple-double-quoted

"""…​"""



\

Slashy

/…​/



\

Dollar slashy

$/…​/$



$

#### 4.9. Characters
Unlike Java, Groovy doesn’t have an explicit character literal. However, you can be explicit about making a Groovy string an actual character, by three different means:

char c1 = 'A' 
assert c1 instanceof Character

def c2 = 'B' as char 
assert c2 instanceof Character

def c3 = (char)'C' 
assert c3 instanceof Character
by being explicit when declaring a variable holding the character by specifying the char type
by using type coercion with the as operator
by using a cast to char operation
The first option 1 is interesting when the character is held in a variable, while the other two (2 and 3) are more interesting when a char value must be passed as argument of a method call.

### 5. Numbers
Groovy supports different kinds of integral literals and decimal literals, backed by the usual Number types of Java.

#### 5.1. Integral literals
The integral literal types are the same as in Java:

byte

char

short

int

long

java.math.BigInteger

You can create integral numbers of those types with the following declarations:

// primitive types
byte  b = 1
char  c = 2
short s = 3
int   i = 4
long  l = 5

// infinite precision
BigInteger bi =  6
If you use optional typing by using the def keyword, the type of the integral number will vary: it’ll adapt to the capacity of the type that can hold that number.

For positive numbers:

def a = 1
assert a instanceof Integer

// Integer.MAX_VALUE
def b = 2147483647
assert b instanceof Integer

// Integer.MAX_VALUE + 1
def c = 2147483648
assert c instanceof Long

// Long.MAX_VALUE
def d = 9223372036854775807
assert d instanceof Long

// Long.MAX_VALUE + 1
def e = 9223372036854775808
assert e instanceof BigInteger
As well as for negative numbers:

def na = -1
assert na instanceof Integer

// Integer.MIN_VALUE
def nb = -2147483648
assert nb instanceof Integer

// Integer.MIN_VALUE - 1
def nc = -2147483649
assert nc instanceof Long

// Long.MIN_VALUE
def nd = -9223372036854775808
assert nd instanceof Long

// Long.MIN_VALUE - 1
def ne = -9223372036854775809
assert ne instanceof BigInteger
5.1.1. Alternative non-base 10 representations
Numbers can also be represented in binary, octal, hexadecimal and decimal bases.

Binary literal
Binary numbers start with a 0b prefix:

int xInt = 0b10101111
assert xInt == 175

short xShort = 0b11001001
assert xShort == 201 as short

byte xByte = 0b11
assert xByte == 3 as byte

long xLong = 0b101101101101
assert xLong == 2925l

BigInteger xBigInteger = 0b111100100001
assert xBigInteger == 3873g

int xNegativeInt = -0b10101111
assert xNegativeInt == -175
Octal literal
Octal numbers are specified in the typical format of 0 followed by octal digits.

int xInt = 077
assert xInt == 63

short xShort = 011
assert xShort == 9 as short

byte xByte = 032
assert xByte == 26 as byte

long xLong = 0246
assert xLong == 166l

BigInteger xBigInteger = 01111
assert xBigInteger == 585g

int xNegativeInt = -077
assert xNegativeInt == -63
Hexadecimal literal
Hexadecimal numbers are specified in the typical format of 0x followed by hex digits.

int xInt = 0x77
assert xInt == 119

short xShort = 0xaa
assert xShort == 170 as short

byte xByte = 0x3a
assert xByte == 58 as byte

long xLong = 0xffff
assert xLong == 65535l

BigInteger xBigInteger = 0xaaaa
assert xBigInteger == 43690g

Double xDouble = new Double('0x1.0p0')
assert xDouble == 1.0d

int xNegativeInt = -0x77
assert xNegativeInt == -119
5.2. Decimal literals
The decimal literal types are the same as in Java:

float

double

java.math.BigDecimal

You can create decimal numbers of those types with the following declarations:

// primitive types
float  f = 1.234
double d = 2.345

// infinite precision
BigDecimal bd =  3.456
Decimals can use exponents, with the e or E exponent letter, followed by an optional sign, and an integral number representing the exponent:

assert 1e3  ==  1_000.0
assert 2E4  == 20_000.0
assert 3e+1 ==     30.0
assert 4E-2 ==      0.04
assert 5e-1 ==      0.5
Conveniently for exact decimal number calculations, Groovy chooses java.math.BigDecimal as its decimal number type. In addition, both float and double are supported, but require an explicit type declaration, type coercion or suffix. Even if BigDecimal is the default for decimal numbers, such literals are accepted in methods or closures taking float or double as parameter types.

Decimal numbers can’t be represented using a binary, octal or hexadecimal representation.

#### 5.3. Underscore in literals
When writing long literal numbers, it’s harder on the eye to figure out how some numbers are grouped together, for example with groups of thousands, of words, etc. By allowing you to place underscore in number literals, it’s easier to spot those groups:

long creditCardNumber = 1234_5678_9012_3456L
long socialSecurityNumbers = 999_99_9999L
double monetaryAmount = 12_345_132.12
long hexBytes = 0xFF_EC_DE_5E
long hexWords = 0xFFEC_DE5E
long maxLong = 0x7fff_ffff_ffff_ffffL
long alsoMaxLong = 9_223_372_036_854_775_807L
long bytes = 0b11010010_01101001_10010100_10010010

#### 5.4. Number type suffixes
We can force a number (including binary, octals and hexadecimals) to have a specific type by giving a suffix (see table below), either uppercase or lowercase.

Type	Suffix
BigInteger

G or g

Long

L or l

Integer

I or i

BigDecimal

G or g

Double

D or d

Float

F or f

Examples:

assert 42I == Integer.valueOf('42')
assert 42i == Integer.valueOf('42') // lowercase i more readable
assert 123L == Long.valueOf("123") // uppercase L more readable
assert 2147483648 == Long.valueOf('2147483648') // Long type used, value too large for an Integer
assert 456G == new BigInteger('456')
assert 456g == new BigInteger('456')
assert 123.45 == new BigDecimal('123.45') // default BigDecimal type used
assert .321 == new BigDecimal('.321')
assert 1.200065D == Double.valueOf('1.200065')
assert 1.234F == Float.valueOf('1.234')
assert 1.23E23D == Double.valueOf('1.23E23')
assert 0b1111L.class == Long // binary
assert 0xFFi.class == Integer // hexadecimal
assert 034G.class == BigInteger // octal
5.5. Math operations
Although operators are covered in more detail elsewhere, it’s important to discuss the behavior of math operations and what their resulting types are.

Division and power binary operations aside (covered below),

binary operations between byte, char, short and int result in int

binary operations involving long with byte, char, short and int result in long

binary operations involving BigInteger and any other integral type result in BigInteger

binary operations involving BigDecimal with byte, char, short, int and BigInteger result in BigDecimal

binary operations between float, double and BigDecimal result in double

binary operations between two BigDecimal result in BigDecimal

The following table summarizes those rules:

byte	char	short	int	long	BigInteger	float	double	BigDecimal
byte

int

int

int

int

long

BigInteger

double

double

BigDecimal

char

int

int

int

long

BigInteger

double

double

BigDecimal

short

int

int

long

BigInteger

double

double

BigDecimal

int

int

long

BigInteger

double

double

BigDecimal

long

long

BigInteger

double

double

BigDecimal

BigInteger

BigInteger

double

double

BigDecimal

float

double

double

double

double

double

double

BigDecimal

BigDecimal

Thanks to Groovy’s operator overloading, the usual arithmetic operators work as well with BigInteger and BigDecimal, unlike in Java where you have to use explicit methods for operating on those numbers.

##### 5.5.1. The case of the division operator
The division operators / (and /= for division and assignment) produce a double result if either operand is a float or double, and a BigDecimal result otherwise (when both operands are any combination of an integral type short, char, byte, int, long, BigInteger or BigDecimal).

BigDecimal division is performed with the divide() method if the division is exact (i.e. yielding a result that can be represented within the bounds of the same precision and scale), or using a MathContext with a precision of the maximum of the two operands' precision plus an extra precision of 10, and a scale of the maximum of 10 and the maximum of the operands' scale.

For integer division like in Java, you should use the intdiv() method, as Groovy doesn’t provide a dedicated integer division operator symbol.

##### 5.5.2. The case of the power operator
The power operation is represented by the ** operator, with two parameters: the base and the exponent. The result of the power operation depends on its operands, and the result of the operation (in particular if the result can be represented as an integral value).

The following rules are used by Groovy’s power operation to determine the resulting type:

If the exponent is a decimal value

if the result can be represented as an Integer, then return an Integer

else if the result can be represented as a Long, then return a Long

otherwise return a Double

If the exponent is an integral value

if the exponent is strictly negative, then return an Integer, Long or Double if the result value fits in that type

if the exponent is positive or zero

if the base is a BigDecimal, then return a BigDecimal result value

if the base is a BigInteger, then return a BigInteger result value

if the base is an Integer, then return an Integer if the result value fits in it, otherwise a BigInteger

if the base is a Long, then return a Long if the result value fits in it, otherwise a BigInteger

We can illustrate those rules with a few examples:

// base and exponent are ints and the result can be represented by an Integer
assert    2    **   3    instanceof Integer    //  8
assert   10    **   9    instanceof Integer    //  1_000_000_000

// the base is a long, so fit the result in a Long
// (although it could have fit in an Integer)
assert    5L   **   2    instanceof Long       //  25

// the result can't be represented as an Integer or Long, so return a BigInteger
assert  100    **  10    instanceof BigInteger //  10e20
assert 1234    ** 123    instanceof BigInteger //  170515806212727042875...

// the base is a BigDecimal and the exponent a negative int
// but the result can be represented as an Integer
assert    0.5  **  -2    instanceof Integer    //  4

// the base is an int, and the exponent a negative float
// but again, the result can be represented as an Integer
assert    1    **  -0.3f instanceof Integer    //  1

// the base is an int, and the exponent a negative int
// but the result will be calculated as a Double
// (both base and exponent are actually converted to doubles)
assert   10    **  -1    instanceof Double     //  0.1

// the base is a BigDecimal, and the exponent is an int, so return a BigDecimal
assert    1.2  **  10    instanceof BigDecimal //  6.1917364224

// the base is a float or double, and the exponent is an int
// but the result can only be represented as a Double value
assert    3.4f **   5    instanceof Double     //  454.35430372146965
assert    5.6d **   2    instanceof Double     //  31.359999999999996

// the exponent is a decimal value
// and the result can only be represented as a Double value
assert    7.8  **   1.9  instanceof Double     //  49.542708423868476
assert    2    **   0.1f instanceof Double     //  1.0717734636432956

### 6. Booleans
Boolean is a special data type that is used to represent truth values: true and false. Use this data type for simple flags that track true/false conditions.

Boolean values can be stored in variables, assigned into fields, just like any other data type:

def myBooleanVariable = true
boolean untypedBooleanVar = false
booleanField = true
true and false are the only two primitive boolean values. But more complex boolean expressions can be represented using logical operators.

In addition, Groovy has special rules (often referred to as Groovy Truth) for coercing non-boolean objects to a boolean value.

### 7. Lists
Groovy uses a comma-separated list of values, surrounded by square brackets, to denote lists. Groovy lists are plain JDK java.util.List, as Groovy doesn’t define its own collection classes. The concrete list implementation used when defining list literals are java.util.ArrayList by default, unless you decide to specify otherwise, as we shall see later on.

def numbers = [1, 2, 3]         

assert numbers instanceof List  
assert numbers.size() == 3      
We define a list numbers delimited by commas and surrounded by square brackets, and we assign that list into a variable
The list is an instance of Java’s java.util.List interface
The size of the list can be queried with the size() method, and shows our list contains 3 elements
In the above example, we used a homogeneous list, but you can also create lists containing values of heterogeneous types:

def heterogeneous = [1, "a", true]  
Our list here contains a number, a string and a boolean value
We mentioned that by default, list literals are actually instances of java.util.ArrayList, but it is possible to use a different backing type for our lists, thanks to using type coercion with the as operator, or with explicit type declaration for your variables:

def arrayList = [1, 2, 3]
assert arrayList instanceof java.util.ArrayList

def linkedList = [2, 3, 4] as LinkedList    
assert linkedList instanceof java.util.LinkedList

LinkedList otherLinked = [3, 4, 5]          
assert otherLinked instanceof java.util.LinkedList
We use coercion with the as operator to explicitly request a java.util.LinkedList implementation
We can say that the variable holding the list literal is of type java.util.LinkedList
You can access elements of the list with the [] subscript operator (both for reading and setting values) with positive indices or negative indices to access elements from the end of the list, as well as with ranges, and use the << leftShift operator to append elements to a list:

def letters = ['a', 'b', 'c', 'd']

assert letters[0] == 'a'     
assert letters[1] == 'b'

assert letters[-1] == 'd'    
assert letters[-2] == 'c'

letters[2] = 'C'             
assert letters[2] == 'C'

letters << 'e'               
assert letters[ 4] == 'e'
assert letters[-1] == 'e'

assert letters[1, 3] == ['b', 'd']         
assert letters[2..4] == ['C', 'd', 'e']    
Access the first element of the list (zero-based counting)
Access the last element of the list with a negative index: -1 is the first element from the end of the list
Use an assignment to set a new value for the third element of the list
Use the << leftShift operator to append an element at the end of the list
Access two elements at once, returning a new list containing those two elements
Use a range to access a range of values from the list, from a start to an end element position
As lists can be heterogeneous in nature, lists can also contain other lists to create multidimensional lists:

def multi = [[0, 1], [2, 3]]     
assert multi[1][0] == 2          
Define a list of numbers
Access the second element of the top-most list, and the first element of the inner list

### 8. Arrays
Groovy reuses the list notation for arrays, but to make such literals arrays, you need to explicitly define the type of the array through coercion or type declaration.

String[] arrStr = ['Ananas', 'Banana', 'Kiwi']  

assert arrStr instanceof String[]    
assert !(arrStr instanceof List)

def numArr = [1, 2, 3] as int[]      

assert numArr instanceof int[]       
assert numArr.size() == 3
Define an array of strings using explicit variable type declaration
Assert that we created an array of strings
Create an array of ints with the as operator
Assert that we created an array of primitive ints
You can also create multi-dimensional arrays:

def matrix3 = new Integer[3][3]         
assert matrix3.size() == 3

Integer[][] matrix2                     
matrix2 = [[1, 2], [3, 4]]
assert matrix2 instanceof Integer[][]
You can define the bounds of a new array
Or declare an array without specifying its bounds
Access to elements of an array follows the same notation as for lists:

String[] names = ['Cédric', 'Guillaume', 'Jochen', 'Paul']
assert names[0] == 'Cédric'     

names[2] = 'Blackdrag'          
assert names[2] == 'Blackdrag'
Retrieve the first element of the array
Set the value of the third element of the array to a new value

#### 8.1. Java-style array initialization
Groovy has always supported literal list/array definitions using square brackets and has avoided Java-style curly braces so as not to conflict with closure definitions. In the case where the curly braces come immediately after an array type declaration however, there is no ambiguity with closure definitions, so Groovy 3 and above support that variant of the Java array initialization expression.

Examples:

def primes = new int[] {2, 3, 5, 7, 11}
assert primes.size() == 5 && primes.sum() == 28
assert primes.class.name == '[I'

def pets = new String[] {'cat', 'dog'}
assert pets.size() == 2 && pets.sum() == 'catdog'
assert pets.class.name == '[Ljava.lang.String;'

// traditional Groovy alternative still supported
String[] groovyBooks = [ 'Groovy in Action', 'Making Java Groovy' ]
assert groovyBooks.every{ it.contains('Groovy') }

### 9. Maps
Sometimes called dictionaries or associative arrays in other languages, Groovy features maps. Maps associate keys to values, separating keys and values with colons, and each key/value pairs with commas, and the whole keys and values surrounded by square brackets.

def colors = [red: '#FF0000', green: '#00FF00', blue: '#0000FF']   

assert colors['red'] == '#FF0000'    
assert colors.green  == '#00FF00'    

colors['pink'] = '#FF00FF'           
colors.yellow  = '#FFFF00'           

assert colors.pink == '#FF00FF'
assert colors['yellow'] == '#FFFF00'

assert colors instanceof java.util.LinkedHashMap
We define a map of string color names, associated with their hexadecimal-coded html colors
We use the subscript notation to check the content associated with the red key
We can also use the property notation to assert the color green’s hexadecimal representation
Similarly, we can use the subscript notation to add a new key/value pair
Or the property notation, to add the yellow color
When using names for the keys, we actually define string keys in the map.
Groovy creates maps that are actually instances of java.util.LinkedHashMap.
If you try to access a key which is not present in the map:

assert colors.unknown == null

def emptyMap = [:]
assert emptyMap.anyKey == null
You will retrieve a null result.

In the examples above, we used string keys, but you can also use values of other types as keys:

def numbers = [1: 'one', 2: 'two']

assert numbers[1] == 'one'
Here, we used numbers as keys, as numbers can unambiguously be recognized as numbers, so Groovy will not create a string key like in our previous examples. But consider the case you want to pass a variable in lieu of the key, to have the value of that variable become the key:

def key = 'name'
def person = [key: 'Guillaume']      

assert !person.containsKey('name')   
assert person.containsKey('key')     
The key associated with the 'Guillaume' name will actually be the "key" string, not the value associated with the key variable
The map doesn’t contain the 'name' key
Instead, the map contains a 'key' key
You can also pass quoted strings as well as keys: ["name": "Guillaume"]. This is mandatory if your key string isn’t a valid identifier, for example if you wanted to create a string key containing a dash like in: ["street-name": "Main street"].
When you need to pass variable values as keys in your map definitions, you must surround the variable or expression with parentheses:

person = [(key): 'Guillaume']        

assert person.containsKey('name')    
assert !person.containsKey('key')    
This time, we surround the key variable with parentheses, to instruct the parser we are passing a variable rather than defining a string key
The map does contain the name key
But the map doesn’t contain the key key as before