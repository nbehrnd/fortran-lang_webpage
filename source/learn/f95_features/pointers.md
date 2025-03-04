# Pointers

## Basics

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

## Association

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

## Pointers in expressions and assignments

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

## Pointer arguments

If an actual argument is a pointer then, if the dummy argument is also a
pointer,

- it must have same rank,
- it receives its association status from the actual argument,
- it returns its final association status to the actual argument
  (note: the target may be undefined!),
- it may not have the `INTENT` attribute (it would be ambiguous),
- it requires an interface block.

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

## Pointer functions

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

## Arrays of pointers

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

## Pointers as dynamic aliases

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
