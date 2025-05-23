r[expr.if]
# `if` and `if let` expressions

## `if` expressions

r[expr.if.syntax]
```grammar,expressions
IfExpression ->
    `if` Expression _except [StructExprStruct]_ BlockExpression
    (`else` ( BlockExpression | IfExpression | IfLetExpression ) )?
```
<!-- TODO: The exception above isn't accurate, see https://github.com/rust-lang/reference/issues/569 -->

r[expr.if.intro]
An `if` expression is a conditional branch in program control.
The syntax of an `if` expression is a condition operand, followed by a consequent block, any number of `else if` conditions and blocks, and an optional trailing `else` block.

r[expr.if.condition-bool]
The condition operands must have the [boolean type].

r[expr.if.condition-true]
If a condition operand evaluates to `true`, the consequent block is executed and any subsequent `else if` or `else` block is skipped.

r[expr.if.else-if]
If a condition operand evaluates to `false`, the consequent block is skipped and any subsequent `else if` condition is evaluated.

r[expr.if.else]
If all `if` and `else if` conditions evaluate to `false` then any `else` block is executed.

r[expr.if.result]
An if expression evaluates to the same value as the executed block, or `()` if no block is evaluated.

r[expr.if.type]
An `if` expression must have the same type in all situations.

```rust
# let x = 3;
if x == 4 {
    println!("x is four");
} else if x == 3 {
    println!("x is three");
} else {
    println!("x is something else");
}

let y = if 12 * 15 > 150 {
    "Bigger"
} else {
    "Smaller"
};
assert_eq!(y, "Bigger");
```

r[expr.if.let]
## `if let` expressions

r[expr.if.let.syntax]
```grammar,expressions
IfLetExpression ->
    `if` `let` Pattern `=` Scrutinee _except [LazyBooleanExpression]_ BlockExpression
    (`else` ( BlockExpression | IfExpression | IfLetExpression ) )?
```

r[expr.if.let.intro]
An `if let` expression is semantically similar to an `if` expression but in place of a condition operand it expects the keyword `let` followed by a pattern, an `=` and a [scrutinee] operand.

r[expr.if.let.pattern]
If the value of the scrutinee matches the pattern, the corresponding block will execute.

r[expr.if.let.else]
Otherwise, flow proceeds to the following `else` block if it exists.

r[expr.if.let.result]
Like `if` expressions, `if let` expressions have a value determined by the block that is evaluated.

```rust
let dish = ("Ham", "Eggs");

// this body will be skipped because the pattern is refuted
if let ("Bacon", b) = dish {
    println!("Bacon is served with {}", b);
} else {
    // This block is evaluated instead.
    println!("No bacon will be served");
}

// this body will execute
if let ("Ham", b) = dish {
    println!("Ham is served with {}", b);
}

if let _ = 5 {
    println!("Irrefutable patterns are always true");
}
```

r[expr.if.let.else-if]
`if` and `if let` expressions can be intermixed:

```rust
let x = Some(3);
let a = if let Some(1) = x {
    1
} else if x == Some(2) {
    2
} else if let Some(y) = x {
    y
} else {
    -1
};
assert_eq!(a, 3);
```

r[expr.if.let.desugaring]
An `if let` expression is equivalent to a [`match` expression] as follows:

<!-- ignore: expansion example -->
```rust,ignore
if let PATS = EXPR {
    /* body */
} else {
    /*else */
}
```

is equivalent to

<!-- ignore: expansion example -->
```rust,ignore
match EXPR {
    PATS => { /* body */ },
    _ => { /* else */ },    // () if there is no else
}
```

r[expr.if.let.or-pattern]
Multiple patterns may be specified with the `|` operator. This has the same semantics as with `|` in `match` expressions:

```rust
enum E {
    X(u8),
    Y(u8),
    Z(u8),
}
let v = E::Y(12);
if let E::X(n) | E::Y(n) = v {
    assert_eq!(n, 12);
}
```

r[expr.if.let.lazy-bool]
The expression cannot be a [lazy boolean operator expression][expr.bool-logic].
Use of a lazy boolean operator is ambiguous with a planned feature change of the language (the implementation of if-let chains - see [eRFC 2947][_eRFCIfLetChain_]).
When lazy boolean operator expression is desired, this can be achieved by using parenthesis as below:

<!-- ignore: pseudo code -->
```rust,ignore
// Before...
if let PAT = EXPR && EXPR { .. }

// After...
if let PAT = ( EXPR && EXPR ) { .. }

// Before...
if let PAT = EXPR || EXPR { .. }

// After...
if let PAT = ( EXPR || EXPR ) { .. }
```

[_eRFCIfLetChain_]: https://github.com/rust-lang/rfcs/blob/master/text/2497-if-let-chains.md#rollout-plan-and-transitioning-to-rust-2018
[`match` expression]: match-expr.md
[boolean type]: ../types/boolean.md
[scrutinee]: ../glossary.md#scrutinee
