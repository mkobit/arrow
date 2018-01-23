---
layout: docs
title: MonadFilter
permalink: /docs/typeclasses/monadfilter/
---

## MonadFilter

`MonadFilter` is a type class that abstracts away the option of interrupting computation if a given predicate is not satisfied. 

All instances of `MonadFilter` provide syntax over their respective data types to comprehend monadically over their computation:

## continueWith

Binding over `MonadFilter` instances with `bindingFilter` brings into scope the `continueIf` guard that requires a `Boolean` predicate as value. If the predicate is `true` the computation will continue and if the predicate returns `false` the computation is short-circuited returning monad filter instance `empty()` value. 

In the example below we demonstrate monadic comprehension over the `MonadFilter` instances for both `Option` and `ListKW` since both data types can provide a safe `empty` value. 

When `continueIf` is satisfied the computation continues

```kotlin
import arrow.*
import arrow.core.*
import arrow.mtl.*

Option.monadFilter().bindingFilter {
 val a = Option(1).bind()
 val b = Option(1).bind()
 val c = a + b
 continueIf(c > 0)
 yields(c)
}
//Some(2)
```

```kotlin
import arrow.data.*

ListKW.monadFilter().bindingFilter {
 val a = listOf(1).k().bind()
 val b = listOf(1).k().bind()
 val c = a + b
 continueIf(c > 0)
 yields(c)
}
//ListKW(list=[2])
```

When `continueIf` returns `false` the computation is interrupted and the `empty()` value is returned 

```kotlin
Option.monadFilter().bindingFilter {
 val a = Option(1).bind()
 val b = Option(1).bind()
 val c = a + b
 continueIf(c < 0)
 yields(c)
}
//None
```

```kotlin
ListKW.monadFilter().bindingFilter {
 val a = listOf(1).k().bind()
 val b = listOf(1).k().bind()
 val c = a + b
 continueIf(c < 0)
 yields(c)
}
//ListKW(list=[])
```

##bindWithFilter

Binding over `MonadFilter` instances with `bindingFilter` brings into scope the `bindWithFilter` guard that requires a `Boolean` predicate as value getting matched on the monad capturing inner value. If the predicate is `true` the computation will continue and if the predicate returns `false` the computation is short-circuited returning the monad filter instance `empty()` value. 

When `bindWithFilter` is satisfied the computation continues

```kotlin
Option.monadFilter().bindingFilter {
 val a = Option(1).bind()
 val b = Option(1).bindWithFilter { it == a } //continues
 val c = a + b
 yields(c)
}
//Some(2)
```

```kotlin
ListKW.monadFilter().bindingFilter {
 val a = listOf(1).k().bind()
 val b = listOf(1).k().bindWithFilter { it == a } //continues
 val c = a + b
 yields(c)
}
//ListKW(list=[2])
```

When `bindWithFilter` returns `false` the computation short circuits yielding the monad's empty value

```kotlin
Option.monadFilter().bindingFilter {
 val a = Option(0).bind()
 val b = Option(1).bindWithFilter { it == a } //short circuits because a is 0
 val c = a + b
 yields(c)
}
//None
```

```kotlin
ListKW.monadFilter().bindingFilter {
 val a = listOf(0).k().bind()
 val b = listOf(1).k().bindWithFilter { it == a } //short circuits because a is 0
 val c = a + b
 yields(c)
}
//ListKW(list=[])
```
