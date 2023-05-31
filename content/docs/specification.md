---
title: "Specification"
weight: 4
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
math: true
---

# Specification

As its name suggests, all SCALE type encodings are arrived at by simply concatenating the encodings of its constituents, i.e. the basic types making up the respective type. This table provides a quick overview of the SCALE codec. For more hands-on explanations refer to the [examples section](/docs/examples).

As a visual aid the intermediary hexadecimal representation is provided as well as the final SCALE encoding.


| Data type | Encoding Description |  SCALE decoded value	| SCALE encoded value |
| --        | --          | -- | -- | 
| Unit | Dropped in Encoding. | `()` | `` | `` |
| Boolean    | Encoded using the least significant bit of a single byte. | `true` | `0x01` |
|           |                                                           | `false`| `0x00` |
| Integer | By default integers are encoded using a fixed-width little-endian format. | `69i8` | `0x45`|
|         |                                                                      | `69i16`| `0x4500`|
|         |                                                                      | `42u16`| `0x2a00`|
|         | Non-negative integers also have a compact encoding. Here, the two least significant bits denote the mode. | | |
|         | Single-byte mode: Upper six bits are the LE encoding of the value. Valid only for values of $0$ through $63$ |`0u8` | `0x0`|
|         | Two-byte mode: Upper six bits and the following byte is the LE encoding of the value. Valid only for values $64$ through $2^{14} - 1$. |`69u8` | `0x1501`|
|         | Four-byte mode: Upper six bits and the following three bytes are the LE encoding of the value. Valid only for values $2^{14}$ through $2^{30} - 1$. |`65535u32` | `0xfeff0300`|
|         | Big-integer mode: The upper six bits are the number of bytes following, plus four. The value is contained, LE encoded, in the bytes following. The final (most significant) byte must be non-zero. Valid only for values $2^{30}$ through $2^{536} - 1$. |`65535u32` | `0xfeff0300`|
| Vector | Encoded by concatening the encodings of its items and prepending with the compactly encoded length of the vector. |`vec![1u8, 2u8, 4u8]` | `0x0c010204`|
| String | Encoded as `Vec<u8>` with UTF-8 characters. | `"SCALE♡"` | `0x205343414c45e299a1` |
| Tuple, Struct, Array | All three types are encoded by concatenating the encodings of their respective elements consecutively. |`(1u16, true, "OK")` | `0x010001084f4b`|
| | | `MyStruct{id: 1u16, is_val: true, msg: "OK"}`| `0x010001084f4b` |
| | |`[64u16, 512u16]` | `0x40000002`|
| Result | Results are encoded by prepending the encoded inner value with `0x00` if the operation was successful and `0x01` if the operation was unsuccessful. |`Ok(42)` | `0x002a`|
| |  |`Err(false)` | `0x0100`|
| Option | Options are encoded by prepending the inner encoded value of `Some` with `0x01` and encoding `None` as `0x00`. |`Some(69u8)` | `0x0100`|
|  |  |`None` | `0x00`|
| Enum | Enums are encoded by prepending the relevant u8 index, followed by the value if present. | `Example::Second(8u16)` | `0x010800` |