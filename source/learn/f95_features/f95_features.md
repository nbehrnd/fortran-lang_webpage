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
