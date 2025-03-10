# Data transfer

## Formatted input/output

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

## Internal files

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

```shell
Takings for day  3 are  4329.15 dollars
```

## List-directed I/O

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

## Non-advancing I/O

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

## Edit descriptors

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

### Data edit descriptors

### Control edit descriptors

*Control edit descriptors setting conditions*: *Control edit descriptors
for immediate processing*:

## Unformatted I/O

This type of I/O should be used only in cases where the records are
generated by a program on one computer, to be read back on the same
computer or another computer using the same internal number
representations:

```f90
OPEN(UNIT=4, FILE='test', FORM='unformatted')
READ(UNIT=4) q
WRITE(UNIT=nout, IOSTAT=ios) a  ! no fmt=
```

## Direct-access files

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
