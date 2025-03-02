
## Language elements

Fortran is <a href="case-insensitive" class="wikilink"
title="case-insensitive">case-insensitive</a>. The convention of writing
Fortran keywords in upper case and all other names in lower case is
adopted in this article; except, by way of contrast, in the input/output
descriptions
(<a href="#Data_transfer" class="wikilink" title="Data transfer">Data
transfer</a> and
<a href="#Operations_on_external_files" class="wikilink"
title="Operations on external files">Operations on external files</a>).

### Basics

The basic component of the Fortran language is its *character set*. Its
members are

-   the letters A ... Z and a ... z (which are equivalent outside a
    character context)
-   the numerals 0 ... 9
-   the underscore \_
-   the special characters
    `= : + blank - * / ( ) [ ] , . $ ' ! " % & ; < > ?`

<a href="Token_(parser)" class="wikilink" title="Token">Tokens</a> that
have a syntactic meaning to the compiler are built from those
components. There are six classes of tokens:

| Label     | `123`                                                |
|-----------|------------------------------------------------------|
| Constant  | `123.456789_long`                                    |
| Keyword   | `ALLOCATABLE`                                        |
| Operator  | `.add.`                                              |
| Name      | `solve_equation` (up to 31 characters, including \_) |
| Separator | `/ ( ) (/ /) [ ] , = => : :: ; %`                    |

From the tokens, <a href="statement_(programming)" class="wikilink"
title="statement">statements</a> are built. These can be coded using the
new free *source form* which does not require positioning in a rigid
column structure:

```f90
FUNCTION string_concat(s1, s2)                             ! This is a comment
   TYPE (string), INTENT(IN) :: s1, s2
   TYPE (string) string_concat
   string_concat%string_data = s1%string_data(1:s1%length) // &
      s2%string_data(1:s2%length)                          ! This is a continuation
   string_concat%length = s1%length + s2%length
END FUNCTION string_concat
```

Note the trailing comments and the trailing continuation mark. There may
be 39 continuation lines, and 132 characters per line. Blanks are
significant. Where a token or character constant is split across two
lines:

```f90
               ...        start_of&
        &_name
               ...   'a very long &
        &string'
```

a leading `&` on the continued line is also required.

### Intrinsic data types

Fortran has five *intrinsic data types*: `INTEGER`, `REAL`, `COMPLEX`,
`LOGICAL` and `CHARACTER`. Each of those types can be additionally
characterized by a *kind*. Kind, basically, defines internal
representation of the type: for the three numeric types, it defines the
precision and range, and for the other two, the specifics of storage
representation. Thus, it is an abstract concept which models the limits
of data types' representation; it is expressed as a member of a set of
whole numbers (e.g. it may be {1, 2, 4, 8} for integers, denoting bytes
of storage), but those values are not specified by the Standard and not
portable. For every type, there is a *default kind*, which is used if no
kind is explicitly specified. For each intrinsic type, there is a
corresponding form of *literal constant*. The numeric types `INTEGER`
and `REAL` can only be signed (there is no concept of sign for type
`COMPLEX`).

#### Literal constants and kinds

##### INTEGER

Integer literal constants of the default kind take the form

```f90
1   0   -999   32767   +10
```

Kind can be defined as a named constant. If the desired range is
±10<sup>kind</sup>, the portable syntax for defining the appropriate
kind, `two_bytes` is

```f90
INTEGER, PARAMETER :: two_bytes = SELECTED_INT_KIND(4)
```

that allows subsequent definition of constants of the form

```f90
-1234_two_bytes   +1_two_bytes
```

Here, `two_bytes` is the kind type parameter; it can also be an explicit
default integer literal constant, like

```f90
-1234_2
```

but such use is non-portable.

The KIND function supplies the value of a kind type parameter:

```f90
KIND(1)            KIND(1_two_bytes)
```

and the `RANGE` function supplies the actual decimal range (so the user
must make the actual mapping to bytes):

