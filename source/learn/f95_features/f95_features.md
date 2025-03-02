
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

## Expressions and assignments

### Scalar numeric

The usual arithmetic operators are available `+, -, *, /, **` (given
here in increasing order of precedence).

Parentheses are used to indicate the order of evaluation where
necessary:

```f90
a*b + c     ! * first
a*(b + c)   ! + first
```

The rules for *scalar numeric* expressions and assignments accommodate
the non-default kinds. Thus, the mixed-mode numeric expression and
assignment rules incorporate different kind type parameters in an
expected way:

```f90
real2 = integer0 + real1
```

converts `integer0` to a real value of the same kind as `real1`; the
result is of same kind, and is converted to the kind of `real2` for
assignment.

These functions are available for controlled
<a href="rounding" class="wikilink" title="rounding">rounding</a> of
real numbers to integers:

-   `NINT`: round to nearest integer, return integer result
-   `ANINT`: round to nearest integer, return real result
-   `INT`: truncate (round towards zero), return integer result
-   `AINT`: truncate (round towards zero), return real result
-   `CEILING`: smallest integral value not less than argument (round up)
    (Fortran-90)
-   `FLOOR`: largest integral value not greater than argument (round
    down) (Fortran-90)

### Scalar relational operations

For *scalar relational* operations of numeric types, there is a set of
built-in operators:

`<    <=    ==   /=   >   >=`  
`.LT. .LE. .EQ. .NE. .GT. .GE.`

(the forms above are new to Fortran-90, and older equivalent forms are
given below them). Example expressions:

```f90
a < b .AND. i /= j      ! for numeric variables
flag = a == b           ! for logical variable flags
```

### Scalar characters

In the case of *scalar characters* and given

```f90
CHARACTER(8) result
```

it is legal to write

```f90
result(3:5) = result(1:3)    ! overlap allowed
result(3:3) = result(3:2)    ! no assignment of null string
```

Concatenation is performed by the operator '//'.

```f90
result = 'abcde'//'123'
filename = result//'.dat'
```

### Derived-data types

No built-in operations (except assignment, defined on component-by
component basis) exist between *derived data types* mutually or with
intrinsic types. The meaning of existing or user-specified operators can
be (re)defined though:

```f90
TYPE string80
   INTEGER       length
   CHARACTER(80) value
END TYPE string80
CHARACTER::    char1, char2, char3
TYPE(string80):: str1,  str2,  str3
```

we can write

```f90
str3  = str1//str2       ! must define operation
str3  = str1.concat.str2 ! must define operation
char3 = char2//char3     ! intrinsic operator only
str3  = char1            ! must define assignment
```

Notice the "<a href="Operator_overloading" class="wikilink"
title="overloaded">overloaded</a>" use of the intrinsic symbol `//` and
the named operator, `.concat.` . A difference between the two cases is
that, for an intrinsic operator token, the usual precedence rules apply,
whereas for named operators, precedence is the highest as a unary
operator or the lowest as a binary one. In

```f90
vector3 = matrix    *    vector1  + vector2
vector3 =(matrix .times. vector1) + vector2
```

the two expressions are equivalent only if appropriate parentheses are
added as shown. In each case there must be defined, in a
<a href="#Modules" class="wikilink" title="module">module</a>,
procedures defining the operator and assignment, and corresponding
operator-procedure association, as follows:

```f90
INTERFACE OPERATOR(//) !Overloads the // operator as invoking string_concat procedure
  MODULE PROCEDURE string_concat
END INTERFACE
```

The string concatenation function is a more elaborated version of that
shown already in
<a href="#Basics" class="wikilink" title="Basics">Basics</a>. Note that
in order to handle the error condition that arises when the two strings
together exceed the preset 80-character limit, it would be safer to use
a subroutine to perform the concatenation (in this case
operator-overloading would not be applicable.)

```f90
MODULE string_type
   IMPLICIT NONE
   TYPE string80
      INTEGER length
      CHARACTER(LEN=80)   :: string_data
   END TYPE string80
   INTERFACE ASSIGNMENT(=)
      MODULE PROCEDURE c_to_s_assign, s_to_c_assign
   END INTERFACE
   INTERFACE OPERATOR(//)
      MODULE PROCEDURE string_concat
   END INTERFACE
CONTAINS
   SUBROUTINE c_to_s_assign(s, c)
      TYPE (string80), INTENT(OUT)    :: s
      CHARACTER(LEN=*), INTENT(IN)  :: c
      s%string_data = c
      s%length = LEN(c)
   END SUBROUTINE c_to_s_assign
   SUBROUTINE s_to_c_assign(c, s)
      TYPE (string80), INTENT(IN)     :: s
      CHARACTER(LEN=*), INTENT(OUT) :: c
      c = s%string_data(1:s%length)
   END SUBROUTINE s_to_c_assign
   TYPE(string80) FUNCTION string_concat(s1, s2)
      TYPE(string80), INTENT(IN) :: s1, s2
      TYPE(string80) :: s
      INTEGER :: n1, n2
      CHARACTER(160) :: ctot
      n1 = LEN_TRIM(s1%string_data)
      n2 = LEN_TRIM(s2%string_data)
      IF (n1+n2 <= 80) then
         s%string_data = s1%string_data(1:n1)//s2%string_data(1:n2)
      ELSE  ! This is an error condition which should be handled - for now just truncate
         ctot = s1%string_data(1:n1)//s2%string_data(1:n2)
         s%string_data = ctot(1:80)
      END IF
      s%length = LEN_TRIM(s%string_data)
      string_concat = s
   END FUNCTION string_concat
END MODULE string_type

PROGRAM main
   USE string_type
   TYPE(string80) :: s1, s2, s3
   CALL c_to_s_assign(s1,'My name is')
   CALL c_to_s_assign(s2,' Linus Torvalds')
   s3 = s1//s2
   WRITE(*,*) 'Result: ',s3%string_data
   WRITE(*,*) 'Length: ',s3%length
END PROGRAM
```

Defined operators such as these are required for the expressions that
are allowed also in structure constructors (see
<a href="#Derived-data_types" class="wikilink"
title="Derived-data types">Derived-data types</a>):

```f90
str1 = string(2, char1//char2)  ! structure constructor
```

### Arrays

In the case of arrays then, as long as they are of the same shape
(conformable), operations and assignments are extended in an obvious
way, on an element-by-element basis. For example, given declarations of

```f90
REAL, DIMENSION(10, 20) :: a, b, c
REAL, DIMENSION(5)      :: v, w
LOGICAL                    flag(10, 20)
```

it can be written:

```f90
a = b                                       ! whole array assignment
c = a/b                                     ! whole array division and assignment
c = 0.                                      ! whole array assignment of scalar value
w = v + 1.                                  ! whole array addition to scalar value
w = 5/v + a(1:5, 5)                         ! array division, and addition to section
flag = a==b                                 ! whole array relational test and assignment
c(1:8, 5:10) = a(2:9, 5:10) + b(1:8, 15:20) ! array section addition and assignment
v(2:5) = v(1:4)                             ! overlapping section assignment
```

