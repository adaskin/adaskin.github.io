---
layout: post
title:  "Thoughts on quantum programming languages"
date:   2024-06-06 12:31:19 +0300
categories: quantum-computing
tags: quantum programming language
comments: true
---
{% include mathjax.html %}

There are a few quantum programming languages (QPL). While looking at one of the promising QPL [Silq](https://silq.ethz.ch) and its comparison to other QPLs, I have thought how I would define some basics for a QPL. 



A generic compiler for a PL consists of syntax, semantic, optimization, and machine code generation parts that generates accurate assembly code the desired computation. 
Therefore the ultimate goal of a QPL should be also the ability to generate **accurate** quantum assembly code(e.g. QASM) from a given program.
 While doing this, one of the difficulty is to distinguish classical user-space code from quantum reality. 
 Since we write the code on classical machine, I think it should retain the basics such as constants, strings, loops and conditions. 
 And the compiler should remove the classical part of the code from the generated quantum code (Maybe it is for this reason, most people use quantum computer libraries such as Qiskit).

In addition to these basics, to be able to write fast code, I think the followings are the essentials for a QPL:
## Variables
We should be able to define variables such as `Register` type for a register or a qubit and `Operation` type for operations.  
Also, we should be able to give their initial value (it can be typed or duck-typed language).
For instance, the following C language family syntax can be used for these variables:
```
x = 1, y = 0;
Qubit a, b = 0, c = 1; 
Qubit d = {2, 3.5, normalize=true}, e = {1,2};
Register r1 = "111000";
Register r2 = "11{0.6,0.7}0{0.9,0.1}1";
```

## Operations
Syntax for the definition of an operation can be Python-numpy like or Matlab like:
```
Operation A = [[1, 2],[3, 4]];
Operation B = [1 2;3 4];
Operation C = MyOperation()
```
We can also define new Operation class by using the inheritance.
```
MyOperation:Operation{
}
```
 Since quantum computing is mainly a matrix-vector transformation, I think the  main theme for a QPL should focus on the following three type of operations:
- Qubit operations: These applies a single operation to a qubit, e..g:
    - `X.apply(q1)` applies an `X` to a register or qubit. 
    - Or controlled operations such as `X.apply(target, control)`.
- Vector operations: Applies an operation to part or parts of vector:  
    - `X.vec_apply(target_elements)`. 
    - This can be also,  `X.vec_apply(target_elements, control)` 
- Matrix operations such as swapping rows or columns. Or applying another matrix operation to this operation. 
    - `C.swap(i,j)`

## Measurement
Measurement is a kind of an operation too. Therefore, a class with a few related methods can be enough.

...

The rest is left for the developers:smile:

{% include disqus.html %}
