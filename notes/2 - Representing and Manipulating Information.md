# 2.1 Information Storage
## Byte
- Made up of 8 bits
- Dec range: 0~255

## Hexadecimal Notation
- `0x`

### Binary Conversion
|   Hex  | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | A    | B    | C    | D    | E    | F    |
| --- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|    Bin | 0000 | 0001 | 0010 | 0011 | 0100 | 0101 | 0110 | 0111 | 1000 | 1001 | 1010 | 1011 | 1100 | 1101 | 1110 | 1111 |

#### Exponents of two
$$2^n$$
$$n=i+4j$$

### Decimal Conversion
multiplication and division

## Words
- determines the size of virtual memory space:  $0\quad to\quad 2^w-1$
- compiling to 32 and 64-bit
- 64-bit arch supports 32-bit programs

### Word and Variable Size
![[/img/variable_size.png]]

## Addressing and Byte Ordering
- Big Endian: the front segments go to lower addresses
- Little Endian: the front segments go to higher addresses

## Strings
an array of chars followed by `0x0`

## Boolean Algebra

## Bit Operation
- Masking
- Bit Operations vs. Logical Operations
### Shift Operations
- Left Shift
- Right Shift: logical & arithmatic

## Operator Precedence in C
![[/img/截屏2022-09-14 下午8.18.36.png]]


# 2.2 Integer Representation
![[/img/截屏2022-09-10 下午1.42.44.png]]

## Unsigned Encoding
$B2U_w$ function maps bit sequence to non-negative integer

## Signed Encoding: Two's Complement
$B2T_w$ function maps bit sequence to integer

## Range
### Unsigned
- min: 0
- max: $2^w-1$

### Signed
- min: $-2^{w-1}$
- max: $2^{w-1}-1$

- representation of -1: all 1s

## Conversion
- Principle: bit sequence unchanged, only change the mapping function

![[/img/截屏2022-09-10 下午7.31.05.png]]

- Important when comparing or operating on different data types

## Expansion
- Unsigned: zero extension
- Signed: sign extension (similiar to arithmatic right shift)

## Integer Type Promotion in C
1. Any types shorter than int (and their unsigned counterparts), are implicitly expanded to int before any operations
2. When types of signed and unsigned variety and same length encounter, the signed variety is converted to unsigned.
3. Things are messy when we go above int...
![[/img/截屏2022-09-14 下午7.31.04.png]]
...which is why we get `(-2147483648 < 2147483647) == false` in 32-bit C90.
4. using `auto` to check

## Truncation
- Unsigned: mod operation
- Signed
![[/img/截屏2022-09-10 下午7.36.54.png]]
	1. convert binary sequence to unsigned
	2. truncate as unsigned number
	3. convert unsigned back to signed


# 2.3 Integer Arithmatic
## Unsigned Addition
- Add and discard overflow bit
![[/img/截屏2022-09-14 下午1.10.35.png]]

### Overflow Detection
`sum >= x`

## Signed Addition
![[/img/截屏2022-09-14 下午1.20.12.png]]

### Overflow Detection
![[/img/截屏2022-09-14 下午1.39.08.png]]

## Subtraction
### Additive Inverse

$$
-^u_wx = \begin{cases}
x &\text{x=0}\\
2^w-x &x \geq 0
\end{cases}
$$
$$
-^t_wx = \begin{cases}
-x &x > {TMin}_w\\
TMin_w &x = TMin_w
\end{cases}
$$

## Multiplication
### Unsigned
truncation

### Signed
$$U2T_w((x*y)mod2^w)$$

### Unsigned vs. Signed: Bit Sequence Equivalence

### Constant Multiplication
- Convert const to the sum of power of 2
- Left shift

## Division by Power of Two
- Right shift: arithmatic/ logical

### Round Toward Zero
- Unsigned: no extra steps needed
- Signed: if neg, add bias $2^k - 1$


# 2.4 Floating Point
## Fractional Binary Numbers
Limitations:
1. Only ${x}/{2^k}$
2. Limited dynamic range

## Floating Point Representation
![[/img/截屏2022-09-14 下午2.20.00.png]]

### Can (Almost) Use Unsigned Integer Comparison 
- Must first compare sign bits
- Must consider −0 = 0 
- NaNs problematic
	- Will be greater than any other values
	- What should comparison yield?
- Otherwise OK
	- Denorm vs. normalized
	- Normalized vs. infinity

## Types
![[/img/截屏2022-09-14 下午2.21.40.png]]

### 1. Normalized
![[/img/截屏2022-09-14 下午2.24.02.png]]
![[/img/截屏2022-09-14 下午2.24.27.png]]

### 2. Denormalized
For representing pos and neg 0 and very small numbers
![[/img/截屏2022-09-14 下午2.26.42.png]]

- positive and negative zero are equal, and any non-equal comparisons yield false
- $1/+0 = +\infty$ and $1/-0 = -\infty$

### 3. Special
![[/img/截屏2022-09-14 下午2.27.55.png]]

#### [How Infinity and nan works](https://www.gnu.org/software/libc/manual/html_node/Infinity-and-NaN.html#:~:text=In%20comparison%20operations%2C%20positive%20infinity,less%20than%20anything%2C%20including%20itself.)
The basic operations and math functions all accept infinity and NaN and produce sensible output. Infinities propagate through calculations as one would expect: for example, _2 + ∞ = ∞_, _4/∞ = 0_, _tan (∞) = π/2_. 

NaN, on the other hand, infects any calculation that involves it. Unless the calculation would produce the same result no matter what real value replaced NaN, the result is NaN.

In comparison operations, positive infinity is larger than all values except itself and NaN, and negative infinity is smaller than all values except itself and NaN. NaN is _unordered_: it is not equal to, greater than, or less than anything, _including itself_. `x == x` is false if the value of `x` is NaN. You can use this to test whether a value is NaN or not, but the recommended way to test for NaN is with the `isnan` function (see [Floating-Point Number Classification Functions](https://www.gnu.org/software/libc/manual/html_node/Floating-Point-Classes.html)). In addition, `<`, `>`, `<=`, and `>=` will raise an exception when applied to NaNs.

### Design Choices
1. The different definition of E in normalized and denormalized numbers mean that **normalized and denormalized numbers transition smoothly**
2. Non-negative floating point numbers can be sorted like unsigned ints

## Distribution
![[/img/截屏2022-09-14 下午3.36.02.png]]
![[/img/截屏2022-09-14 下午3.36.09.png]]

## Converting Int to Floating Point

## Rounding
1. **Round to Even**![[/img/截屏2022-09-14 下午2.35.36.png]]
2. Round Toward 0
3. Round Up
4. Round Down

- Round to Even has no statistic bias
- Fractions
- Binary: even means 0, halfway means 100...

## Operations
### Principle
$$x +_f y = Round(x+y)$$
$$x \times_f y = Round(x \times y)$$

Calculate exact result first, then round and overflow

### Multiplication
![[/img/截屏2022-09-14 下午3.56.05.png]]

### Addition
![[/img/截屏2022-09-14 下午3.58.17.png]]

### Properties
![[/img/截屏2022-09-14 下午3.59.40.png]]
![[/img/截屏2022-09-14 下午4.00.35.png]]

### Conversion
1. int -> float
	no overflow, but might round
2. int/ float -> double
	precise conversion
3. double -> float
	might overflow, might round
4. float/ double -> int
	- truncated
	- rounded toward zero
	- undefined behavior when out of range or NaN

## Building a Floating Point Number
1. Normalize to have a leading 1
2. Round
3. Postnormalize, in case leading 1 is lost