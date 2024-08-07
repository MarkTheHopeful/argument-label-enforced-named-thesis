# Kotlin Features Research: Enforced Named Argument Form and Argument Labels

This repository is dedicated to my (Mark Ipatov) project, which was done as a bachelor thesis, regarding the benefits, use, and implementation details of two features: **Enforced Named Argument Form** and **Argument Labels**.

## Structure of the repository

As these features are interconnected, the [introduction](introduction.md) file presents basic descriptions and general reasons for implementing them.

More specific information regarding the features, discussions and existing prototypes is placed in their corresponding directories:

* Argument Labels: [Directory](ArgumentLabels/)
    * [Main document](ArgumentLabels/main.md)
    * [Prototype using jumper function (sugar)](https://github.com/MarkTheHopeful/kotlin/tree/argument-label-proto)
    * [Prorotype using additional field in structures](https://github.com/MarkTheHopeful/kotlin/tree/argument-label-proto-2)
    * [Data gathered about the interface overrides with different parameter names](argument-names-override.md)
* Enforced Named Argument Form: [Directory](EnforcedNamedForm/)
    * [Main document](EnforcedNamedForm/main.md)
    * [Prototype for the Enforced Named Form feature](https://github.com/MarkTheHopeful/kotlin/tree/enforced-named-proto)

Some additional findings not directly related to either of the features are gathered in the [corresponding document](additional-findings.md).

## Sources
There were many sources for discussions, ideas, and code snippets; they are collected here to preserve them for further discussion.

* [Discussion about internal and external names](https://discuss.kotlinlang.org/t/kotlin-internal-and-external-parameter-name-propose/7906)
* [Another discussion on the internal parameter names](https://discuss.kotlinlang.org/t/internal-function-parameter-name/17634)
* [Issue KT-34895 (Internal and External names) on the Kotlin youtrack, one of the starting points](https://youtrack.jetbrains.com/issue/KT-34895/Internal-and-external-name-for-a-parameter-aka-Argument-Label)
* [Issue KT-59531 (Non-stable parameter names of interface functions), regarding the overrides with different names](https://youtrack.jetbrains.com/issue/KT-59531/Add-a-way-to-make-parameter-names-of-interface-functions-non-stable)
* [Stackoverflow discussion on unnamed function arguments](https://stackoverflow.com/questions/50672203/kotlin-explicitly-unnamed-function-arguments)
* [Issue KT-9872 (disallow calling a method with named argument), to allow overrides with different names without problems](https://youtrack.jetbrains.com/issue/KT-9872/Disallow-calling-a-method-with-named-argument)
* [Issue KT-8112 (provide syntax for nameless parameters), to suppress warning and IDE quickfix](https://youtrack.jetbrains.com/issue/KT-8112/Provide-syntax-for-nameless-parameters)
* [Issue KTIJ-10594 ("Parameter is never used" quickfix), how this quickfix can render the code invalid](https://youtrack.jetbrains.com/issue/KTIJ-10594)
* [Issue KT-14934 (Enforce parameter usage only in named form), one of the starting points](https://youtrack.jetbrains.com/issue/KT-14934/Enforce-parameter-usage-only-in-named-form)
* [Issue KTIJ-1634 (Add inspection for boolean literals passed without named parameters), about an inspection which motivation is close to one for the enforced named form feature](https://youtrack.jetbrains.com/issue/KTIJ-1634)
* [Issue KTIJ-18880 (Consider more sophisticated inspection...), same as the previous one, but not only for Booleans](https://youtrack.jetbrains.com/issue/KTIJ-18880/Consider-more-sophisticated-inspection-for-same-looking-arguments-without-corresponding-parameter-names)
* [Small topic on enforcing of named arguments](https://discuss.kotlinlang.org/t/add-annotation-or-other-mechanism-to-force-named-arguments-for-method/15636)
* [Stackoverflow discussion on vararg Nothing for enforcing named form](https://stackoverflow.com/questions/37394266/how-can-i-force-calls-to-some-constructors-functions-to-use-named-arguments/37394267#37394267)
* [Issue KT-12846 (Allow vararg parameters of type Nothing), directly related to the previous discussion](https://youtrack.jetbrains.com/issue/KT-27282/Allow-vararg-parameters-of-type-Nothing)
* [Swift documentation on functions, argument labels and their enforced named form](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/)
* [Discussion on Swift forums about when to use argument labels and the named form itself](https://forums.swift.org/t/when-to-use-argument-labels-a-new-approach/1289)

