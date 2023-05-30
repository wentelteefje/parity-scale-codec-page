---
title: "Intro"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

![logo](logo.png)

SCALE (<u>S</u>imple Concatenated Aggregate Little-Endian) is the data format for types used in the Parity Substrate framework.

SCALE is a light-weight format which allows encoding (and decoding) which makes it highly suitable for resource-constrained execution environments like blockchain runtimes and low-power, low-memory devices.

SCALE is non-descriptive. This means, that the encoding context (knowledge of how the types and data structures look) needs to be known separately at both encoding and decoding ends. The encoded data does not include this contextual information.

The Rust implementation of SCALE is https://github.com/paritytech/parity-scale-codec.