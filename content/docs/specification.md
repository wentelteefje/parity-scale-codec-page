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

SCALE defines encodings for the most elementary types. Encodings for more complex types are obtained by concatenating the encodings of their constituents – that is, the simple types that form these respective compound types. Encodings for variable-length types have length data prepended.

This table offers a concise overview of the SCALE codec. For more detailed, hands-on explanations, please refer to the [encode section]({{< ref "/docs/encode" >}}). For the formal specification, please refer to the [Polkadot specification](https://spec.polkadot.network/id-cryptography-encoding#sect-scale-codec). Both the intermediary hexadecimal representation and the final byte-array SCALE encoding are provided to enhance readability.

| Data type | Encoding Description |  SCALE decoded value	| SCALE encoded value |
| --        | --          | -- | -- |
| Unit | Encoded as an empty byte array. | `()` | `` |
| | | | `[]` |
| Boolean    | Encoded using the least significant bit of a single byte. | `true` | `0x01` |
| | | | `[01]` |
|           |                                                           | `false`| `0x00` |
| | | | `[00]` |
| Integer | By default integers are encoded using a fixed-width little-endian format. | `69i8` | `0x45`|
| | | | `[45]` |
|         |                                                                      | `69i16`| `0x4500`|
| | | | `[00, 45]` |
|         |                                                                      | `42u16`| `0x2a00`|
| | | | `[00, 2a]` |
|         | Non-negative integers $n$ also have a compact encoding. There are four modes.| | | |
|         | Single-byte mode: Upper six bits are the LE encoding of the value. For $0 \leq n \leq 2^6 - 1$. |`0u8` | `0x0`|
| | | | `[00]` |
|         | Two-byte mode: Upper six bits and the following byte is the LE encoding of the value. For $2^6 \leq n \leq 2^{14} - 1$. |`69u8` | `0x1501`|
| | | | `[01, 15]` |
|         | Four-byte mode: Upper six bits and the following three bytes are the LE encoding of the value. For $2^{14} \leq n \leq 2^{30} - 1$. |`65535u32` | `0xfeff0300`|
| | | | `[00, 03, ff, fe]` |
|         | Big-integer mode: The upper six bits are the number of bytes following, plus four. The value is contained, LE encoded, in the bytes following. The final (most significant) byte must be non-zero. For $2^{30} \leq n \leq 2^{536} - 1$. |`65535u32` | `0xfeff0300`|
| | | | `[00, 03, ff, fe]` |
| Vector | Encoded by concatening the encodings of its items and prepending with the compactly encoded length of the vector. |`vec![1u8, 2u8, 4u8]` | `0x0c010204`|
| | | | `[04, 02, 01, 0c]`|
| String | Encoded as `Vec<u8>` with UTF-8 characters. | `"SCALE♡"` | `0x205343414c45e299a1` |
| | | | `[a1, 99, e2, 45, 4c, 41, 43, 53, 20]`|
| Tuple, Struct, Array | All three types are encoded by concatenating the encodings of their respective elements consecutively. |`(1u16, true, "OK")` | `0x010001084f4b`|
| | | | `[4b, 4f, 08, 01, 00, 01]`|
| | | `MyStruct{id: 1u16, is_val: true, msg: "OK"}`| `0x010001084f4b` |
| | | | `[4b, 4f, 08, 01, 00, 01]`|
| | |`[64u16, 512u16]` | `0x40000002`|
| | | | `[02, 00, 00, 40]`|
| Result | Results are encoded by prepending the encoded inner value with `0x00` if the operation was successful and `0x01` if the operation was unsuccessful. |`Ok(42)` | `0x002a`|
| | | |  `[2a, 00]` |
| |  |`Err(false)` | `0x0100`|
| | | |  `[00, 01]`|
| Option | Options are encoded by prepending the inner encoded value of `Some` with `0x01` and encoding `None` as `0x00`. |`Some(69u8)` | `0x0100`|
| | | |  `[00, 01]`|
|  |  |`None` | `0x00`|
| | | |  `[00]`|
| Enum | Enums are encoded by prepending the relevant u8 index, followed by the value if present. | `Example::Second(8u16)` | `0x010800` |
| | | |  `[00, 08, 01]`|