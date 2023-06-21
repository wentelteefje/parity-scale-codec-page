---
title: "SCALE crates"
weight: 6
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
math: true
---

# 1. `scale-info`
# 2. `scale-decode`
[Crates.io](https://crates.io/crates/scale-decode/)
[Github](https://github.com/paritytech/scale-decode)
[docs.rs](https://docs.rs/scale-decode/latest/scale_decode/)

The `scale-decode` crate facilitates the decoding of SCALE-encoded bytes into custom data structures by using type information from a `scale-info` registry. By implementing the `Visitor` trait and utilizing the `decode_with_visitor` function, developers can map decoded values to their chosen types with enhanced flexibility.

{{<mermaid>}}
flowchart TD
    SD[[scale-decode]]
    SDT1(Visitor)
    SDT1-- trait -->SD
    SDT4("decode_with_visitor
    (bytes, typeid, registry, visitor)")-- function -->SD
    SDT4-->Decode("SCALE bytes decoded into custom data structure")
{{</mermaid>}}

# 3. `scale-value`