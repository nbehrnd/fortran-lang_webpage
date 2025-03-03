# Program units and procedures

## Definitions

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

## Internal procedures

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

## Modules

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

## Controlling accessibility

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

## Arguments

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

## Interface blocks

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

## Overloading and generic interfaces

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

## Recursion

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

## Pure procedures

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