```f90
RANGE(1_two_bytes)
```

Also, in <a href="#DATA_statement" class="wikilink"
title="DATA (initialization) statements"><code>DATA</code>
(initialization) statements</a>, binary (B), octal (O) and hexadecimal
(Z) constants may be used (often informally referred to as "BOZ
constants"):

```f90
B'01010101'   O'01234567'   Z'10fa'
```

##### REAL

There are at least two real kindsthe default and one with greater
precision (this replaces

```f90
DOUBLE PRECISION
```

).

```f90
SELECTED_REAL_KIND
```

functions returns the kind number for desired range and precision; for
at least 9 decimal digits of precision and a range of 10<sup>−99</sup>
to 10<sup>99</sup>, it can be specified as:

```f90
INTEGER, PARAMETER :: long = SELECTED_REAL_KIND(9, 99)
```

and literals subsequently specified as

```f90
1.7_long
```

Also, there are the intrinsic functions

```f90
KIND(1.7_long)   PRECISION(1.7_long)   RANGE(1.7_long)
```

that give in turn the kind type value, the actual precision (here at
least 9), and the actual range (here at least 99).

##### COMPLEX

`COMPLEX` data type is built of two integer or real components:

```f90
(1, 3.7_long)
```

##### LOGICAL

There are only two basic values of logical constants: `.TRUE.` and
`.FALSE.`. Here, there may also be different kinds. Logicals don't have
their own kind inquiry functions, but use the kinds specified for
`INTEGER`s; default kind of `LOGICAL` is the same as of INTEGER.

```f90
.FALSE.   .true._one_byte
```

and the `KIND` function operates as expected:

```f90
KIND(.TRUE.)
```

##### CHARACTER

The forms of literal constants for `CHARACTER` data type are

```f90
'A string'   "Another"   'A "quote"'   '''''''
```

(the last being an empty string). Different kinds are allowed (for
example, to distinguish
<a href="ASCII" class="wikilink" title="ASCII">ASCII</a> and
<a href="UNICODE" class="wikilink" title="UNICODE">UNICODE</a> strings),
but not widely supported by compilers. Again, the kind value is given by
the `KIND` function:

```f90
KIND('ASCII')
```

#### Number model and intrinsic functions

The numeric types are based on number models with associated inquiry
functions (whose values are independent of the values of their
arguments; arguments are used only to provide kind). These functions are
important for portable numerical software:

|                  |                                          |
|------------------|------------------------------------------|
| `DIGITS(X)`      | Number of significant digits             |
| `EPSILON(X)`     | Almost negligible compared to one (real) |
| `HUGE(X)`        | Largest number                           |
| `MAXEXPONENT(X)` | Maximum model exponent (real)            |
| `MINEXPONENT(X)` | Minimum model exponent (real)            |
| `PRECISION(X)`   | Decimal precision (real, complex)        |
| `RADIX(X)`       | Base of the model                        |
| `RANGE(X)`       | Decimal exponent range                   |
| `TINY(X)`        | Smallest positive number (real)          |

### Scalar variables

Scalar <a href="Variable_(programming)" class="wikilink"
title="variables">variables</a> corresponding to the five intrinsic
types are specified as follows:

```f90
INTEGER(KIND=2) :: i
REAL(KIND=long) :: a
COMPLEX         :: current
LOGICAL         :: Pravda
CHARACTER(LEN=20) :: word
CHARACTER(LEN=2, KIND=Kanji) :: kanji_word
```

where the optional `KIND` parameter specifies a non-default kind, and
the `::` notation delimits the type and attributes from variable name(s)
and their optional initial values, allowing full variable specification
and initialization to be typed in one statement (in previous standards,
attributes and initializers had to be declared in several statements).
While it is not required in above examples (as there are no additional
attributes and initialization), most Fortran-90 programmers acquire the
habit to use it everywhere.

```f90
LEN=
```

specifier is applicable only to `CHARACTER`s and specifies the string
length (replacing the older `*len` form). The explicit `KIND=` and
`LEN=` specifiers are optional:

```f90
CHARACTER(2, Kanji) :: kanji_word
```

works just as well.

There are some other interesting character features. Just as a substring
as in

```f90
CHARACTER(80) :: line   
... = line(i:i)                     ! substring
```

was previously possible, so now is the substring

```f90
'0123456789'(i:i)
```

Also, zero-length strings are allowed:

```f90
line(i:i-1)       ! zero-length string
```

Finally, there is a set of intrinsic character functions, examples being

|            |                              |
|------------|------------------------------|
| `ACHAR`    | `IACHAR` (for ASCII set)     |
| `ADJUSTL`  | `ADJUSTR`                    |
| `LEN_TRIM` | `INDEX(s1, s2, BACK=.TRUE.)` |
| `REPEAT`   | `SCAN`(for one of a set)     |
| `TRIM`     | `VERIFY`(for all of a set)   |

### Derived data types

For derived data types, the form of the type must be defined first:

```f90
TYPE person
   CHARACTER(10) name
   REAL          age
END TYPE person
```

and then, variables of that type can be defined:

```f90
TYPE(person) you, me
```

To select components of a derived type, `%` qualifier is used:

```f90
you%age
```

Literal constants of derived types have the form
*`TypeName(1stComponentLiteral, 2ndComponentLiteral, ...)`*:

```f90
you = person('Smith', 23.5)
```

which is known as a *structure constructor*. Definitions may refer to a
previously defined type:

```f90
TYPE point
   REAL x, y
END TYPE point
TYPE triangle
   TYPE(point) a, b, c
END TYPE triangle
```

and for a variable of type triangle, as in

```f90
TYPE(triangle) t
```

each component of type `point` is accessed as

```f90
t%a   t%b   t%c
```

which, in turn, have ultimate components of type real:

```f90
t%a%x   t%a%y   t%b%x   etc.
```

(Note that the `%` qualifier was chosen rather than dot (`.`) because of
potential ambiguity with operator notation, like `.OR.`).

### Implicit and explicit typing

Unless specified otherwise, all variables starting with letters I, J, K,
L, M and N are default `INTEGER`s, and all others are default `REAL`;
other data types must be explicitly declared. This is known as *implicit
typing* and is a heritage of early FORTRAN days. Those defaults can be
overridden by *`IMPLICIT TypeName (CharacterRange)`* statements, like:

```f90
IMPLICIT COMPLEX(Z)
IMPLICIT CHARACTER(A-B)
IMPLICIT REAL(C-H,N-Y)
```

However, it is a good practice to explicitly type all variables, and
this can be forced by inserting the statement

```f90
IMPLICIT NONE
```

at the beginning of each program unit.

### Arrays

Arrays are considered to be variables in their own right. Every array is
characterized by its
<a href="type_(computer_programming)" class="wikilink"
title="type">type</a>,
<a href="rank_(computer_programming)" class="wikilink"
title="rank">rank</a>, and *shape* (which defines the extents of each
dimension). Bounds of each dimension are by default 1 and *size*, but
arbitrary bounds can be explicitly specified. `DIMENSION` keyword is
optional and considered an attribute; if omitted, the array shape must
be specified after array-variable name. For example,

```f90
REAL:: a(10)
INTEGER, DIMENSION(0:100, -50:50) :: map
```

declares two arrays, rank-1 and rank-2, whose elements are in
<a href="column-major_order" class="wikilink"
title="column-major order">column-major order</a>. Elements are, for
example,

```f90
a(1)  a(i*j)
```

and are scalars. The subscripts may be any scalar integer expression.

*Sections* are parts of the array variables, and are arrays themselves:

```f90
a(i:j)               ! rank one
map(i:j, k:l:m)      ! rank two
a(map(i, k:l))       ! vector subscript
a(3:2)               ! zero length
```

Whole arrays and array sections are array-valued objects. Array-valued
constants (constructors) are available, enclosed in `(/ ... /)`:

```f90
(/ 1, 2, 3, 4 /)
(/ ( (/ 1, 2, 3 /), i = 1, 4) /)
(/ (i, i = 1, 9, 2) /)
(/ (0, i = 1, 100) /)
(/ (0.1*i, i = 1, 10) /)
```

making use of an implied-DO loop notation. Fortran 2003 allows the use
of brackets: `[1, 2, 3, 4]` and `[([1,2,3], i=1,4)]` instead of the
first two examples above, and many compilers support this now. A derived
data type may, of course, contain array components:

```f90
TYPE triplet
   REAL, DIMENSION(3) :: vertex
END TYPE triplet
TYPE(triplet), DIMENSION(4) :: t
```

so that

-   ```f90
    t(2)
    ```

    is a scalar (a structure)

-   ```f90
    t(2)%vertex
    ```

    is an array component of a scalar

### Data initialization

Variables can be given initial values as specified in a specification
statement:

```f90
REAL, DIMENSION(3) :: a = (/ 0.1, 0.2, 0.3 /)
```

and a default initial value can be given to the component of a derived
data type:

```f90
TYPE triplet
   REAL, DIMENSION(3) :: vertex = 0.0
END TYPE triplet
```

When local variables are initialized within a procedure they implicitly
acquire the SAVE attribute:

```f90
REAL, DIMENSION(3) :: point = (/ 0.0, 1.0, -1.0 /)
```

This declaration is equivalent to

```f90
REAL, DIMENSION(3), SAVE :: point = (/ 0.0, 1.0, -1.0 /)
```

for local variables within a subroutine or function. The SAVE attribute
causes local variables to retain their value after a procedure call and
then to initialize the variable to the saved value upon returning to the
procedure.

#### PARAMETER attribute

A named constant can be specified directly by adding the `PARAMETER`
attribute and the constant values to a type statement:

```f90
REAL, DIMENSION(3), PARAMETER :: field = (/ 0., 1., 2. /)
TYPE(triplet), PARAMETER :: t = triplet( (/ 0., 0., 0. /) )
```

#### DATA statement

The `DATA` statement can be used for scalars and also for arrays and
variables of derived type. It is also the only way to initialise just
parts of such objects, as well as to initialise to binary, octal or
hexadecimal values:

```f90
TYPE(triplet) :: t1, t2
DATA t1/triplet( (/ 0., 1., 2. /) )/, t2%vertex(1)/123./
DATA array(1:64) / 64*0/
DATA i, j, k/ B'01010101', O'77', Z'ff'/
```

#### Initialization expressions

The values used in `DATA` and `PARAMETER` statements, or with these
attributes, are constant expressions that may include references to:
array and structure constructors, elemental intrinsic functions with
integer or character arguments and results, and the six transformational
functions `REPEAT, SELECTED_INT_KIND, TRIM, SELECTED_REAL_KIND, RESHAPE`
and `TRANSFER` (see <a href="#Intrinsic_procedures" class="wikilink"
title="Intrinsic procedures">Intrinsic procedures</a>):

```f90
INTEGER, PARAMETER :: long = SELECTED_REAL_KIND(12),   &
                      array(3) = (/ 1, 2, 3 /)
```

### Specification expressions

It is possible to specify details of variables using any non-constant,
scalar, integer expression that may also include inquiry function
references:

```f90
SUBROUTINE s(b, m, c)
   USE mod                                 ! contains a
   REAL, DIMENSION(:, :)             :: b
   REAL, DIMENSION(UBOUND(b, 1) + 5) :: x
   INTEGER                           :: m
   CHARACTER(LEN=*)                  :: c
   CHARACTER(LEN= m + LEN(c))        :: cc
   REAL (SELECTED_REAL_KIND(2*PRECISION(a))) :: z
```

