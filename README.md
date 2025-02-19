# Delegates in managed C++

This sample demonstrates single- and multi-cast delegates using C++, including declaration, creation and usage, and a discussion on type safety.
<!-- Download Links -->



## Introduction

Delegates are .NET's type safe equivalent to function pointers.
Delegates go further than this though. Instead of a single 
delegate having the ability to point to, and invoke, a single
function, delegates in .NET give you the ability to have a single
delegate point to a list of methods, each which will be called in 
turn.

## Creating a single cast delegate

A single cast delegate is one which points to a single method. To 
create a delegate you must first declare a delegate type that
has the same signature as the methods you wish it to invoke. For
instance, if we wished to have a delegate call a function that took
a `String*` as a parameter and returned an `int`, we might declare it
as

```cpp
__delegate int MyDelegate(String *str);
```

Single cast delegates are implicitely derived from `Delegate`.
To use a delegate to invoke your methods you must create an instance
of the delegate and pass in an object and the method of that object
you wish to call.

For instance, suppose we had a managed class

```cpp
__gc class MyClass 
{
public:
int MethodA(String *str) 
{
    Console::WriteLine(S"MyClass::MethodA - The value of str is: {0}", str);
    return str->Length;
}
}
```

we could declare an object of type MyClass and a delegate to call the
objects methods like

```cpp
MyClass *pMC = new MyClass();
MyDelegate *pDelegate = new MyDelegate(pMC, &MyClass::MethodA);
```

To invoke the object's method using the delegate is as simple as calling

```cpp
pDelegate->Invoke("Invoking MethodA");
```



This would output:

```cpp
MyClass::MethodA - The value of str is: Invoking MethodA
```

## Creating a multi cast delegate

Multi cast delegates allow you to chain together methods so that each one
will be called in turn when the delegate is invoked. To create a multicast
method use the static `Delegate::Combine` method to combine delegates 
into a multicast delegate.

When creating multicast delegates in beta 1, you had to declare your delegates
you wished to combine as multicast delegates using the 
`__delegate(multicast)` directive. This creates a delegate
derived from `MulticastDelegate`. In beta 2 you do not use the 
`(multicast)` specifier, and you can use Delegate::Combine
your single cast delegates into a multicast delegate.

Below is some sample code showing 2 multicast delegates being combined (beta 2)

```cpp
// Declare a delegate
__delegate int MyDelegate(String *str);

// Create a simple managed reference class
__gc class MyClass 
{
public:
    int MethodA(String *str) 
    {
        Console::WriteLine(S"MyClass::MethodA - The value of str is: {0}", str);
        return str->Length;
    }

    int MethodB(String* str) 
    {
        Console::WriteLine(S"MyClass::MethodB - The value of str is: {0}", str);
        return str->Length * 2;
    }
};

...

MyClass *pMC = new MyClass();
MyDelegate *pDelegate1 = new MyDelegate(pMC, &MyClass::MethodA);
MyDelegate *pDelegate2 = new MyDelegate(pMC, &MyClass::MethodB);

MyDelegate *pMultiDelegate = 
static_cast<MyDelegate *>(Delegate::Combine(pDelegate, pDelegate2));
```

When you invoke the multicast delegate *pMultiDelegate* it will first call
`MyClass::MethodA`, then `MyClass::MethodB`. For instance,
calling

```cpp
pMultiDelegate->Invoke("Invoking Multicast delegate");
```

would result in

```cpp
MyClass::MethodA - The value of str is: Invoking multicast delegate
MyClass::MethodB - The value of str is: Invoking multicast delegate
```

## Type Safety

Delegates are inherently type safe. You cannot compile calls to a delegate 
using the wrong parameters, or assign the return value of a delegate to a
type that cannot be implicitly cast from the return type of the method the
delegate is calling. Because an object and method are passed to the delegates
constructor, the compiler has all the information it needs to ensure that
errors caused by mismatched parameters and return types do not occur.
