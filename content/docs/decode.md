---
title: "Decode"
weight: 3
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
math: true
---


# 1. Decoding
Since SCALE is non-descriptive, the proper metadata is needed to decode raw bytes into the appropriate types.
## 1.1 Depth Limit
Greater complexity in the decode type leads to increased computational resources used for value decoding. Generally you always want to `decode_with_depth_limit`. Substrate uses a limit of `256`.

## 1.2 Optimizations and Tricks
- DecodeLength: Read the length of a collection (like Vec) without decoding everything.
- EncodeAppend: Append an item without decoding all the other items. (like Vec)

