---
layout: post
published: false
title: "Password hashing: Some thoughts on Catena"
description: Exploring some ideas around extending the basic Catena password hashing framework.
---


### Lambda cyclical bit reversal
Problem that bit-reversal is cyclical after two passes. Ideally want a lambda-cyclical transform, that still results in nodes close in layer N being far away in layer N-1 (but also in layer N-1 to 0). Cyclical in lambda means that layer 0 and the final layer are identical, but layer 0 is easily computed anyway so this is not a significant weakness. The advantage is that the first and last passes are sequential.

Suggest a lambda-cyclical bit reversal. For the simple case of an even number of bits and lambda=4 (four bit groups), it progresses like this (capital letter is bit reversed lowercase).

    abcd
    bDAc
    DCBA
    CadB
    abcd

The least significant bit moves up to the 1/4 most significant bit. For a garlic value of 4, the node graph looks like this.

[Insert graph]

### Using Keccak single pass

### Potential to multicore the inner passes
Want mixing between threads, so the nodes that thread X computes in layer N must depend on nodes computed by all other threads in layer N-1.