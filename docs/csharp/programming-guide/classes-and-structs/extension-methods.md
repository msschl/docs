---
title: "Extension Methods - C# Programming Guide"
ms.custom: seodec18
ms.date: 07/20/2015
helpviewer_keywords: 
  - "methods [C#], adding to existing types"
  - "extension methods [C#]"
  - "methods [C#], extension"
ms.assetid: 175ce3ff-9bbf-4e64-8421-faeb81a0bb51
---
# Extension Methods (C# Programming Guide)
Extension methods enable you to "add" methods to existing types without creating a new derived type, recompiling, or otherwise modifying the original type. Extension methods are a special kind of static method, but they are called as if they were instance methods on the extended type. For client code written in C#, F# and Visual Basic, there is no apparent difference between calling an extension method and the methods that are actually defined in a type.  
  
 The most common extension methods are the LINQ standard query operators that add query functionality to the existing <xref:System.Collections.IEnumerable?displayProperty=nameWithType> and <xref:System.Collections.Generic.IEnumerable%601?displayProperty=nameWithType> types. To use the standard query operators, first bring them into scope with a `using System.Linq` directive. Then any type that implements <xref:System.Collections.Generic.IEnumerable%601> appears to have instance methods such as <xref:System.Linq.Enumerable.GroupBy%2A>, <xref:System.Linq.Enumerable.OrderBy%2A>, <xref:System.Linq.Enumerable.Average%2A>, and so on. You can see these additional methods in IntelliSense statement completion when you type "dot" after an instance of an <xref:System.Collections.Generic.IEnumerable%601> type such as <xref:System.Collections.Generic.List%601> or <xref:System.Array>.  
  
 The following example shows how to call the standard query operator `OrderBy` method on an array of integers. The expression in parentheses is a lambda expression. Many standard query operators take lambda expressions as parameters, but this is not a requirement for extension methods. For more information, see [Lambda Expressions](../statements-expressions-operators/lambda-expressions.md).  
  
 [!code-csharp[csProgGuideExtensionMethods#3](~/samples/snippets/csharp/VS_Snippets_VBCSharp/csProgGuideExtensionMethods/cs/extensionmethods.cs#3)]  
  
 Extension methods are defined as static methods but are called by using instance method syntax. Their first parameter specifies which type the method operates on, and the parameter is preceded by the [this](../../language-reference/keywords/this.md) modifier. Extension methods are only in scope when you explicitly import the namespace into your source code with a `using` directive.  
  
 The following example shows an extension method defined for the <xref:System.String?displayProperty=nameWithType> class. Note that it is defined inside a non-nested, non-generic static class:  
  
 [!code-csharp[csProgGuideExtensionMethods#4](~/samples/snippets/csharp/VS_Snippets_VBCSharp/csProgGuideExtensionMethods/cs/extensionmethods.cs#4)]  
  
 The `WordCount` extension method can be brought into scope with this `using` directive:  
  
```csharp  
using ExtensionMethods;  
```  
  
 And it can be called from an application by using this syntax:  
  
```csharp  
string s = "Hello Extension Methods";  
int i = s.WordCount();  
```  
  
 In your code you invoke the extension method with instance method syntax. However, the intermediate language (IL) generated by the compiler translates your code into a call on the static method. Therefore, the principle of encapsulation is not really being violated. In fact, extension methods cannot access private variables in the type they are extending.  
  
 For more information, see [How to implement and call a custom  extension method](./how-to-implement-and-call-a-custom-extension-method.md).
  
 In general, you will probably be calling extension methods far more often than implementing your own. Because extension methods are called by using instance method syntax, no special knowledge is required to use them from client code. To enable extension methods for a particular type, just add a `using` directive for the namespace in which the methods are defined. For example, to use the standard query operators, add this `using` directive to your code:  
  
```csharp  
using System.Linq;  
```  
  
 (You may also have to add a reference to System.Core.dll.) You will notice that the standard query operators now appear in IntelliSense as additional methods available for most <xref:System.Collections.Generic.IEnumerable%601> types.  
  
## Binding Extension Methods at Compile Time  
 You can use extension methods to extend a class or interface, but not to override them. An extension method with the same name and signature as an interface or class method will never be called. At compile time, extension methods always have lower priority than instance methods defined in the type itself. In other words, if a type has a method named `Process(int i)`, and you have an extension method with the same signature, the compiler will always bind to the instance method. When the compiler encounters a method invocation, it first looks for a match in the type's instance methods. If no match is found, it will search for any extension methods that are defined for the type, and bind to the first extension method that it finds. The following example demonstrates how the compiler determines which extension method or instance method to bind to.  
  
## Example  
 The following example demonstrates the rules that the C# compiler follows in determining whether to bind a method call to an instance method on the type, or to an extension method. The static class `Extensions` contains extension methods defined for any type that implements `IMyInterface`. Classes `A`, `B`, and `C` all implement the interface.  
  
 The `MethodB` extension method is never called because its name and signature exactly match methods already implemented by the classes.  
  
 When the compiler cannot find an instance method with a matching signature, it will bind to a matching extension method if one exists.  
  
 [!code-csharp[csProgGuideExtensionMethods#5](~/samples/snippets/csharp/VS_Snippets_VBCSharp/csProgGuideExtensionMethods/cs/extensionmethods.cs#5)]  
  
## General Guidelines  
 In general, we recommend that you implement extension methods sparingly and only when you have to. Whenever possible, client code that must extend an existing type should do so by creating a new type derived from the existing type. For more information, see [Inheritance](./inheritance.md).  
  
 When using an extension method to extend a type whose source code you cannot change, you run the risk that a change in the implementation of the type will cause your extension method to break.  
  
 If you do implement extension methods for a given type, remember the following points:  
  
- An extension method will never be called if it has the same signature as a method defined in the type.  
  
- Extension methods are brought into scope at the namespace level. For example, if you have multiple static classes that contain extension methods in a single namespace named `Extensions`, they will all be brought into scope by the `using Extensions;` directive.  
  
 For a class library that you implemented, you shouldn't use extension methods to avoid incrementing the version number of an assembly. If you want to add significant functionality to a library for which you own the source code, you should follow the standard .NET Framework guidelines for assembly versioning. For more information, see [Assembly Versioning](../../../standard/assembly/versioning.md).  
  
## See also

- [C# Programming Guide](../index.md)
- [Parallel Programming Samples (these include many example extension methods)](https://code.msdn.microsoft.com/Samples-for-Parallel-b4b76364)
- [Lambda Expressions](../statements-expressions-operators/lambda-expressions.md)
- [Standard Query Operators Overview](../concepts/linq/standard-query-operators-overview.md)
- [Conversion rules for Instance parameters and their impact](https://blogs.msdn.microsoft.com/sreekarc/2007/10/11/conversion-rules-for-instance-parameters-and-their-impact)
- [Extension methods Interoperability between languages](https://blogs.msdn.microsoft.com/sreekarc/2007/10/11/extension-methods-interoperability-between-languages)
- [Extension methods and Curried Delegates](https://blogs.msdn.microsoft.com/sreekarc/2007/05/01/extension-methods-and-curried-delegates)
- [Extension method Binding and Error reporting](https://blogs.msdn.microsoft.com/sreekarc/2007/04/26/extension-method-binding-and-error-reporting)
