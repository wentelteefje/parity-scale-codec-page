---
title: "Encode"
weight: 2
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
math: true
---

<!-- # 0. How to use this page

This pages gives a list of common examples in the usage of the SCALE codec. The examples are ordered roughly by increasing complexity. It is advised to read the whole page in order, but more advanced users can also skip the more basic topics. For a quick overview see the [specification](/docs/specification). -->

# 1 Little-endian Encoding

SCALE encoded types are stored in little-endian (LE) mode. Little-endian systems store the least significant byte at the smallest memory address. In contrast, big-endian (BE) systems store the most significant byte at the smallest memory address. For example, consider the `u32` integer given by the hexadecimal representation $\text{0x0a0b0c0d}$. In LE systems the least significant byte, that is $\text{0x0d}$, would be stored at the smallest memory address. The next diagram illustrates how the integer would be stored in memory for the two different modes.

| Memory Address | $a$ | $a+1$ | $a+2$ | $a+3$ | ... |
|----------------|---|-----|-----|-----|-----|
|LE mode | $\text{0x0d}$ | $\text{0x0c}$ | $\text{0x0b}$ | $\text{0x0a}$ |
|BE mode | $\text{0x0a}$ | $\text{0x0b}$ | $\text{0x0c}$ | $\text{0x0d}$ |

Note, that endianness is not a property of numbers themselves and therefore not reflected in their binary or hexadecimal representations. However, in Rust you can use the `to_le_bytes()`-method to obtain a little-endian vector representation of an integer. For example:

```rust
fn main() {
    println!("{:b}", 42u16);
    println!("{:02x?}", 42u16.to_le_bytes());
    println!("{:b}", 16777215u32);
    println!("{:02x?}", 16777215u32.to_le_bytes());
}

101010
[2a, 00]
111111111111111111111111
[ff, ff, ff, 00]
```
In analogy to how the data would be stored in memory, the least significant byte is stored at the smallest vector index. Of course, this is only useful once the type is bigger than one byte.

# 2 SCALE Encoding
This section provides code snippets to get you started with encoding your types with SCALE. 

## 2.1 Integers

Out of the box integers in SCALE follow a fixed-width Little-endian encoding. Just call the `encode()`-method on your integers to encode them using SCALE.

```rust
use parity_scale_codec::Encode;

fn main() {
    println!("{:02x?}", 69i8.encode());
    println!("{:02x?}", 42u16.encode());
    println!("{:02x?}", 16777215u32.encode());
}

[45]
[2a, 00]
[ff, ff, ff, 00]
```

## 2.2 Unit and Booleans

The SCALE encoding for the unit type `()` is equal to a byte array of length zero. As for booleans, `true` is encoded as `0x01` and `false` as `0x00`.

```rust
use parity_scale_codec::Encode;

fn main() {
println!("{:02x?}", ().encode());
    println!("{:02x?}", true.encode());
    println!("{:02x?}", false.encode());
}
[]
[0x01]
[0x00]
```

## 2.3 Options and Results

If the option is `Some` and contains a value it is encoded by `0x01` followed by the encoded value. `None` is encoded by `0x00`. An `Ok(T)` result is encoded by `0x00` followed by the encoded value of `T`, whereas an `Err(E)` result is encoded by `0x01` followed by the encoded error `E`.

```rust
use parity_scale_codec::Encode;

fn main() {
    println!("{:02x?}", Ok::<u32, ()>(42u32).encode());
    println!("{:02x?}", Err::<u32, ()>(()).encode());
    println!("{:02x?}", Some(42u32).encode());
    println!("{:02x?}", None::<u32>.encode());
}
[00, 2a, 00, 00, 00]
[01]
[01, 2a, 00, 00, 00]
[00]
```

## 2.2 Tuples, Arrays and Structs

Tuples, arrays and structs are all fixed-length data types in Rust and hence encoded by simply concatenating their respective items. For structs you need to derive the `Encode` trait before being able to encode them.

```rust
use parity_scale_codec::Encode;
use parity_scale_codec_derive::Encode;

#[derive(Encode)]
struct Example {
    number: u8,
    is_cool: bool,
    optional: Option<u32>,
}

fn main() {
    let my_struct = Example {
        number: 0,
        is_cool: true,
        optional: Some(69),
    };
    println!("{:02x?}", [0u8, 1u8, 2u8, 3u8, 4u8].encode());
    println!("{:02x?}", (0u8, true, Some(69u32)).encode());
    println!("{:02x?}", my_struct.encode());
}
[00, 01, 02, 03, 04]
[00, 01, 01, 45, 00, 00, 00]
[00, 01, 01, 45, 00, 00, 00]
```
Note, that since SCALE is a non-descriptive encoding there's no way to distinguish between the encoded values. This will become important when decoding values.

