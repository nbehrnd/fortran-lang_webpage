# Array handling

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

## Zero-sized arrays

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

## Assumed-shape arrays

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

## Automatic arrays

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

## ALLOCATABLE and ALLOCATE

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

## Elemental operations, assignments and procedures

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

## WHERE

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

## The FORALL statement and construct

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

## Array elements

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

## Array subobjects (sections)

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

## Arrays intrinsic functions

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

