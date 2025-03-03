# Expressions and assignments

## Scalar numeric

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
[rounding](https://en.wikipedia.org/wiki/Rounding)
of real numbers to integers:

- `NINT`: round to nearest integer, return integer result
- `ANINT`: round to nearest integer, return real result
- `INT`: truncate (round towards zero), return integer result
- `AINT`: truncate (round towards zero), return real result
- `CEILING`: smallest integral value not less than argument (round up)
  (Fortran-90)
- `FLOOR`: largest integral value not greater than argument (round
  down) (Fortran-90)

## Scalar relational operations

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

## Derived-data types

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

Notice the
["overloaded"](https://en.wikipedia.org/wiki/Operator_overloading)
use of the intrinsic symbol `//` and
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
[module](modules),
procedures defining the operator and assignment, and corresponding
operator-procedure association, as follows:

```f90
INTERFACE OPERATOR(//) !Overloads the // operator as invoking string_concat procedure
  MODULE PROCEDURE string_concat
END INTERFACE
```

The string concatenation function is a more elaborated version of that
shown already in
[Basics](Basics).
Note that
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
[Derived-data types](Derived-data_types):

```f90
str1 = string(2, char1//char2)  ! structure constructor
```

## Arrays

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

- `CEILING`
- `FLOOR`
- `MODULO`
  (also integer)
- `EXPONENT`
- `FRACTION`
- `NEAREST`
- `RRSPACING`
- `SPACING`
- `SCALE`
- `SET_EXPONENT`

These are array valued for array arguments (elemental), like all
[FORTRAN 77](https://en.wikipedia.org/wiki/FORTRAN_77)
functions (except LEN):

- `INT`
- `REAL`
- `CMPLX`
- `AINT`
- `ANINT`
- `NINT`
- `ABS`
- `MOD`
- `SIGN`
- `DIM`
- `MAX`
- `MIN`

Powers, logarithms, and trigonometric functions

- `SQRT`
- `EXP`
- `LOG`
- `LOG10`
- `SIN`
- `COS`
- `TAN`
- `ASIN`
- `ACOS`
- `ATAN`
- `ATAN2`
- `SINH`
- `COSH`
- `TANH`

Complex numbers:

- `AIMAG`
- `CONJG`

The following are for characters:

- `LGE`
- `LGT`
- `LLE`
- `LLT`
- `ICHAR`
- `CHAR`
- `INDEX`