The order of expression evaluation is not specified in order to allow
for optimization on parallel and vector machines. Of course, any
operators for arrays of derived type must be defined.

Some real intrinsic functions that are useful for numeric computations
are

-   ```f90
    CEILING
    ```

-   ```f90
    FLOOR
    ```

-   ```f90
    MODULO
    ```

    (also integer)

-   ```f90
    EXPONENT
    ```

-   ```f90
    FRACTION
    ```

-   ```f90
    NEAREST
    ```

-   ```f90
    RRSPACING
    ```

-   ```f90
    SPACING
    ```

-   ```f90
    SCALE
    ```

-   ```f90
    SET_EXPONENT
    ```

These are array valued for array arguments (elemental), like all
<a href="FORTRAN_77" class="wikilink" title="FORTRAN 77">FORTRAN 77</a>
functions (except LEN):

-   ```f90
    INT
    ```

-   ```f90
    REAL
    ```

-   ```f90
    CMPLX
    ```

-   ```f90
    AINT
    ```

-   ```f90
    ANINT
    ```

-   ```f90
    NINT
    ```

-   ```f90
    ABS
    ```

-   ```f90
    MOD
    ```

-   ```f90
    SIGN
    ```

-   ```f90
    DIM
    ```

-   ```f90
    MAX
    ```

-   ```f90
    MIN
    ```

Powers, logarithms, and trigonometric functions

-   ```f90
    SQRT
    ```

-   ```f90
    EXP
    ```

-   ```f90
    LOG
    ```

-   ```f90
    LOG10
    ```

-   ```f90
    SIN
    ```

-   ```f90
    COS
    ```

-   ```f90
    TAN
    ```

-   ```f90
    ASIN
    ```

-   ```f90
    ACOS
    ```

-   ```f90
    ATAN
    ```

-   ```f90
    ATAN2
    ```

-   ```f90
    SINH
    ```

-   ```f90
    COSH
    ```

-   ```f90
    TANH
    ```

Complex numbers:

-   ```f90
    AIMAG
    ```

-   ```f90
    CONJG
    ```

The following are for characters:

-   ```f90
    LGE
    ```

-   ```f90
    LGT
    ```

-   ```f90
    LLE
    ```

-   ```f90
    LLT
    ```

-   ```f90
    ICHAR
    ```

-   ```f90
    CHAR
    ```

-   ```f90
    INDEX
    ```

## Control statements

### Branching and conditions

The simple `GO TO` *label* exists, but is usually avoided in most cases,
a more specific branching construct will accomplish the same logic with
more clarity.

The simple conditional test is the `IF` statement:

```f90
IF (a > b) x = y
```

A full-blown `IF` construct is illustrated by

```f90
IF (i < 0) THEN
   IF (j < 0) THEN
      x = 0.
   ELSE
      z = 0.
   END IF
ELSE IF (k < 0) THEN
   z = 1.
ELSE
   x = 1.
END IF
```

### CASE construct

The `CASE` construct is a replacement for the computed `GOTO`, but is
better structured and does not require the use of statement labels:

```f90
SELECT CASE (number)       ! number of type integer
CASE (:-1)                 ! all values below 0
   n_sign = -1
CASE (0)                   ! only 0
   n_sign = 0
CASE (1:)                  ! all values above 0
   n_sign = 1
END SELECT
```

Each `CASE` selector list may contain a list and/or range of integers,
character or logical constants, whose values may not overlap within or
between selectors:

```f90
CASE (1, 2, 7, 10:17, 23)
```

A default is available:

```f90
CASE DEFAULT
```

There is only one evaluation, and only one match.

### DO construct

A simplified but sufficient form of the `DO` construct is illustrated by

```f90
outer: DO
inner:    DO i = j, k, l      ! from j to k in steps of l (l is optional)
             :
             IF (...) CYCLE
             :
             IF (...) EXIT outer
             :
          END DO inner
       END DO outer
```

where we note that loops may be optionally named so that any EXIT or
CYCLE statement may specify which loop is meant.

Many, but not all, simple loops can be replaced by array expressions and
assignments, or by new intrinsic functions. For instance

```f90
tot = 0.
DO i = m, n
   tot = tot + a(i)
END DO
```

becomes simply

```f90
tot = SUM( a(m:n) )
```

## Program units and procedures

### Definitions

In order to discuss this topic we need some definitions. In logical
terms, an executable program consists of one *main program* and zero or
more *subprograms* (or *procedures*) - these do something. Subprograms
are either *functions*or *subroutines*, which are either *external,
internal* or *module* subroutines. (External subroutines are what we
knew from FORTRAN 77.)

From an organizational point of view, however, a complete program
consists of *program units*. These are either *main programs, external
subprograms* or *modules* and can be separately compiled.

An example of a main (and complete) program is

```f90
PROGRAM test
   PRINT *, 'Hello world!'
END PROGRAM test
```

An example of a main program and an external subprogram, forming an
executable program, is

```f90
PROGRAM test
   CALL print_message
END PROGRAM test
SUBROUTINE print_message
   PRINT *, 'Hello world!'
END SUBROUTINE print_message
```

The form of a function is

```f90
FUNCTION name(arg1, arg2) ! zero or more arguments
   :                     
   name = ...
   :
END FUNCTION name
```

The form of reference of a function is

```f90
x = name(a, b)
```

### Internal procedures

An internal subprogram is one *contained* in another (at a maximum of
one level of nesting) and provides a replacement for the statement
function:

```f90
SUBROUTINE outer
   REAL x, y
   :
CONTAINS
   SUBROUTINE inner
      REAL y
      y = x + 1.
      :
   END SUBROUTINE inner     ! SUBROUTINE mandatory
END SUBROUTINE outer
```

We say that `outer` is the *host* of `inner`, and that `inner` obtains
access to entities in `outer` by *host association* (e.g. to `x`),
whereas `y` is a *local* variable to `inner`.

The *scope* of a named entity is a *scoping unit*, here `outer` less
`inner`, and `inner`.

The names of program units and external procedures are *global*, and the
names of implied-DO variables have a scope of the statement that
contains them.

### Modules

Modules are used to package

-   global data (replaces COMMON and BLOCK DATA from Fortran 77);
-   type definitions (themselves a scoping unit);
-   subprograms (which among other things replaces the use of ENTRY from
    Fortran 77);
-   interface blocks (another scoping unit, see
    <a href="#Interface_blocks" class="wikilink"
    title="Interface blocks">Interface blocks</a>);
-   namelist groups (see any textbook).

An example of a module containing a type definition, interface block and
function subprogram is

