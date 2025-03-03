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