## 2.4 Compact Integer encoding

Non-negative integers between $0$ and $2^{536} - 1$ can be more efficiently encoded using SCALE's compact encoding. While the ordinary fixed-width integer encoding depends on the size of the given integer's type (e.g. `u8`, `u16`, `u32`, ...), the compact encoding only looks at the number itself and disregards the type information. For example, the compact encodings of the integers `60u8`, `60u16` and `60u32` are all the same: $\text{Enc}\_{\text{SC}}^{\text{Comp}}(60) = \text{[0xf0]}$.

 It can be used by defining a wrapper struct and deriving the `Encode` trait for it. Here, the `codec(compact)` attribute of the derive macro makes the field following it use compact encoding.

```rust
use parity_scale_codec::{Encode, HasCompact};
use parity_scale_codec_derive::Encode;

#[derive(Encode)]
struct AsCompact<T: HasCompact>(#[codec(compact)] T);

fn main() {
    println!("{:02x?}", 0u8.encode());
    println!("{:02x?}", 0u16.encode());
    println!("{:02x?}", 0u32.encode());
    println!("{:02x?}", AsCompact(60u8).encode());
    println!("{:02x?}", AsCompact(60u16).encode());
    println!("{:02x?}", AsCompact(60u32).encode());
}
[00]
[00, 00]
[00, 00, 00, 00]
[f0]
[f0]
[f0]
```

There are four different modes in compact encoding. Which mode is used is automatically determined dependent on the size of the given integer. A quick overview of the cases is covered in the [specification](/docs/specification). Each mode is specified using a bit flag appended as the least significant two bits of the compact encoding.

| Mode | Single-byte | Two-byte | Four-byte | Big-integer |
| -- | -- | -- | -- | -- | 
| Bit flag | 00 | 01 | 10 | 11 |

The following section provides more in-depth examples of how the different modes operate. It's not strictly necessary to understand how the values are encoded to use compact encoding. Since the appropriate mode is deduced automatically during encoding, there's no difference in usage from the perspective of a Rust user.

In what follows $\text{0x}$ indicates the hexadecimal representation of a number and $\text{0b}$ indicates its binary representation.

### 2.4.1 Single-byte mode
In this mode a given non-negative integer $n$ is encoded as a single byte. This is possible for $0 \leq n \leq 63$. Here the six most significant bits are the encoding of the value. As an example, consider the case $n = 42 = 2^5 + 2^3 + 2^1$. The compact encoding of $n$ is obtained by appending ${\color{red}00}$ as its least significant bits:

$$ 42 = 0b101010 \Longrightarrow 0b101010{\color{red}00} = 168$$

Therefore, the compact encoding of $n$ is given by the byte array $\text{Enc}\_{\text{SC}}^{\text{Comp}}(n) = \text{[0xa8]}$. Since there's only one byte in this mode the LE aspect cannot be seen.

```rust
use parity_scale_codec::{Encode, HasCompact};
use parity_scale_codec_derive::Encode;

#[derive(Encode)]
struct AsCompact<T: HasCompact>(#[codec(compact)] T);

fn main() {
    println!("{:02x?}", 42u8.encode());
    println!("{:02x?}", 42u32.encode());
    println!("{:02x?}", AsCompact(42u8).encode());
    println!("{:02x?}", AsCompact(42u32).encode());
}

[2a]
[2a, 00, 00, 00]
[a8]
[a8]
```

### 2.4.2 Two-byte mode
In this mode two bytes are used for the encoding. The six most significant bits of the first byte and the following byte are the LE encoding of the value. It applies for $2^6 \leq n \leq 2^{14} -1$. Consider the case $n = 69 = 2^6 + 2^2 + 2^0$. The compact encoding of $n$ is obtained by appending ${\color{red}01}$ as its least significant bits:
$$ 69 = 0b1000101 \Longrightarrow 0b1000101{\color{red}01} = 277.$$
Since the resulting integer exceeds one byte, the number is split up starting with the least-significant byte. The compact encoding $\text{Enc}\_{\text{SC}}^{\text{Comp}}(n)$ is given by the byte array:
$$ 0b00000001\\;00010101 = \text{[0x15, 0x01]}.$$

```rust
use parity_scale_codec::{Encode, HasCompact};
use parity_scale_codec_derive::Encode;

#[derive(Encode)]
struct AsCompact<T: HasCompact>(#[codec(compact)] T);

fn main() {
    println!("{:02x?}", 69u8.encode());
    println!("{:02x?}", 69u32.encode());
    println!("{:02x?}", AsCompact(69u8).encode());
    println!("{:02x?}", AsCompact(69u32).encode());
}
[45]
[45, 00, 00, 00]
[15, 01]
[15, 01]
```