```f90
MODULE interval_arithmetic
   TYPE interval
      REAL lower, upper
   END TYPE interval
   INTERFACE OPERATOR(+)
       MODULE PROCEDURE add_intervals
   END INTERFACE
   :
CONTAINS
   FUNCTION add_intervals(a,b)
      TYPE(interval), INTENT(IN) :: a, b
      TYPE(interval) add_intervals
      add_intervals%lower = a%lower + b%lower
      add_intervals%upper = a%upper + b%upper
   END FUNCTION add_intervals             ! FUNCTION mandatory
   :
END MODULE interval_arithmetic
```

and the simple statement

```f90
     
USE interval_arithmetic
```

provides *use association* to all the module's entities. Module
subprograms may, in turn, contain internal subprograms.

### Controlling accessibility

The `PUBLIC` and `PRIVATE` attributes are used in specifications in
modules to limit the scope of entities. The attribute form is

```f90
REAL, PUBLIC     :: x, y, z           ! default
INTEGER, PRIVATE :: u, v, w
```

and the statement form is

```f90
PUBLIC  :: x, y, z, OPERATOR(.add.)
PRIVATE :: u, v, w, ASSIGNMENT(=), OPERATOR(*)
```

The statement form has to be used to limit access to operators, and can
also be used to change the overall default:

```f90
PRIVATE                        ! sets default for module
PUBLIC  :: only_this
```

For derived types there are three possibilities: the type and its
components are all PUBLIC, the type is PUBLIC and its components PRIVATE
(the type only is visible and one can change its details easily), or all
of it is PRIVATE (for internal use in the module only):

```f90
MODULE mine
   PRIVATE
   TYPE, PUBLIC :: list
      REAL x, y
      TYPE(list), POINTER :: next
   END TYPE list
   TYPE(list) :: tree
   :
END MODULE mine
```

The `USE` statement's purpose is to gain access to entities in a module.
It has options to resolve name clashes if an imported name is the same
as a local one:

```f90
USE mine, local_list => list
```

or to restrict the used entities to a specified set:

```f90
USE mine, ONLY : list
```

These may be combined:

```f90
USE mine, ONLY : local_list => list
```

### Arguments

We may specify the intent of dummy arguments:

```f90
SUBROUTINE shuffle (ncards, cards)
  INTEGER, INTENT(IN)  :: ncards
  INTEGER, INTENT(OUT), DIMENSION(ncards) :: cards
```

Also, INOUT is possible: here the actual argument must be a variable
(unlike the default case where it may be a constant).

Arguments may be optional:

```f90
SUBROUTINE mincon(n, f, x, upper, lower, equalities, inequalities, convex, xstart)
   REAL, OPTIONAL, DIMENSION :: upper, lower
   :
   IF (PRESENT(lower)) THEN   ! test for presence of actual argument
   :
```

allows us to call `mincon` by

```f90
CALL mincon (n, f, x, upper)
```

Arguments may be keyword rather than positional (which come first):

```f90
CALL mincon(n, f, x, equalities=0, xstart=x0)
```

Optional and keyword arguments are handled by explicit interfaces, that
is with internal or module procedures or with interface blocks.

### Interface blocks

Any reference to an internal or module subprogram is through an
interface that is 'explicit' (that is, the compiler can see all the
details). A reference to an external (or dummy) procedure is usually
'implicit' (the compiler assumes the details). However, we can provide
an explicit interface in this case too. It is a copy of the header,
specifications and END statement of the procedure concerned, either
placed in a module or inserted directly:

```f90
REAL FUNCTION minimum(a, b, func)
  ! returns the minimum value of the function func(x)
  ! in the interval (a,b)
  REAL, INTENT(in) :: a, b
  INTERFACE
    REAL FUNCTION func(x)
      REAL, INTENT(IN) :: x
    END FUNCTION func
  END INTERFACE
  REAL f,x
  :
  f = func(x)   ! invocation of the user function.
  :
END FUNCTION minimum
```

An explicit interface is obligatory for

-   optional and keyword arguments;
-   POINTER and TARGET arguments (see
    <a href="#Pointers" class="wikilink" title="Pointers">Pointers</a>);
-   POINTER function result;
-   new-style array arguments and array functions
    (<a href="#Array_handling" class="wikilink" title="Array handling">Array
    handling</a>).

It allows full checks at compile time between actual and dummy
arguments.

**In general, the best way to ensure that a procedure interface is
explicit is either to place the procedure concerned in a module or to
use it as an internal procedure.**

### Overloading and generic interfaces

Interface blocks provide the mechanism by which we are able to define
generic names for specific procedures:

```f90
INTERFACE gamma                   ! generic name
   FUNCTION sgamma(X)              ! specific name
      REAL (SELECTED_REAL_KIND( 6)) sgamma, x
   END
   FUNCTION dgamma(X)              ! specific name
      REAL (SELECTED_REAL_KIND(12)) dgamma, x
   END
END INTERFACE
```

where a given set of specific names corresponding to a generic name must
all be of functions or all of subroutines. If this interface is within a
module, then it is simply

```f90
INTERFACE gamma
   MODULE PROCEDURE sgamma, dgamma
END INTERFACE
```

We can use existing names, e.g. SIN, and the compiler sorts out the
correct association.

We have already seen the use of interface blocks for defined operators
and assignment (see
<a href="#Modules" class="wikilink" title="Modules">Modules</a>).

### Recursion

Indirect recursion is useful for multi-dimensional integration. For

```f90
volume = integrate(fy, ybounds)
```

We might have

```f90
RECURSIVE FUNCTION integrate(f, bounds)
   ! Integrate f(x) from bounds(1) to bounds(2)
   REAL integrate
   INTERFACE
      FUNCTION f(x)
         REAL f, x
      END FUNCTION f
   END INTERFACE
   REAL, DIMENSION(2), INTENT(IN) :: bounds
   :
END FUNCTION integrate
```

and to integrate *f(x, y)* over a rectangle:

```f90
FUNCTION fy(y)
   USE func           ! module func contains function f
   REAL fy, y
   yval = y
   fy = integrate(f, xbounds)
END
```

Direct recursion is when a procedure calls itself, as in

```f90
RECURSIVE FUNCTION factorial(n) RESULT(res)
   INTEGER res, n
   IF(n.EQ.0) THEN
      res = 1
   ELSE
      res = n*factorial(n-1)
   END IF
END
```

Here, we note the `RESULT` clause and termination test.

### Pure procedures

This is a feature for parallel computing.

In <a href="#The_FORALL_statement_and_construct" class="wikilink"
title="the FORALL statement and construct">the FORALL statement and
construct</a>, any side effects in a function can impede optimization on
a parallel processor the order of execution of the assignments could
affect the results. To control this situation, we add the `PURE` keyword
to the `SUBROUTINE` or `FUNCTION` statementan assertion that the
procedure (expressed simply):

-   alters no global variable,
-   performs no I/O,
-   has no saved variables (variables with the `SAVE` attribute that
    retains values between invocations), and
-   for functions, does not alter any of its arguments.

A compiler can check that this is the case, as in

