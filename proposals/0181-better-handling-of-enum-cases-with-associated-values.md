# String Index Overhaul

* Proposal: [SE-0181](0181-better-handling-of-enum-cases-with-associated-values.md)
* Author: [Sash Zats](https://github.com/zats)
* Review Manager:
* Status:
* Decision Notes:
* Pull Request:

## Introduction

Enum cases with associated values are very useful tools when modeling data
in Swift. However it has a somewhat tedious aspect associated with them
(no pun intended).
When dealing with values associated with particular case, we have to explicitly
declare each one of them even though usually we just want to use the same names
as case had declared initially. This introduces additional friction and makes
me consider other options even when enum is a good fit.

## Motivation

Consider following example:

```swift
enum Value {
  case container(leafs: [Value])
  case leaf(data: Any?, moreData: Any?, evenMoreData: Any?)
}

let value = Value.leaf(data: nil, moreData: someOtherData, evenMoreData: nil)

switch value {
case let .leaf(data, moreData, evenMoreData):
  print(data, moreData, evenMoreData)
case .container:
  break
}
```

This gets even more tedious when case has more values associated with it.
It usually pushes me to use other solutions, that make code less expressive
(e.g. falling back to `struct`s instead of `enum`s)

## Proposed solution

Allow new type of construct to be used with `switch` case:

```swift
switch value {
case let x = .leaf:
  print(x.data, x.moreData, x.evenMoreData)
case .container:
  break
}
```

## Detailed design

I do not have in-depth knowledge of the Swift internals, so following is a
speculation more than detailed design.
Proposed solution can generate implicit `struct` types for each case with
associated values:

```swift
struct _ValueLeaf {
  let data: Any?
  let moreData: Any?
  let evenMoreData: Any?
}
```

Accessing the field without declaring explicit fields will implicitly type `x`
to `_ValueLeaf`:

```swift
// ...
case let x: _ValueLeaf = .leaf:
  print(x.data, x.moreData, x.eventMoreData)
// ...
```

## Source compatibility

This is an additive change, and shouldn't affect existent code.

## Effect on ABI stability

Not sure.

## Effect on API resilience

Not sure.

## Alternatives considered

The only alternative considered was no action.
