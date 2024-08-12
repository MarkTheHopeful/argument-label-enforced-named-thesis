# Enforced Named Arguments Form

## Description

As was stated in the introduction, by the named form of an argument or a named function call, we understand an argument passed in the function call with the name of the argument specified. The main idea of the enforced named arguments form feature is to allow developers to mark some of the arguments in a function (or perhaps the whole function) as requiring the named form.

This will require the following modifications to be implemented:
1. Add a way for a developer to mark an argument or a function (or a class? or a file?) as requiring the named form for the function calls (or specific arguments in these calls).
2. Transform this mark into a property of an argument, a function or any other entity
3. Verify, ensure, or check if an argument is marked as requiring the named form.

One possible example of such marking can be achieved by introducing a new parameter keyword (here --- `enf`), as can be seen on the following listing:

```kotlin
fun callMe(regular: Int, enf enforced: Int) {}

callMe(30, 566) // Compilation error
callMe(30, enforced=566) // Compiles
```

### Discussion history

The discussion was first recorded in the issue [KT-14934](https://youtrack.jetbrains.com/issue/KT-14934/Enforce-parameter-usage-only-in-named-form) on the Kotlin Youtrack. The discussion about argument labels was included at some point, but soon after creation, it was separated into a dedicated issue (more about that in its corresponding document).

Initially, the idea was to allow developers to specify for specific parameters that they should be provided only with arguments in named form, prohibiting the positional usage of such arguments.

It was stated that this behaviour would be useful for the parameters that are not direct input of a function, but rather some options that affect the function behaviour, like Boolean flags (for which there is already an [inspection present](https://youtrack.jetbrains.com/issue/KTIJ-1634), and a request for more sophisticated one [exists for three years now](https://youtrack.jetbrains.com/issue/KTIJ-18880/Consider-more-sophisticated-inspection-for-same-looking-arguments-without-corresponding-parameter-names)).

The initial idea is still present in this work, with some additional ideas to consider, including applying the modifier to an argument, functions, and other units (such as classes and files)

There were many further discussions both in the issue (and a few other issues) and in the posts at [Kotlin Discussions](https://discuss.kotlinlang.org/t/add-annotation-or-other-mechanism-to-force-named-arguments-for-method/15636) revolving around these features. Several findings and points from them are present in this document.

### Possible benefits

Adding a feature to a programming language, especially a big one, requires much work and can lead to significant changes and increased support and attention from the language development team. Therefore, it needs proper motivation and requests from the language community, which is the case in this situation.

Two common reasons between this feature and Argument Labels are already described in the introduction document, even though we still can briefly mention them here:

1. Introduction of enforced named form can enforce explicit indication of the meaning of the arguments passed, especially when the arguments are literals or objects constructed in place. Such direct indication of meaning makes the code more self-documenting
2. Changing APIs and libraries under development can benefit from enforcing named form for the arguments that can be affected by changes in future. When all such arguments are passed strictly in named form, they can freely be reordered on the developer's side without any changes needed from the end user. Apart from that, new parameters with default values could be added to the function, and it would not break the existing calls regardless of the place they were added. Bonus: trailing lambdas are usually the last argument (and positional), which makes newly added arguments to be second-to-last.

More about these reasons can be read in the introduction document, while more specific reasons are described here.

#### Explicit indication of arguments meaning

Even though this benefit is already partly described in the introduction, restating it once more with more examples and references can be helpful.

Suppose a function has many parameters of the same type, especially when this type is primitive and even more when the arguments are passed as constants or objects created at the call site. In that case, it becomes easy to mix up different parameters, ultimately resulting in hard-to-detect bugs.

Most straightforward examples of this kind include functions with boolean flags to configure the function’s behaviour or various numerical parameters being passed to configure or tune the function's behaviour.

**Enforcing named arguments form** can be used to solve this problem. The developer can mark the function or some parameters as requiring a named form. Then, everyone using it will have to specify the names of arguments, explicitly indicating the meaning of passed arguments.

The problem is not a new thing, so there are already related solutions as boolean literal inspection (solution to [KTIJ-1634](https://youtrack.jetbrains.com/issue/KTIJ-1634/Add-inspection-for-boolean-literals-passed-without-using-named-parameters-feature)) as well as requests for more in-depth inspections ([KTIJ-18880](https://youtrack.jetbrains.com/issue/KTIJ-18880/Consider-more-sophisticated-inspection-for-same-looking-arguments-without-corresponding-parameter-names))

There is even a section in Coding Conventions about it: “Use the named argument syntax when a method takes multiple parameters of the same primitive type, or for parameters of `Boolean` type, unless the meaning of all parameters is clear from the context.” ([Named arguments](https://kotlinlang.org/docs/coding-conventions.html#named-arguments))

Talking about more specific examples, suppose we have the following function declaration:

```kotlin
fun reformat(
    str: String,
    normalizeCase: Boolean = true,
    upperCaseFirstLetter: Boolean = true,
    divideByCamelHumps: Boolean = false,
    wordSeparator: Char = ' ',
) { /*...*/ }
```

Compare two following ways to call it:

```kotlin
reformat("String!", false, false, true, '_')
// VS
reformat(
    "String!",
    normalizeCase = false,
    upperCaseFirstLetter = false,
    divideByCamelHumps = true,
    wordSeparator = '_'
)
```

Even though the second one looks way more verbose, the meaning of the first one is impossible to understand without looking at the function declaration.

Here are another couple of examples from actual code from a Kotlin machine-learning library: [kotlindl](https://github.com/Kotlin/kotlindl):

```kotlin                                                                                                                                                 
Conv2D(
        filters = 16,
        kernelSize = intArrayOf(5, 5),
        strides = intArrayOf(1, 1, 1, 1),
        activation = Activations.Tanh,
        kernelInitializer = GlorotNormal(SEED),
        biasInitializer = Zeros(),
        padding = ConvPadding.SAME
    )
```

```kotlin
model.use {
    it.compile(
        optimizer = Adam(),
        loss = Losses.SOFT_MAX_CROSS_ENTROPY_WITH_LOGITS,
        metric = Metrics.ACCURACY
    )

    it.printSummary()

    it.fit(
        dataset = train,
        epochs = 10,
        batchSize = 100
    )

    val accuracy = it.evaluate(dataset = test, batchSize = 100).metrics[Metrics.ACCURACY]

    println("Accuracy: $accuracy")
    it.save(File("model/my_model"), writingMode = WritingMode.OVERRIDE)
}
```

It is difficult to find examples of the use of these functions without the usage of named form, but if one were to use it, the meaning of the calls and purposes of the arguments would be impossible to guess. So, to prevent it, it makes perfect sense to mark such functions or these parameters as parameters with enforced named argument form.

#### Remark: some statistics about "bad" function calls
 
According to the data obtained via different sources (including search by regexp on GitHub), functions with multiple primitive arguments are widely present in the global codebase. Examples include Android Development, various UI functions, utilities to work with images or canvases and more. 

Some of the examples are part [of an Android application, responsible for working with a database](https://github.com/GrzegorzDawidek/AndroidZal/blob/e942947fdd01b3191f472cf378e46c6523a93721/app/src/main/java/com/example/sqllitetask/DatabaseHelper.kt), a [simple image utility](https://github.com/2T-T2/ImageUtil/blob/acc3eb444365caf89e014de9747831b0fec9cbe6/src/ImageUtil.kt) and a [small part of a shopping app](https://github.com/jiangyuanyuan/KotlinMall/blob/0e58f238614a4ba2644712ce4190290c9e19bed0/GoodsCenter/src/main/java/com/kotlin/goods/presenter/GoodsDetailPresenter.kt).
 
While those are definitely not examples of clean and good code, the fact is that people write like that, and they use Kotlin, and, probably, some people even depend on their code. 
 
If we are to move to concrete data, GitHub search by file using regular expression, we can get the following numbers:

| Amount of arguments, at least `x` | Files with at least `x` positional arguments | Files with at least `x` literal arguments |
|-----------------------------------|----------------------------------------------|-------------------------------------------|
| 1 | 11400000 | 8700000 |
| 2 | 7900000 | 2000000 |
| 3 | 3600000 | 553000 |
| 4 | 1800000 | 314000 |
| 5 | 745000 | 151000 |
| 6 | 446000 | 89600 |
| 7 | 267000 | 51500 |
 
This does look like a significant portion of the codebase and may serve as an argument for introducing the Enforced Named Arguments Form.

#### Non-trivial or confusing order of arguments

If, for some reason, a function has an unexpected order of arguments, that can lead to confusion and further bugs, especially if some of those parameters have the same type, it might be helpful to mark those arguments or the whole function as requiring a named argument form. 

#### Accidental trailing lambda

Sometimes, one has a function that, for some reason, has a functional type as its last parameter, which is not actually supposed to be used as trailing lambda.

One possible example of such can be functions that operate on two or more equally important functions, which all are just parameters, without the specific meaning of a callback, something to be executed directly or anything else, such as the following:

```kotlin
foo(onSuccess = {...}, onFail = {...})
groupBy(keySelector = {...}, valueSelector = {...})
```

It would be confusing to pass one of the functional parameters here as a regular parameter and the other as a trailing lambda. Therefore, it can be helpful to prohibit this parameter from being used in the positional form. Perhaps one can mark their function, class or module with a special `no-trailing-lambda` keyword, annotation or compilation flag.

#### Allowing using diffent lambdas as trailing

Currently in Kotlin lambda can only be trailing if it is specificed as last argument of a function. This could create some possible disbalance in situations where you might want to have the opportunity for two different function-arguments to accept a proper lambda. One, for example, can imagine the function accepting the `onSuccess` and `onError` functions as arguments. In some situations you might want to have proper `success` processing, and thus passing it as a trailing lambda, while passing `onError` to some other handler, and in some situations you might want to do the contrary. But as for now, you can only put one argument as the "last" one in the function declaration, thus only one of `onSuccess` and `onError` will be available for "trailing" use.

Perhaps not directly related to the enforcement of named arguments form, but one idea can be suggested for this problem. What if we do it so any lambda can be passed as trailing if all the others are specified in named form? That does solve the situation, and does not create ambiguity, at least not from the compiler side.

The downside of this approach is that the user, reading this code for their first time, would still have to somehow understand what was the name of that last trailing argument. And to deduce, which name is "missing" they would have to read the function declaration anyway.

Nevertheless, the idea and the problem itself can be mentioned and considered.

### Possible drawbacks

#### Additional clutter on the call sites

Currently, most IDEs highlight argument names when passing them in positional form. If so, then why would one need to write additional code?

There are several counterarguments to this position:
1. Not everyone has such kind of IDE. One most natural example of such a situation is code review in GitHub.
2. It only works against the "self-documentation" point. The enforcement is still needed if it is done for a different reason (as with lambdas).

## Ways to implement

After a basic introduction and the vision behind why this feature could be implemented (or not), we should discuss what needs to be implemented, what things need to be considered, and how the feature can be implemented.

### The formal task

By "adding a way to enforce named argument form", we will denote the presence of the following changes to the Kotlin compiler:

1. The syntax to mark a parameter, a function, a class or a whole file as requiring the named form of arguments for function calls of/in the marked declarations. The behaviour should not be affected if such a "mark" is absent. The arguments it affects will be referred to here as "requiring named form".
2. During the argument to parameter mapping, if an argument requires the named form and is currently being passed in the call using the positional form, a warning or an error should be produced.

### Things to consider

#### Different areas of effect

It was already mentioned several times that we might want to enforce named arguments form for different entities on different levels, from more specific to global:

1. Mark a specific argument as requiring the named form. Simple, concise, and point-wise. It blows up the size of the declaration if one wants more than one or two arguments to have this property
2. Mark all arguments starting from a specific one requiring a named form. It works well if a function has a large "tail" of optional parameters for configuration. This choice can lead to confusion, as one will somehow need to place a "separator", which will not be related to a specific parameter by itself but rather to a range of parameters. (One can say that one will only need to put the mark on the first such parameter, but since in Kotlin, one can mix named and positional arguments, this is not going to work)
3. Mark a function as requiring a named form. Also simple, all the arguments of such a function will require the named form. It is the same as 2., but without the inconsistency about the modifier being put somewhere in the middle. It has problems when one might want to keep some arguments in positional form, like the first one (which is often the function's direct object).
4. Mark a whole class as requiring a named form. If one decides that in a class, one will mark all functions as requiring the named form or wants to write the code in Swift style, one may want to enforce a named form in each method. It has the same problem as the 3., but in addition, users may forget about the modifier being present somewhere at the beginning of the class.
5. Mark the whole file — no additional comments here, basically the same as the previous point.
6. Mark the whole project using the named form. It may be helpful if one is working on a library, which can still be in development, or wants to enforce this specific code style in the project on the compiler level.

Everything starting from the 3. level seems to be overkill, especially regarding the function's first argument. Moreover, what if we want to prohibit only a specific pattern of usage of positional form? More about that in the next part.

#### Different modes of operation

Maybe we want to use not only one specific keyword to enforce the named form for something but perhaps something more, especially since we have several reasons to use them, and simple enforcement on different levels has disadvantages. The possible proposed modes/enforcement types are the following:

1. Named-only: the regular one enforces the entity marked to have all parameters requiring the named argument form. It can be used for functions where all arguments can easily be mixed up, such as functions with `Boolean` arguments, math functions, or many others discussed earlier
2. Positional-only: the inverse of the previous one enforces a parameter to be used in the positional form only. It can be used for methods with overloads in case the parameter name was changed in the overload or for functions where the positional form is highly unexpected (such as `equals`, operators or infix functions)
3. Mixed: simple to describe: we want some parameters to be used in the positional form while enforcing the named form for others. Examples can be seen in functions like `minOf(a, b, comparator, selector)` where `a` and `b` are positional, and `comparator` and `selector` are named. That sounds good, but how do we implement it from a design point of view? Mark the function as "named only" and the first parameters as "unnamed"?
4. No-trailing-lambda: a specific mode that prohibits trailing lambdas for only a function, class, or more. Targets that one described issue specifically. It may be helpful to apply it for all classes or libraries instead of enforcing each last functional parameter to be named.

Properly supporting these modes would require more than one kind of mark. What comes to mind is something of the kind: "named", "positional", and "any". Do we need the third one?

#### Remark: global paradigm shift

Swift programming language, which may be referenced as the source of the initial ideas, enforces named form by default. Maybe we should change the Kotlin paradigm and enforce the named argument form as a default? Obviously, with a grace period, a way to use an unnamed form and, perhaps, some smart way to not enforce the named form when the passed variable has the same name as the parameter it is being passed into.

#### Remark: Synergy with Argument Labels

Both features were initially discussed together and are present in Swift; therefore, it makes sense that their joined implementation can lead to additional benefits.

For example, one could say that using an "empty" Argument Label can be counted as specifying this parameter as "positional-only" and providing an actual argument label --- as specifying "named-only". This approach has disadvantages (like two different (although related) behaviours linked to one feature and being a complete rip-off of Swift), but it is still worth investigating.

#### Level of diagnostic 

Instead of throwing errors/warnings on the Compiler level, one may propose this feature as an IDE warning and inspection. This idea sounds like a possible approach, but if it will work only in IDE, where parameter names are already highlighted even when one does not write them, this approach seems dubious.

#### Migration levels

If a library is to introduce the usage of enforced named argument form in its functions, this will affect all the code that uses these functions. How should it affect it?

1. Perhaps errors should be generated, breaking the code using such functions in positional form. This approach will break the usage but could increase the adoption speed of the named form. Moreover, it could uncover some possible bugs in the process.
2. Perhaps warnings should be generated, only reporting the usage of marked functions or arguments in positional form. This approach will not break anything, but it seems possible that many will just ignore such warnings, which effectively nullifies the purpose of the feature.
3. Combine these two in some way. Maybe leave it to the library developer to set a configuration flag to determine whether to throw an error or a warning. Maybe make it a suppressible error or warning via compiler flags.

#### Binary compatibility

Another essential topic and reason behind this feature (and argument labels) is trying to preserve binary compatibility during function evolution. 

Is it possible to achieve this using the named form? How are these named calls being compiled against library functions? How does function ABI change when an optional parameter is introduced?

Another approach is to move the parameters subject to change into an "Argument Object" --- a data class holding the parameters, which can be easily constructed at the call site and deconstructed inside the function. While being a theoretical proposal, it may be further used for ABI instead of ENF and AL. More about this is in its specific document.

#### Data class destruction

Quote from the original issue: "The related issues is a position-based destructing for data classes. If the way to enforce parameter usage in named form is implemented, then it would be logical to extend it all the way to the data-classes restructuring. That is, if constructor parameters' usage is somehow enforced to be in named form, then positional restructuring for those parameters shall be disabled, too."

### Existing solutions

Before moving to the possible implementation details of our own, we should first try to look at how these features are emulated in Kotlin if they are, and how they are implemented in other programming languages.

#### Imitations in Kotlin: Variadic arguments

There is a way to enforce some arguments to always be in the named form --- it is the `vararg` keyword. More specifically, every argument after a variadic one must be used in the named form.

That makes it a way in Kotlin to make some arguments of a function to consistently be in named form by using `vararg nothings: Nothing`. Currently, such a variadic parameter is forbidden (there is a compilation error, which can be suppressed), and it looks weird and takes space for, in fact, a separator.

```kotlin
/* requires passing all arguments by name */
fun f0(vararg nothings: Nothing, arg0: Int, arg1: Int, arg2: Int) {}
f0(arg0 = 0, arg1 = 1, arg2 = 2)    // compiles with named arguments
//f0(0, 1, 2)                       // doesn't compile without each required named argument

/* requires passing some arguments by name */
fun f1(arg0: Int, vararg nothings: Nothing, arg1: Int, arg2: Int) {}
f1(arg0 = 0, arg1 = 1, arg2 = 2)    // compiles with named arguments
f1(0, arg1 = 1, arg2 = 2)           // compiles without optional named argument
//f1(0, 1, arg2 = 2)                // doesn't compile without each required named argument
```

The example is taken from [stackoverflow](https://stackoverflow.com/questions/37394266/how-can-i-force-calls-to-some-constructors-functions-to-use-named-arguments/37394267#37394267), where the question about the possibility of doing this was initially asked.

This method was discussed in issue [KT-12846](https://youtrack.jetbrains.com/issue/KT-27282/Allow-vararg-parameters-of-type-Nothing) and deemed strange and “looks really like a hack”. The issue was closed in favour of the original issue for the introduction of enforcement of named argument form in Kotlin: [KT-14934](https://youtrack.jetbrains.com/issue/KT-14934/Enforce-parameter-usage-only-in-named-form).

A brief search on GitHub also showed that developers do not use this approach.

#### Imitations in Kotlin: Annotation

Many things that can be implemented as any kind of keyword or modifier can probably be implemented using either Compile- or Run-time annotation. The same goes with enforced named form: at some point it was proposed to imitate the feature behaviour using annotations, which [were requested](https://discuss.kotlinlang.org/t/add-annotation-or-other-mechanism-to-force-named-arguments-for-method/15636/2), and, sometime later, [implemented](https://github.com/chao2zhang/RequireNamedArgument).

```kotlin
@NamedArgsOnly
fun buildSomeInstance(param1: Boolean = true, param2: Boolean = false /* so on */)

// Ok
buildSomeInstance(param1=false, param2=true)

// Compilation error
buildSomeInstance(false, true)
```

The problem with this method is that it is an annotation, a mechanism that is (apparently) unreliable and heavily abused to modify the compiler’s behaviour. Moreover, this annotation is compile-time, and this approach will not work for libraries requiring consumers to specify the argument names. Perhaps a run-time annotation will work for this, but the part about the heavily used mechanism remains true.

#### Imitations in Kotlin: More inspections

We have already mentioned the [inspections for Boolean literals (implemented)](https://youtrack.jetbrains.com/issue/KTIJ-1634) and the [one for more sophisticated cases (requested)](https://youtrack.jetbrains.com/issue/KTIJ-18880/Consider-more-sophisticated-inspection-for-same-looking-arguments-without-corresponding-parameter-names). Perhaps the whole feature or feature set can be implemented as a set of such inspections? This idea was already discussed in the part "Level of diagnostic", but we already have argument names highlighting in IDE, so it would probably not be very useful to ask for argument labels only in the IDE.

### Abount named form in other languages

Currently, several popular programming languages do not even have the regular named form for passing arguments in a function call, like Java or C++. Still, some do have, and among them, at least one enforcing the usage of named form.

#### Approach in Swift

Once again --- Swift enforces all arguments to be in the named form by default, and the only way to use an argument in positional form is to specify it as having an empty argument label, which will render it impossible to use this argument in the named form.

An example from the [Swift documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/):

```swift
func fun(argumentLabel parameterName: Type) {}

func greet(person: String, from hometown: String) -> String {
    return "Hello \(person)!  Glad you could visit from \(hometown)."
}
print(greet(person: "Bill", from: "Cupertino"))
// Prints "Hello Bill!  Glad you could visit from Cupertino."
```

There is a long discussion on how and when to use the named form (outdated by eight years, but still, why not): [on swift forums](https://forums.swift.org/t/when-to-use-argument-labels-a-new-approach/1289)

It is essential to notice that all arguments in Swift **still have fixed positions**, and one cannot change their order despite the arguments being passed in named form. Perhaps due to this or argument labels being nothing more than separators in this situation, they do not have to be unique.

On the implementation side, all the arguments are still mapped positionally. However, the presence of the argument name is checked (if the argument label for this parameter was not set to "empty".

#### Remark: function arguments in Kotlin

Named arguments do not have to be passed in specific orders. Named and unnamed arguments can be mixed, but only when the values of all other arguments *can be deduced* from the named ones. That is not always clear. There can be only one variadic argument in a function, but it can be placed at any point of the arguments list, although all arguments after it have to be in a named form. Except for the case when the last argument is a (lambda) function, which does not have to be named if passed as a trailing lambda

### Specific implementation ideas

Now, after we move through the existing implementation, we need to think about which we can use for prototype or actual implementation, regarding their benefits and drawbacks.

#### Runtime annotation

We have already mentioned and discussed existing annotation, which allows us to enforce the named form for every argument in a function. The direct problem of that specific annotation was that it was compiled time, meaning that it was not preserved in the compilation artifact, making using the feature for libraries impossible.

One way to deal with it is just to use runtime annotation and an annotation processor, as they work in runtime rather than compile time, which is the idea for this approach --- analyze and rewrite this compile plugin into an annotation processor.

However, there are several downsides to this approach. Firstly, neither compiler plugins nor annotation processors support targets other than JVM, so adding a language feature in such a way will contradict Kotlin's multiplatformness. Secondly, annotations are already heavily overused mechanisms for such purposes, sometimes resulting in the code being less readable, which contradicts the very goal of this work. 

However, this approach can still be tried to explore this direction further.

#### Syntactic sugar: variadic argument insertion

Another approach highly based on the community idea, described in the corresponding part, is adding a separator-like entity in the argument list in function declarations, which will be compared to the variadic pseudo-argument construction we have seen before.

This way, writing something like `nowOnlyNamed` will be more apparent than writing the `vararg _: Nothing}` construction each time we want to introduce parameters with enforced named argument form, enhancing code clarity. Apart from that, this approach seems relatively easy to implement, as it requires a simple transformation of one specific construction to an existing language construction.

Still, even though we have fixed the problems with the obscurity of the construction, other problems described earlier persist, such as situations with functions already containing variadic arguments, usage of prohibited construction and limitation to the "all-after-separator" approach.

#### New keyword(s)

The more regular way to implement this feature is to add a new weak keyword to the Kotlin language, marking a function or an argument of a function as requiring a named form. Due to this keyword being weak, it will not affect parameters that already had the same name as the new keyword, like the existing weak parameter keywords: `vararg`, `noinline` and `crossinline`.

From the internal side, this feature is implemented by adding a boolean field to related nodes of source trees, along with proper changes to initialize those nodes. Apart from adding this flag, we will need to add analysis during the function calls resolution, which will check that all parameters marked with this flag are passed only using the named form.

The possible disadvantage of this approach is the requirement to modify many structures and places of initialization of such structures, which may be a tremendous amount of work.

The advantages of it, however, are that it is just a cleaner way to do the initial task and the possibility to easily extend it to function-level constructors and other places without significant problems.

#### Remark: on keywords in Kotlin

There is a difference between a soft keyword and a regular one. Regular or hard keywords (like `if`, `fun` or `break`) are always parsed as a keyword and cannot be used as an identifier. Meanwhile, soft keywords (like `vararg`, `catch` or `inline`) act as keywords in the context in which they are applicable; they are treated as identifiers. To be more precise, what in this work is called soft keywords can be split into soft keywords, modifier keywords and special identifiers, but we do not consider this separation in this work, even though what we are doing is, in fact, a modifier keyword. 

What is essential for us is that introducing a new soft keyword here will not break the existing code, even if it uses the identifier with the same name as the one we choose for the keyword.

### Possible technical details

This section collects different details and moments that should be noted during the implementation.

#### Scope of usage

It is clear where to put a keyword if we want it to affect an argument, a function or a class. However, where should it be placed if we want it to affect a subset of arguments or functions? Where should it be placed if we want it to act on the whole file or project? Perhaps keywords will not be of much use in these scopes.

#### Separate compilation

One of the significant reasons for this feature is for it to work with libraries. Libraries are usually compiled separately from the project using them. Therefore, we must somehow make the feature present in the compilation artifacts. How should it be stored? What are the possible options?

The one option that comes to mind first is to store just like any other keywords are stored in the .class file. `noinline` and `crossinline` are already being serialized somehow to .class files and/or Kotlin metadata. Therefore, it is possible to add our keyword there.

Are there any other options? Is it possible to do this without changing the binary format of .class files?

#### Interoperability with Java

Kotlin has to support interoperability with Java, which means Kotlin functions can be called from the Java code and vice versa.

Once again, Java does not support named arguments at all. How should our feature work in this situation? The three possible options stand out for possible discussion:
1. Java should simply ignore these modifiers. They are Kotlin metadata, and we are not ones to modify how Java Virtual Machine works in any way.
2. Such functions should not be callable in Java. Perhaps there is a way to prohibit these functions from executing in Java? However, why would we do it?
3. Make them possible to use with a warning being issued or leave the decision in the form of a compilation flag for a library --- the developer will decide whether the library should be accessible from Java.

Perhaps the second and the third approaches violate the interoperability. Do they?

#### Not-JVM backends

We have not fully discussed or looked at other backends for Kotlin apart from the JVM one. Still, if the feature is implemented on the sugar level, there should not be such a problem if the language in question supports named form and variadic arguments. 

#### Inheritance

We need to know how the parameter modifiers are preserved during the interface implementation/function overrides. Will the writer have to write the keyword for each argument if they want it to keep being enforced? Can one make an argument that had enforced named form to lose this property in a descendant? What about vice versa? How would it act in the calls via supertype?

#### Related diagnostics

The end user of the compiler should be provided with meaningful error messages in various new cases related to using positional/named form when it is prohibited. 

Also, should we add other diagnostics for impossible specification of "forced named" and "forced positional" forms? If somebody were to put a "forced positional" argument after a "forced named" argument, it would be impossible to use this function in some cases.

## Evaluation

The prototype for the enforced named arguments form feature was developed to test some ideas, collect additional findings, and provide a starting point for possible further implementations.

A prototype here is a version of a Kotlin compiler with modifications made to support the new feature. The feature can be covered by tests and different benchmarks in the prototype, but it can have poor code quality and/or questionable design choices. It may not work or be untested for some specific cases, which are primarily noted in the corresponding part.

### Prototypes implemented

It was decided that annotations were not a scope of our work for that moment, and the idea of inserting variadic arguments seemed to be too hacky to implement. Therefore, we decided to go on with the prototype with a new keyword. We decided to add only a keyword to enforce the named form of arguments, applicable to arguments, but this behaviour can easily be extended. Here, we will provide the details for the implementation.

#### Via new keyword

As we decided to introduce a new soft keyword, the work affected parsing, then the FIR node for value parameters, and then --- the process of function call resolution, namely the mapping of call arguments to function parameters. 

For the parsing stage, we needed to add a new soft keyword and make it applicable to function parameters, making it a modifier keyword. The keyword was integrated into the parser smoothly, as keywords already act as parameter modifiers, such as `vararg` and `noinline`. Therefore, this part of the task was to add our new keyword, `enf`, to the others. 

On the listing here, one can see how the introduced keyword looks in use:

```kotlin
fun callMe(regular: Int, enf other: Int) {}

callMe(30, 566) // Compilation error
callMe(30, other=566) // Compiles
```

After the parser modification, the next step was to add the corresponding boolean field indicating the enforcement, `isENF`, to the class for the FIR node responsible for storing a function parameter, namely `FirValueParameter`. 

With the modifier being parsed and its presence indicated in the FIR node, the remaining task was to add the logic for it ~--- if an argument is being passed to a parameter with this flag, a diagnostic, that means a proper message with the place and the reason for the error, must be generated, which results in a compilation error. The file for related logic is `FirArgumentsToParameterMapper.kt`, so the corresponding check was implemented there. 

After that, we only needed to add the diagnostic to produce there in case the named form was violated.

#### Remark: on the additional modification

One can notice that this implementation does not mention IR or serialization. The initial implementation did not affect them and, therefore, was not propagated into IR or .class, resulting in the modifier being lost after the compilation, rendering the feature useless for separate compilation.

However, additional work was done to introduce the related field into the IR tree node, with the serialization added likewise to the existing parameter modifiers (`noinline`, for example). Even though a large number of files needed to be changed to reflect the update (75!), in the end, the serialization and deserialization were working fine.

### Implementation results

The described prototype can be found in its branch in the GitHub repository: [prototype introducing a new keyword](https://github.com/MarkTheHopeful/kotlin/tree/enforced-named-proto)

It can be used in the following ways:

1. Checkout the needed branch
2. Run `./gradlew dist`
3. Use the compiler from `./dist/kotlinc/bin/kotlinc "filename"` to compile the file using the prototype

Now, we should move to the evaluation of the prototypes.

#### Tests and behaviour

In this prototype, the changes were more localized since the parsing stage was affected just slightly, and there was only one new field in the ValueParameter structures, so it did not change the behaviour on unrelated tests.

Even though there were multiple failing tests due to the new property, `isENF`, not being initialized by default in some places. Fortunately, this was fixed by locating such places and modifying them accordingly. After this, there were no more new failing tests, which was a success.

Additional tests were created to check that the behaviour is enforced with the presence of the keyword, although they currently do not cover all obscure cases. 

Separate compilation was tested separately and, after the modification mentioned earlier, worked correctly.

#### Benchmarks

The prototype was benchmarked on several tests, both with and without the presence of the new feature, and no significant change in the compilation time was noticed.

The benchmarks included tests with many (hundreds to thousands) functions with small or large (hundreds) amounts of arguments and many function calls. Every test has a version without and with the enforcement of the named form.

#### Existing problems

Even though the prototype worked successfully on tests given, there are still problems, more related to the design of the feature:
1. As the feature currently applies only to arguments, if one wants to mark all the arguments in their function as requiring the named form, they will have to put the new keyword to each argument.
2. Putting the keyword to a variadic argument is not prohibited. This placement will render the function unusable, as variadic arguments cannot be passed in positional form.

### Further work

Even though this document is a complete report on the research of enforced named argument form, there are still directions to move before the final decision and, if positive, implementing the feature into the Kotlin programming language.

The three main directions can be summarized in the following list:

1. Conduct further research on specific design choices, ranging from which report level to use for adaptation, which areas to effect to allow, and which types/modes of keywords to include. Additional research can be done on how these features would work on different backends.
2. Add further improvements to the prototype. The test set can be expanded, additional keywords can be added, and behaviour in more specific cases can be checked. Additional diagnostics can be added, too.
3. To check additional implementation directions, try to develop any of the other two ideas into a proper prototype. Maybe adding a runtime annotation could be useful to check how the feature would work if implemented as an annotation.

### Final results

During the work on this issue, many insights and information were gathered from the issues, discussions, and other documents, and some additional research was concluded, including existing implementations in other languages. Different possible use cases, benefits and drawbacks were discussed, possible ways of implementation were analyzed, and different peculiarities were discovered and recorded. Finally, the prototype was implemented to prove the concept and allow for further experiments and research regarding the feature.

## Additional remarks

Some remarks are not directly related to this document's parts but are still related to the enforced argument named form feature. Those are described in this part.

### On variadic arguments in Swift and Kotlin

In Kotlin, it is possible to use only one variadic argument per function, while in Swift, such limitation was lifted. Why can't we lift it in Kotlin? Probably because of interoperability with Java and the fact that the variadic argument always comes last in Java Bytecode. How does Kotlin deal with it?
Another fact: Scala, another JVM language, supports multiple variadic arguments if they are provided in separate argument lists (another feature of Scala is that arguments can be separated into groups for partial initialization/evaluation).