```f90
PURE FUNCTION calculate (x)
```

All the intrinsic functions are pure.

## Array handling

Array handling is included in Fortran for two main reasons:

-   the notational convenience it provides, bringing the code closer to
    the underlying mathematical form;
-   for the additional optimization opportunities it gives compilers
    (although there are plenty of opportunities for degrading
    optimization too!).

At the same time, major extensions of the functionality in this area
have been added. We have already met whole arrays above
<a href="#Arrays" class="wikilink" title="#Arrays 1">#Arrays 1</a> and
here
<a href="#Arrays_2" class="wikilink" title="#Arrays 2">#Arrays 2</a> -
now we develop the theme.

### Zero-sized arrays

A zero-sized array is handled by Fortran as a legitimate object, without
special coding by the programmer. Thus, in

```f90
DO i = 1,n
   x(i) = b(i) / a(i, i)
   b(i+1:n) = b(i+1:n) - a(i+1:n, i) * x(i)
END DO
```

no special code is required for the final iteration where `i = n`. We
note that a zero-sized array is regarded as being defined; however, an
array of shape (0,2) is not conformable with one of shape (0,3), whereas

```f90
x(1:0) = 3
```

is a valid 'do nothing' statement.

### Assumed-shape arrays

These are an extension and replacement for assumed-size arrays. Given an
actual argument like:

```f90
REAL, DIMENSION(0:10, 0:20) :: a
   :
CALL sub(a)
```

the corresponding dummy argument specification defines only the type and
rank of the array, not its shape. This information has to be made
available by an explicit interface, often using an interface block (see
<a href="#Interface_blocks" class="wikilink"
title="Interface blocks">Interface blocks</a>). Thus we write just

```f90
SUBROUTINE sub(da)
   REAL, DIMENSION(:, :) :: da
```

and this is as if `da` were dimensioned (11,21). However, we can specify
any lower bound and the array maps accordingly.

```f90
REAL, DIMENSION(0:, 0:) :: da
```

The shape, not bounds, is passed, where the default lower bound is 1 and
the default upper bound is the corresponding extent.

### Automatic arrays

A partial replacement for the uses to which `EQUIVALENCE` was put is
provided by this facility, useful for local, temporary arrays, as in

```f90
SUBROUTINE swap(a, b)
   REAL, DIMENSION(:)       :: a, b
   REAL, DIMENSION(SIZE(a)) :: work
   work = a
   a = b
   b = work
END SUBROUTINE swap
```

The actual storage is typically maintained on a stack.

### ALLOCATABLE and ALLOCATE

Fortran provides dynamic allocation of storage; it relies on a heap
storage mechanism (and replaces another use of `EQUIVALENCE`). An
example for establishing a work array for a whole program is

```f90
MODULE work_array
   INTEGER n
   REAL, DIMENSION(:,:,:), ALLOCATABLE :: work
END MODULE
PROGRAM main
   USE work_array
   READ (input, *) n
   ALLOCATE(work(n, 2*n, 3*n), STAT=status)
   :
   DEALLOCATE (work)
```

The work array can be propagated through the whole program via a `USE`
statement in each program unit. We may specify an explicit lower bound
and allocate several entities in one statement. To free dead storage we
write, for instance,

```f90
DEALLOCATE(a, b)
```

Deallocation of arrays is automatic when they go out of scope.

### Elemental operations, assignments and procedures

We have already met whole array assignments and operations:

```f90
REAL, DIMENSION(10) :: a, b
a = 0.          ! scalar broadcast; elemental assignment
b = SQRT(a)     ! intrinsic function result as array object
```

In the second assignment, an intrinsic function returns an array-valued
result for an array-valued argument. We can write array-valued functions
ourselves (they require an explicit interface):

```f90
PROGRAM test
   REAL, DIMENSION(3) :: a = (/ 1., 2., 3./),       &
                         b = (/ 2., 2., 2. /),  r
   r = f(a, b)
   PRINT *, r
CONTAINS
   FUNCTION f(c, d)
   REAL, DIMENSION(:) :: c, d
   REAL, DIMENSION(SIZE(c)) :: f
   f = c*d        ! (or some more useful function of c and d)
   END FUNCTION f
END PROGRAM test
```

Elemental procedures are specified with scalar dummy arguments that may
be called with array actual arguments. In the case of a function, the
shape of the result is the shape of the array arguments.

Most intrinsic functions are elemental and Fortran 95 extends this
feature to non-intrinsic procedures, thus providing the effect of
writing, in Fortran 90, 22 different versions, for ranks 0-0, 0-1, 1-0,
1-1, 0-2, 2-0, 2-2, ... 7-7, and is further an aid to optimization on
parallel processors. An elemental procedure must be pure.

```f90
ELEMENTAL SUBROUTINE swap(a, b)
   REAL, INTENT(INOUT)  :: a, b
   REAL                 :: work
   work = a
   a = b
   b = work
END SUBROUTINE swap
```