### 2.4.3 Four-byte mode
This mode uses four bytes to encode the value, which happens when $2^{14} \leq n \leq 2^{30} - 1$. Consider the case $n = 2^{16} - 1 = 65535$. This is the maximum value for the type `u16`. Its compact encoding is obtained by appending ${\color{red}10}$ as its least significant bits:
$$ 65535 = 0b11111111\\;11111111 \Longrightarrow 0b11111111\\;11111111{\color{red}10} = 262142.$$
Analogously to the previous example, the resulting integer exceeds two bytes and needs to be split up using little-endian mode. Additionally, we pad with leading zeros. The compact encoding $\text{Enc}\_{\text{SC}}^{\text{Comp}}(n)$ is given by the byte array:
$$ 0b00000000\\;00000011\\;11111111\\;11111110 = \text{[0xfe, 0xff, 0x03, 0x00]}.$$
```rust
use parity_scale_codec::{Encode, HasCompact};
use parity_scale_codec_derive::Encode;

#[derive(Encode)]
struct AsCompact<T: HasCompact>(#[codec(compact)] T);

fn main() {
    println!("{:02x?}", 65535u16.encode());
    println!("{:02x?}", 65535u32.encode());
    println!("{:02x?}", AsCompact(65535u16).encode());
    println!("{:02x?}", AsCompact(65535u32).encode());
}
[ff, ff]
[ff, ff, 00, 00]
[fe, ff, 03, 00]
[fe, ff, 03, 00]
```
### 2.4.4 Big-integer mode

This mode is intended for non-negative integers between $2^{30}$ and $2^{536} - 1$. It differs from the other three modes in that it is a variable length encoding. Therefore, the first byte of a numbers compact encoding in big-integer mode is used to store the length $m$ of the actual encoding. As a first example, consider the case $n = 2^{30} = 1073741824$. This number's LE encoding is given by:
$$0b 01000000\\;00000000\\;00000000\\;00000000 = \text{[0x00, 0x00, 0x00, 0x40]}.$$

Now, in big-integer mode, the six most significant bits of the first byte are used to store the number of bytes used in the actual encoding of the number *minus four*. That is $m - 4$. Since the LE encoding of $n$ is exactly of length $4$, the upper six bits of the first byte must all be equal to zero. In accordance with the other cases, the mode is indicated using the two least significant bits of the first byte. For big-integer mode we append ${\color{red}11}$ to obtain as the first byte:
$$ 0 = m - 4 = 0b000000 \Longrightarrow 0b000000{\color{red}11} = 3.$$
In total, the compact encoding $\text{Enc}\_{\text{SC}}^{\text{Comp}}(n)$ is given by the byte array:
$$\text{[0x03, 0x00, 0x00, 0x00, 0x40]}.$$

Let's look at another example. The LE encoding of the number $n = 2^{32} = 4294967296$ is given by $\text{[0x00, 0x00, 0x00, 0x00, 0x01]}.$ This time we need five bytes to store it, i.e. $m = 5$. Again, we use six bits to encode this length *minus four* and after that append ${\color{red}11}$:
$$ 1 = m - 4 = 0b000001 \Longrightarrow 0b000001{\color{red}11} = 7. $$
Altogether, the compact encoding $\text{Enc}\_{\text{SC}}^{\text{Comp}}(n)$ is given by the byte array:
$$\text{[0x07, 0x00, 0x00, 0x00, 0x00, 0x01]}.$$
```rust
use parity_scale_codec::{Encode, HasCompact};
use parity_scale_codec_derive::Encode;

#[derive(Encode)]
struct AsCompact<T: HasCompact>(#[codec(compact)] T);

fn main() {
    println!("{:02x?}", 1073741824u32.encode());
    println!("{:02x?}", 4294967296u64.encode());
    println!("{:02x?}", AsCompact(1073741824u32).encode());
    println!("{:02x?}", AsCompact(4294967296u64).encode());
}
[00, 00, 00, 40]
[00, 00, 00, 00, 01, 00, 00, 00]
[03, 00, 00, 00, 40]
[07, 00, 00, 00, 00, 01]
```

## 2.5 Vectors and Strings
Vectors are encoded by concatenating their encoded items and prepending their compact encoded length. Strings are encoded as `u8`-vectors consisting of UTF-8 encoded bytes.

```rust
use parity_scale_codec::Encode;

fn main() {
    println!("{:02x?}", vec![0u8, 1u8, 2u8, 3u8, 4u8].encode());
    println!("{:02x?}", "hello".encode());
    println!("{:02x?}", vec![0u8; 1024].encode());
}
[14, 00, 01, 02, 03, 04]
[14, 68, 65, 6c, 6c, 6f]
[01, 10, 00, 00, ... , 00]
```


## 2.5 Using Compact Encoding in Structs

```rust
use parity_scale_codec::Encode;

```