# KMP Java direct actualization

* **Type**: Design proposal
* **Author**: Nikita Bobko
* **Discussion**: todo
* **Status**: Submitted
* **Related YouTrack issue**: [KT-67202](https://youtrack.jetbrains.com/issue/KT-67202)

In Kotlin, there are **two ways** to write an actual declaration for the existing expect declaration.
You can either write an actual declaration with the same FQN as its appropriate expect (From now on, we will call such actualization a _direct actualization_),
or you can use `actual typealias`.

**The first way.**
_direct actualization_ has a nice property that two declarations share the same FQN.
It's good because when users move code between common and platform fragments, their imports stay unchanged.
But _direct actualization_ has a "downside" that it doesn't allow declaring actuals in external binaries (jars or klibs).
In other words, expect declaration and its appropriate actual must be located in the same "compilation unit".
Below, we will say why, in fact, it's not a "downside" but a "by design" restriction that models the reality. todo

**The second way.**
Contrary, `actual typealias` forces users to change the FQN of the final actual declaration.
(An attempt to specify the very same FQN in the `typealias` target leads to `RECURSIVE_TYPEALIAS_EXPANSION` diagnostic)
But we gain the possibility to declare expect and actual declarations in different "compilation units".

> [!NOTE]
> Though it's a philosophical question what is "the real actual declaration" in this case.
> Is it the `actual typealias` itself (which is still declared in the same "compilation unit"), or is it the target of the `actual typealias` (which, in fact, can be declared in external jar or klib)?

|                                                                     | _Direct actualization_ | `actual typealias` |
| ------------------------------------------------------------------- | ---------------------- | ------------------ |
| Do expect and actual share the same FQN?                            | Yes                    | No                 |
| Can expect and actual be declared in different "compilation units"? | No                     | Yes                |

While `actual typealias` already allows actualizing Kotlin expect declarations with Java declarations (Informally: "Kotlin to Java actualization"), _direct actualization_ only allows Kotlin to Kotlin actualizations.
The idea of this proposal is to support _direct actualization_ for Kotlin to Java actualizations.

## The proposal

**(1)** Introduce `kotlin.jvm.KotlinActual` annotation in Kotlin stdlib
```
package kotlin.jvm;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface KotlinActual {
}

// Or in Kotlin

@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.SOURCE)
@SinceKotlin("2.1")
public annotation class KotlinActual
```

- The annotation is intended to be used on Java declarations
- **Any usage** of the annotation in Kotlin will be prohibited (even as type, even as reference)
- The annotation in Java will function similarly to the `actual` keyword in Kotlin
- The annotation is not marked as `@ExperimentalMultiplatform` since `OPT_IN_USAGE_ERROR` is not reported in Java
- `ElementType.FIELD` annotation target is not specified by design. It's not possible to actualize Kotlin properties with Java fields.
- If Kotlin stdlib infrastructure allows it, we should prefer to declare the annotation in Java programming language rather than Kotlin. (because the annotation is going to be used in Java)

**(2)** If Kotlin expect and Java class have the same FQN, Kotlin compiler should consider Kotlin expect class being actualized with the appropriate Java class.
In other words, support _direct actualization_ for Kotlin to Java actualization.

**(3)** Kotlin compiler should require using `KotlinActual` on the Java top level class and its members that actualize

## Technical challenge:


## Motivation

As stated in the introduction, _direct actualization_ and `actual typealias` have different 

## `actual` keyword is 

## Where to report expect actual diagnostics

## Direct actualization doesn't permit placing expect and actual in different "compilation units", and it's by design

## Alternative design considered

Annotation on expect

















## Where to report expect-actual errors?


Currently, it's possible to actualize expect Kotlin class with "third-party" classes using `actual typealias`.
The "third-party" class could be either a Java class, or another Kotlin class with a different FQN (Fully qualified name).
The third-party class could come in the form of sources (located in the same compilation unit), or be an external library (jar file or Klib).
`actual typealias` imposes an unfortunate restriction: FQNs (Fully qualified name) of the third-party class and expect class must be different (it's not possible to create actual typealias that "targets itself").

The proposal is to introduce a mechanism to actualize expect classes with third-party classes but make it possible to use the same FQN.

## Motivation

One of the `actual typealias` use cases is to commonize existing Java libraries.
The restriction to have different FQNs is pointless and sometimes even harmful in the "commonize existing Java libraries" use case.

(1) `actual typealias` creates two identifiers for the same classifier, which creates the confusion on which FQN should be used

(2) Using the same FQN simplifies the migration of client code from Java library to KMP version of the same library.
Client code does not need to replace imports

(3) It's especially cumbersome to have duplicated FQNs for an entire API surface

(4) Later replacing the Java actualization with a Kotlin `actual class` requires keeping the `typealias` in place indefinitely

## The proposed solution. Implicit Java actualization

It's proposed to introduce a special annotation in common stdlib.
```
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS)
@ExperimentalMultiplatform
@MustBeDocumented
@SinceKotlin("2.1")
public annotation class ImplicitlyActualized
```


If the expect declaration marked with special 

## Repeatable annotation vs array of "targets"

## Alternative solution 1. actual typealias without target


## Alternative solution 2. Don't require `actual` keyword, always actualize implicitly

Never require `actual` keyword. Make "

## Feature interaction with hierarchical multiplatform

## Feature interaction with explict actuals

When an explict `actual` declaration is presented, the error should be reported.

```kotlin
// module: common
@ImplicitlyActualized
expect class Foo

@ImplicitlyActualized
expect class Bar

// module: platform
actual class Foo // error should be reported

actual typealias Bar = BarImpl // error should be reported

class BarImpl
```

todo analogy with `operator`

## Implicit actualization to OptIn / Deprecated

???