The dummy arguments cannot be used in specification expressions (see
<a href="#Specification_expressions" class="wikilink"
title="above">above</a>) except as arguments to certain intrinsic
functions (`BIT_SIZE`, `KIND`, `LEN`, and the numeric inquiry ones, (see
<a href="#Intrinsic_data_types" class="wikilink" title="below">below</a>).

### WHERE

Often, we need to mask an assignment. This we can do using the `WHERE`,
either as a statement:

```f90
WHERE (a /= 0.0) a = 1.0/a  ! avoid division by 0
```

(note: the test is element-by-element, not on whole array), or as a
construct:

```f90
WHERE (a /= 0.0)
   a = 1.0/a
   b = a             ! all arrays same shape
END WHERE
```

or

```f90
WHERE (a /= 0.0)
   a = 1.0/a
ELSEWHERE
   a = HUGE(a)
END WHERE
```

Further:

-   it is permitted to mask not only the `WHERE` statement of the
    `WHERE` construct, but also any `ELSEWHERE` statement that it
    contains;
-   a `WHERE` construct may contain any number of masked `ELSEWHERE`
    statements but at most one `ELSEWHERE` statement without a mask, and
    that must be the final one;
-   `WHERE` constructs may be nested within one another, just `FORALL`
    constructs;
-   a `WHERE` assignment statement is permitted to be a defined
    assignment, provided that it is elemental;
-   a `WHERE` construct may be named in the same way as other
    constructs.

### The FORALL statement and construct

When a `DO` construct is executed, each successive iteration is
performed in order and one after the otheran impediment to optimization
on a parallel processor.

```f90
FORALL(i = 1:n) a(i, i) = x(i)
```

where the individual assignments may be carried out in any order, and
even simultaneously. The `FORALL` may be considered to be an array
assignment expressed with the help of indices.

```f90
FORALL(i=1:n, j=1:n, y(i,j)/=0.) x(j,i) = 1.0/y(i,j)
```

with masking condition.

The `FORALL` construct allows several assignment statements to be
executed in order.

```f90
a(2:n-1,2:n-1) = a(2:n-1,1:n-2) + a(2:n-1,3:n) + a(1:n-2,2:n-1) + a(3:n,2:n-1)
b(2:n-1,2:n-1) = a(2:n-1,2:n-1)
```

is equivalent to the array assignments

```f90
FORALL(i = 2:n-1, j = 2:n-1)
   a(i,j) = a(i,j-1) + a(i,j+1) + a(i-1,j) + a(i+1,j)
   b(i,j) = a(i,j)
END FORALL
```

The `FORALL` version is more readable.

Assignment in a `FORALL` is like an array assignment: as if all the
expressions were evaluated in any order, held in temporary storage, then
all the assignments performed in any order. The first statement must
fully complete before the second can begin.

A `FORALL` may be nested, and may include a `WHERE`. Procedures
referenced within a `FORALL` must be pure.

### Array elements

For a simple case, given

```f90
REAL, DIMENSION(100, 100) :: a
```

we can reference a single element as, for instance, `a(1, 1)`. For a
derived-data type like

```f90
TYPE fun_del
   REAL                  u
   REAL, DIMENSION(3) :: du
END TYPE fun_del
```

we can declare an array of that type:

```f90
TYPE(fun_del), DIMENSION(10, 20) :: tar
```

and a reference like

```f90
tar(n, 2)
```

is an element (a scalar!) of type fun_del, but

```f90
tar(n, 2)%du
```

is an array of type real, and

```f90
tar(n, 2)%du(2)
```

is an element of it. The basic rule to remember is that an array element
always has a subscript or subscripts qualifying at least the last name.

### Array subobjects (sections)

The general form of subscript for an array section is

`      [`*`lower`*`] : [`*`upper`*`] [:`*`stride`*`]`

(where \[ \] indicates an optional item) as in

```f90
REAL a(10, 10)
a(i, 1:n)                ! part of one row
a(1:m, j)                ! part of one column
a(i, : )                 ! whole row
a(i, 1:n:3)              ! every third element of row
a(i, 10:1:-1)            ! row in reverse order
a( (/ 1, 7, 3, 2 /), 1)  ! vector subscript
a(1, 2:11:2)             ! 11 is legal as not referenced
a(:, 1:7)                ! rank two section
```

Note that a vector subscript with duplicate values cannot appear on the
left-hand side of an assignment as it would be ambiguous. Thus,

```f90
b( (/ 1, 7, 3, 7 /) ) = (/ 1, 2, 3, 4 /)
```

is illegal. Also, a section with a vector subscript must not be supplied
as an actual argument to an `OUT` or `INOUT` dummy argument. Arrays of
arrays are not allowed:

```f90
tar%du             ! illegal
```

We note that a given value in an array can be referenced both as an
element and as a section:

```f90
a(1, 1)            !  scalar (rank zero)
a(1:1, 1)          !  array section (rank one)
```

depending on the circumstances or requirements. By qualifying objects of
derived type, we obtain elements or sections depending on the rule
stated earlier:

```f90
tar%u              !  array section (structure component)
tar(1, 1)%u        !  component of an array element
```

### Arrays intrinsic functions

***Vector and matrix multiply***

|               |                                  |
|---------------|----------------------------------|
| `DOT_PRODUCT` | Dot product of 2 rank-one arrays |
| `MATMUL`      | Matrix multiplication            |

***Array reduction***

|           |                                                             |
|-----------|-------------------------------------------------------------|
| `ALL`     | True if all values are true                                 |
| `ANY`     | True if any value is true. Example: `IF (ANY( a > b)) THEN` |
| `COUNT`   | Number of true elements in array                            |
| `MAXVAL`  | Maximum value in an array                                   |
| `MINVAL`  | Minimum value in an array                                   |
| `PRODUCT` | Product of array elements                                   |
| `SUM`     | Sum of array elements                                       |

***Array inquiry***

|             |                                      |
|-------------|--------------------------------------|
| `ALLOCATED` | Array allocation status              |
| `LBOUND`    | Lower dimension bounds of an array   |
| `SHAPE`     | Shape of an array (or scalar)        |
| `SIZE`      | Total number of elements in an array |
| `UBOUND`    | Upper dimension bounds of an array   |

***Array construction***

|          |                                                      |
|----------|------------------------------------------------------|
| `MERGE`  | Merge under mask                                     |
| `PACK`   | Pack an array into an array of rank one under a mask |
| `SPREAD` | Replicate array by adding a dimension                |
| `UNPACK` | Unpack an array of rank one into an array under mask |

***Array reshape***

|           |                  |
|-----------|------------------|
| `RESHAPE` | Reshape an array |

***Array manipulation***

|             |                                   |
|-------------|-----------------------------------|
| `CSHIFT`    | Circular shift                    |
| `EOSHIFT`   | End-off shift                     |
| `TRANSPOSE` | Transpose of an array of rank two |

***Array location***

|          |                                             |
|----------|---------------------------------------------|
| `MAXLOC` | Location of first maximum value in an array |
| `MINLOC` | Location of first minimum value in an array |


## Pointers

### Basics

Pointers are variables with the `POINTER` attribute; they are not a
distinct data type (and so no 'pointer arithmetic' is possible).

```f90
REAL, POINTER :: var
```

They are conceptually a descriptor listing the attributes of the objects
(targets) that the pointer may point to, and the address, if any, of a
target. They have no associated storage until it is allocated or
otherwise associated (by pointer assignment, see
<a href="#Pointers_in_expressions_and_assignments" class="wikilink"
title="below">below</a>):

```f90
ALLOCATE (var)
```

and they are dereferenced automatically, so no special symbol required.
In

```f90
var = var + 2.3
```

the value of the target of var is used and modified. Pointers cannot be
transferred via I/O. The statement

```f90
WRITE *, var
```

writes the value of the target of var and not the pointer descriptor
itself.

A pointer can point to another pointer, and hence to its target, or to a
static object that has the `TARGET` attribute:

```f90
REAL, POINTER :: object
REAL, TARGET  :: target_obj
var => object                  ! pointer assignment
var => target_obj
```

but they are strongly typed:

```f90
INTEGER, POINTER :: int_var
var => int_var                 ! illegal - types must match
```

and, similarly, for arrays the ranks as well as the type must agree.

A pointer can be a component of a derived type:

```f90
TYPE entry                       ! type for sparse matrix
   REAL :: value
   INTEGER :: index
   TYPE(entry), POINTER :: next  ! note recursion
END TYPE entry
```

and we can define the beginning of a linked chain of such entries:

```f90
TYPE(entry), POINTER :: chain
```

After suitable allocations and definitions, the first two entries could
be addressed as

```f90
chain%value           chain%next%value
chain%index           chain%next%index
chain%next            chain%next%next
```

but we would normally define additional pointers to point at, for
instance, the first and current entries in the list.

### Association

A pointer's association status is one of Some care has to be taken not
to leave a pointer 'dangling' by use of `DEALLOCATE` on its target
without nullifying any other pointer referring to it.

The intrinsic function `ASSOCIATED` can test the association status of a
defined pointer:

```f90
IF (ASSOCIATED(ptr)) THEN
```

or between a defined pointer and a defined target (which may, itself, be
a pointer):

```f90
IF (ASSOCIATED(ptr, target)) THEN
```

An alternative way to initialize a pointer, also in a specification
statement, is to use the `NULL` function:

```f90
REAL, POINTER, DIMENSION(:) :: vector => NULL() ! compile time
vector => NULL()                                ! run time
```

### Pointers in expressions and assignments

For intrinsic types we can 'sweep' pointers over different sets of
target data using the same code without any data movement. Given the
matrix manipulation *y = B C z*, we can write the following code
(although, in this case, the same result could be achieved more simply
by other means):

```f90
REAL, TARGET  :: b(10,10), c(10,10), r(10), s(10), z(10)
REAL, POINTER :: a(:,:), x(:), y(:)
INTEGER mult
:
DO mult = 1, 2
   IF (mult == 1) THEN
      y => r              ! no data movement
      a => c
      x => z
   ELSE
      y => s              ! no data movement
      a => b
      x => r
   END IF
   y = MATMUL(a, x)       ! common calculation
END DO
```

For objects of derived type we have to distinguish between pointer and
normal assignment. In

```f90
TYPE(entry), POINTER :: first, current
:
first => current
```

the assignment causes first to point at current, whereas

```f90
first =  current
```

causes current to overwrite first and is equivalent to

```f90
first%value = current%value
first%index = current%index
first%next => current%next
```

### Pointer arguments

If an actual argument is a pointer then, if the dummy argument is also a
pointer,

-   it must have same rank,
-   it receives its association status from the actual argument,
-   it returns its final association status to the actual argument
    (note: the target may be undefined!),
-   it may not have the `INTENT` attribute (it would be ambiguous),
-   it requires an interface block.

If the dummy argument is not a pointer, it becomes associated with the
target of the actual argument:

```f90
   REAL, POINTER :: a (:,:)
      :
   ALLOCATE (a(80, 80))
      :
   CALL sub(a)
      :
SUBROUTINE sub(c)
   REAL c(:, :)
```

### Pointer functions

Function results may also have the `POINTER` attribute; this is useful
if the result size depends on calculations performed in the function, as
in

```f90
USE data_handler
REAL x(100)
REAL, POINTER :: y(:)
:
y => compact(x)
```

where the module data_handler contains

```f90
FUNCTION compact(x)
   REAL, POINTER :: compact(:)
   REAL x(:)
   ! A procedure to remove duplicates from the array x
   INTEGER n
   :              ! Find the number of distinct values, n
   ALLOCATE(compact(n))
   :              ! Copy the distinct values into compact
END FUNCTION compact
```

The result can be used in an expression (but must be associated with a
defined target).

### Arrays of pointers

These do not exist as such: given

```f90
TYPE(entry) :: rows(n)
```

then

```f90
rows%next              ! illegal
```

would be such an object, but with an irregular storage pattern. For this
reason they are not allowed. However, we can achieve the same effect by
defining a derived data type with a pointer as its sole component:

```f90
TYPE row
   REAL, POINTER :: r(:)
END TYPE
```

and then defining arrays of this data type

```f90
TYPE(row) :: s(n), t(n)
```

where the storage for the rows can be allocated by, for instance,

```f90
DO i = 1, n
   ALLOCATE (t(i)%r(1:i)) ! Allocate row i of length i
END DO
```

The array assignment

```f90
s = t
```

is then equivalent to the pointer assignments

```f90
s(i)%r => t(i)%r
```

for all components.

### Pointers as dynamic aliases

Given an array

```f90
REAL, TARGET :: table(100,100)
```

that is frequently referenced with the fixed subscripts

```f90
table(m:n, p:q)
```

these references may be replaced by

```f90
REAL, DIMENSION(:, :), POINTER :: window
   :
window => table(m:n, p:q)
```

The subscripts of window are

```f90
1:n-m+1, 1:q-p+1
```

. Similarly, for

```f90
tar%u
```

(as defined in
<a href="#Array_elements" class="wikilink" title="already">already</a>),
we can use, say,

```f90
taru => tar%u
```

to point at all the u components of tar, and subscript it as

```f90
taru(1, 2)
```

The subscripts are as those of tar itself. (This replaces yet more of
`EQUIVALENCE`.)

In the pointer association

```f90
pointer => array_expression
```

the lower bounds for `pointer` are determined as if `lbound` was applied
to `array_expression`. Thus, when a pointer is assigned to a whole array
variable, it inherits the lower bounds of the variable, otherwise, the
lower bounds default to 1.

<a href="Fortran_2003" class="wikilink" title="Fortran 2003">Fortran
2003</a> allows specifying arbitrary lower bounds on pointer
association, like

```f90
window(r:,s:) => table(m:n,p:q)
```

so that the bounds of `window` become `r:r+n-m,s:s+q-p`.
<a href="Fortran_95" class="wikilink" title="Fortran 95">Fortran 95</a>
does not have this feature; however, it can be simulated using the
following trick (based on the pointer association rules for assumed
shape array dummy arguments):

```f90
FUNCTION remap_bounds2(lb1,lb2,array) RESULT(ptr)
   INTEGER, INTENT(IN)                            :: lb1,lb2
   REAL, DIMENSION(lb1:,lb2:), INTENT(IN), TARGET :: array
   REAL, DIMENSION(:,:), POINTER                  :: ptr
   ptr => array
END FUNCTION
  :
window => remap_bounds2(r,s,table(m:n,p:q))
```

The source code of an extended example of the use of pointers to support
a data structure is in
[pointer.f90](ftp://ftp.numerical.rl.ac.uk/pub/MRandC/pointer.f90).


## Intrinsic procedures

Most of the intrinsic functions have already been mentioned. Here, we
deal only with their general classification and with those that have so
far been omitted. All intrinsic procedures can be used with keyword
arguments:

```f90
CALL DATE_AND_TIME (TIME=t)
```

and many have optional arguments.

The intrinsic procedures are grouped into four categories:

1.  elemental - work on scalars or arrays, e.g. `ABS(a)`;
2.  inquiry - independent of value of argument (which may be undefined),
    e.g. `PRECISION(a)`;
3.  transformational - array argument with array result of different
    shape, e.g. `RESHAPE(a, b)`;
4.  subroutines, e.g. `SYSTEM_CLOCK`.

The procedures not already introduced are

Bit inquiry

|            |                             |
|------------|-----------------------------|
| `BIT_SIZE` | Number of bits in the model |

Bit manipulation

|          |                    |
|----------|--------------------|
| `BTEST`  | Bit testing        |
| `IAND`   | Logical AND        |
| `IBCLR`  | Clear bit          |
| `IBITS`  | Bit extraction     |
| `IBSET`  | Set bit            |
| `IEOR`   | Exclusive OR       |
| `IOR`    | Inclusive OR       |
| `ISHFT`  | Logical shift      |
| `ISHFTC` | Circular shift     |
| `NOT`    | Logical complement |

Transfer function, as in

```f90
INTEGER :: i = TRANSFER('abcd', 0)
```

(replaces part of EQUIVALENCE)

Subroutines

|                 |                                   |
|-----------------|-----------------------------------|
| `DATE_AND_TIME` | Obtain date and/or time           |
| `MVBITS`        | Copies bits                       |
| `RANDOM_NUMBER` | Returns pseudorandom numbers      |
| `RANDOM_SEED`   | Access to seed                    |
| `SYSTEM_CLOCK`  | Access to system clock            |
| `CPU_TIME`      | Returns processor time in seconds |


## Data transfer

### Formatted input/output

These examples illustrate various forms of I/O lists with some simple
formats (see
<a href="#Edit_descriptors" class="wikilink" title="below">below</a>):

```f90
INTEGER             :: i
REAL, DIMENSION(10) :: a
CHARACTER(len=20)   :: word
PRINT "(i10)",     i
PRINT "(10f10.3)", a
PRINT "(3f10.3)",  a(1),a(2),a(3)
PRINT "(a10)",     word(5:14)
PRINT "(3f10.3)",  a(1)*a(2)+i, SQRT(a(3:4))
```

Variables, but not expressions, are equally valid in input statements
using the `READ` statement:

```f90
READ "(i10)", i
```

If an array appears as an item, it is treated as if the elements were
specified in array element order.

Any pointers in an I/O list must be associated with a target, and
transfer takes place between the file and the targets.

An item of derived type is treated as if the components were specified
in the same order as in the type declaration, so

```f90
read "(8f10.5)", p, t  ! types point and triangle
```

has the same effect as the statement

```f90
READ "(8f10.5)", p%x, p%y, t%a%x, t%a%y, t%b%x, &
                           t%b%y, t%c%x, t%c%y
```

An object in an I/O list is not permitted to be of a derived type that
has a pointer component at any level of component selection.

Note that a zero-sized array may occur as an item in an I/O list. Such
an item corresponds to no actual data transfer.

The format specification may also be given in the form of a character
expression:

```f90
CHARACTER(len=*), parameter :: form = "(f10.3)"
:
PRINT form, q
```

or as an asterisk this is a type of I/O known as *list-directed* I/O
(see
<a href="#List-directed_I/O" class="wikilink" title="below">below</a>),
in which the format is defined by the computer system:

```f90
PRINT *, "Square-root of q = ", SQRT(q)
```

Input/output operations are used to transfer data between the storage of
an executing program and an external medium, specified by a *unit
number*. However, two I/O statements, `PRINT` and a variant of `READ`,
do not reference any unit number: this is referred to as terminal I/O.
Otherwise the form is:

```f90
READ (UNIT=4,     FMT="(f10.3)") q
READ (UNIT=nunit, FMT="(f10.3)") q
READ (UNIT=4*i+j, FMT="(f10.3)") a
```

where `UNIT=` is optional. The value may be any nonnegative integer
allowed by the system for this purpose (but 0, 5 and 6 often denote the
error, keyboard and terminal, respectively).

An asterisk is a variantagain from the keyboard:

```f90
READ (UNIT=*, FMT="(f10.3)") q
```

A read with a unit specifier allows
<a href="exception_handling" class="wikilink"
title="exception handling">exception handling</a>:

```f90
READ (UNIT=NUNIT, FMT="(3f10.3)", IOSTAT=ios) a,b,c
IF (ios == 0) THEN
!     Successful read - continue execution.
   :
ELSE
!     Error condition - take appropriate action.
   CALL error (ios)
END IF
```

There a second type of formatted output statement, the `WRITE`
statement:

```f90
WRITE (UNIT=nout, FMT="(10f10.3)", IOSTAT=ios) a
```

### Internal files

These allow format conversion between various representations to be
carried out by the program in a storage area defined within the program
itself.

```f90
INTEGER, DIMENSION(30)         :: ival
INTEGER                        :: key
CHARACTER(LEN=30)              :: buffer
CHARACTER(LEN=6), DIMENSION(3), PARAMETER :: form = (/ "(30i1)", "(15i2)","(10i3)" /)
READ (UNIT=*, FMT="(a30,i1)")      buffer, key
READ (UNIT=buffer, FMT=form(key)) ival(1:30/key)
```

If an internal file is a scalar, it has a single record whose length is
that of the scalar.

If it is an array, its elements, in array element order, are treated as
successive records of the file and each has length that of an array
element.

An example using a `WRITE` statement is

```f90
INTEGER           :: day
REAL              :: cash
CHARACTER(LEN=50) :: line
:
!   write into line
WRITE (UNIT=line, FMT="(a, i2, a, f8.2, a)") "Takings for day ", day, " are ", cash, " dollars"
```

that might write

     Takings for day  3 are  4329.15 dollars

### List-directed I/O

An example of a read without a specified format for input is

```f90
INTEGER               :: i
REAL                  :: a
COMPLEX, DIMENSION(2) :: field
LOGICAL               :: flag
CHARACTER(LEN=12)     :: title
CHARACTER(LEN=4)      :: word
:
READ *, i, a, field, flag, title, word
```

If this reads the input record

```f90
10 6.4 (1.0,0.0) (2.0,0.0) t test/
```

(in which blanks are used as separators), then `i`, `a`, `field`,
`flag`, and `title` will acquire the values 10, 6.4, (1.0,0.0) and
(2.0,0.0), `.true.` and `test` respectively, while `word` remains
unchanged.

Quotation marks or apostrophes are required as delimiters for a string
that contains a blank.

### Non-advancing I/O

This is a form of reading and writing without always advancing the file
position to ahead of the next record. Whereas an advancing I/O statement
always repositions the file after the last record accessed, a
non-advancing I/O statement performs no such repositioning and may
therefore leave the file positioned within a record.

```f90
CHARACTER(LEN=3)  :: key
INTEGER           :: u, s, ios
:
READ(UNIT=u, FMT="(a3)", ADVANCE="no", SIZE=s, IOSTAT=ios) key
IF (ios == 0) THEN
   :
ELSE
!    key is not in one record
   key(s+1:) = ""
   :
END IF
```

A non-advancing read might read the first few characters of a record and
a normal read the remainder.

In order to write a prompt to a terminal screen and to read from the
next character position on the screen without an intervening line-feed,
we can write

```f90
WRITE (UNIT=*, FMT="(a)", ADVANCE="no") "enter next prime number:"
READ  (UNIT=*, FMT="(i10)") prime_number
```

Non-advancing I/O is for external files, and is not available for
list-directed I/O.

### Edit descriptors

It is possible to specify that an edit descriptor be repeated a
specified number of times, using a *repeat count*: `10f12.3`

The slash edit descriptor (see
<a href="#Control_edit_descriptors" class="wikilink"
title="below">below</a>) may have a repeat count, and a repeat count can
also apply to a group of edit descriptors, enclosed in parentheses, with
nesting:

```f90
PRINT "(2(2i5,2f8.2))", i(1),i(2),a(1),a(2), i(3),i(4),a(3),a(4)
```

Entire format specifications can be repeated:

```f90
PRINT "(10i8)", (/ (i(j), j=1,200) /)
```

writes 10 integers, each occupying 8 character positions, on each of 20
lines (repeating the format specification advances to the next line).

#### Data edit descriptors

#### Control edit descriptors

*Control edit descriptors setting conditions*: *Control edit descriptors
for immediate processing*:

### Unformatted I/O

This type of I/O should be used only in cases where the records are
generated by a program on one computer, to be read back on the same
computer or another computer using the same internal number
representations:

```f90
OPEN(UNIT=4, FILE='test', FORM='unformatted')
READ(UNIT=4) q
WRITE(UNIT=nout, IOSTAT=ios) a  ! no fmt=
```

### Direct-access files

This form of I/O is also known as random access or indexed I/O. Here,
all the records have the same length, and each record is identified by
an index number. It is possible to write, read, or re-write any
specified record without regard to position.

```f90
INTEGER, PARAMETER :: nunit=2, length=100
REAL, DIMENSION(length)            :: a
REAL, DIMENSION(length+1:2*length) :: b
INTEGER                            :: i, rec_length
:
INQUIRE (IOLENGTH=rec_length) a
OPEN (UNIT=nunit, ACCESS="direct", RECL=rec_length, STATUS="scratch", ACTION="readwrite")
:
!   Write array b to direct-access file in record 14
WRITE (UNIT=nunit, REC=14) b
:
!
!   Read the array back into array a
READ (UNIT=nunit, REC=14) a
:
DO i = 1, length/2
   a(i) = i
END DO
!
!   Replace modified record
WRITE (UNIT=nunit, REC=14) a
```

The file must be an external file and list-directed formatting and
non-advancing I/O are unavailable.


## Operations on external files

Once again, this is an overview only.

### File positioning statements

### The `OPEN` statement

The statement is used to connect an external file to a unit, create a
file that is preconnected, or create a file and connect it to a unit.
The syntax is

```f90
OPEN (UNIT=u, STATUS=st, ACTION=act [,olist])
```

where `olist` is a list of optional specifiers. The specifiers may
appear in any order.

```f90
OPEN (UNIT=2, IOSTAT=ios, FILE="cities", STATUS="new", ACCESS="direct",  &
      ACTION="readwrite", RECL=100)
```

Other specifiers are `FORM` and `POSITION`.

### The `CLOSE` statement

This is used to disconnect a file from a unit.

```f90
CLOSE (UNIT=u [, IOSTAT=ios] [, STATUS=st])
```

as in

```f90
CLOSE (UNIT=2, IOSTAT=ios, STATUS="delete")
```

### The `inquire` statement

At any time during the execution of a program it is possible to inquire
about the status and attributes of a file using this statement.

Using a variant of this statement, it is similarly possible to determine
the status of a unit, for instance whether the unit number exists for
that system.

Another variant permits an inquiry about the length of an output list
when used to write an unformatted record.

For inquire by unit

```f90
INQUIRE (UNIT=u, ilist)
```

or for inquire by file

```f90
INQUIRE (FILE=fln, ilist)
```

or for inquire by I/O list

```f90
INQUIRE (IOLENGTH=length) olist
```

As an example

```f90
LOGICAL            :: ex, op
CHARACTER (LEN=11) :: nam, acc, seq, frm
INTEGER            :: irec, nr
INQUIRE (UNIT=2, EXIST=ex, OPENED=op, NAME=nam, ACCESS=acc, SEQUENTIAL=seq, &
         FORM=frm, RECL=irec, NEXTREC=nr)
```

yields

```f90
ex      .true.
op      .true.
nam      cities
acc      DIRECT
seq      NO
frm      UNFORMATTED
irec     100
nr       1
```

(assuming no intervening read or write operations).

Other specifiers are
`IOSTAT, OPENED, NUMBER, NAMED, FORMATTED, POSITION, ACTION, READ, WRITE, READWRITE`.

```mediawiki
==References==
{{Reflist}}
=== Bibliography ===
{{refbegin}}
* {{Citation |last=Metcalf |first=Michael |title=Whence Fortran? |date=2004-06-17 |work=Fortran 95/2003 Explained |pages=1–8 |url=https://doi.org/10.1093/oso/9780198526926.003.0001 |access-date=2025-02-25 |publisher=Oxford University PressOxford |isbn=978-0-19-852692-6 |last2=Reid |first2=John |last3=Cohen |first3=Malcolm}}
* {{Citation |title=Introduction to Modern Fortran |work=Statistics and Computing |pages=13–53 |url=https://doi.org/10.1007/0-387-28123-1_2 |access-date=2025-02-25 |place=New York |publisher=Springer-Verlag |isbn=0-387-23817-4}}
* {{Cite journal |last=Gehrke |first=Wilhelm |date=1996 |title=Fortran 95 Language Guide |url=https://doi.org/10.1007/978-1-4471-1025-5 |doi=10.1007/978-1-4471-1025-5}}
* {{Citation |last=Chivers |first=Ian |title=Fortran 2000 and Various Fortran Dialects |date=2000 |work=Introducing Fortran 95 |pages=377–388 |url=https://doi.org/10.1007/978-1-4471-0403-2_29 |access-date=2025-02-25 |place=London |publisher=Springer London |isbn=978-1-85233-276-1 |last2=Sleightholme |first2=Jane}}
* {{cite book|title=Fortran 95|author1-first=Martin|author1-last=Counihan|edition=2nd|publisher=CRC Press|year=2006|isbn=9780203978467}}
* {{cite book|title=Computer programming in FORTRAN 90 and 95|author1-first=V.|author1-last=Ramaraman|publisher=PHI Learning Pvt. Ltd.|year=1997|isbn=9788120311817}}
* {{cite book|title=Modern Fortran Explained: Incorporating Fortran 2023|author1-first=Michael|author1-last=Metcalf|author2-first=John|author2-last=Reid|author3-first=Malcolm|author3-last=Cohen|author4-first=Reinhold|author4-last=Bader|edition=6th|publisher=Oxford University Press|year=2024|isbn=9780198876595}}
* {{cite book|title=An Introduction to Fortran 90/95: Syntax and Programming|author1-first=Yogendra Prasad|author1-last=Joshi|publisher=Allied Publishers|isbn=9788177644746}}
{{refend}}
{{Authority control}}

{{DEFAULTSORT:Fortran Language Features}}
[[Category:Fortran|Features]]
```
