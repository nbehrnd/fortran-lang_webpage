# Intrinsic procedures

Most of the intrinsic functions have already been mentioned. Here, we
deal only with their general classification and with those that have so
far been omitted. All intrinsic procedures can be used with keyword
arguments:

```f90
CALL DATE_AND_TIME (TIME=t)
```

and many have optional arguments.

The intrinsic procedures are grouped into four categories:

1. elemental - work on scalars or arrays, e.g. `ABS(a)`;
1. inquiry - independent of value of argument (which may be undefined),
   e.g. `PRECISION(a)`;
1. transformational - array argument with array result of different
   shape, e.g. `RESHAPE(a, b)`;
1. subroutines, e.g. `SYSTEM_CLOCK`.

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
