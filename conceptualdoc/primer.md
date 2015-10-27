.NET Primer
===========

By Rich Lander, Zlatko Knezevic

.NET is a general purpose development platform. It can be used for any
kind of app type or workload where general purpose solutions are used.
It has several key features that are attractive to many developers,
including automatic memory management and modern programming languages,
that make it easier to efficiently build high-quality apps. .NET enables
a high-level programming environment with many convenience features,
while providing low-level access to native memory and APIs.

Multiple implementations of .NET are available, based on open [.NET
Standards](https://github.com/dotnet/coreclr/blob/master/Documentation/dotnet-standards.md)
that specify the fundamentals of the platform. They are separately
optimized for different app types (e.g. desktop, mobile, gaming, cloud)
and support many chips (e.g. x86/x64, ARM) and operating systems (e.g.
Windows, Linux, iOS, Android, OS X). Open source is also an important
part of the .NET ecosystem, with multiple .NET implementations and many
libraries available under OSI-approved licenses.

You can take a look at the ../getting-started/overview document to
figure out all of the different editions of .NET Framework that are
available, both Microsoft's and others.

Key .NET Concepts
-----------------

There is a certain number of concepts that are very important to
understand if you are new to the .NET Platform. These concepts are the
cornerstone of the entire platform, and understanding them at the outset
is important for general understanding of how .NET works.

-   [Managed Code](managed-code.md)
-   [Common Type System](common-type-system.md)
-   [The runtime](common-language-runtime.md)
-   [Base Class Library](framework-libraries.md)

A stroll through .NET
---------------------

As any mature and advanced application development framework, .NET has
many powerful features that make the developer's job easier and aim to
make writing code more powerful and expressive. This section will
outline the basics of the most salient features and provide pointers to
more detailed discussions where needed. After finishing this stroll, you
should have enough information to be able to read the samples on our
GitHub repos as well as other code and understand what is going on.

-   Automatic memory management
-   Type safety
-   The managed compiler
-   Delegates and lambdas
-   Generic Types (Generics)
-   LINQ
-   Asynchronous support
-   Dynamic language features
-   Code contracts
-   Native interoperability

### Automatic memory management

Garbage collection is the most well-known of .NET features. Developers
do not need to actively manage memory, although there are affordances to
provide more information to the garbage collector (GC). C\# includes the
`new` keyword to allocate memory in terms of a particular type, and the
`using` keyword to provide scope for the usage of the object. The GC
operates on a lazy approach to memory management, preferring application
throughput to the immediate collection of memory.

The following two lines both allocate memory:

``` csharp
var title = ".NET Primer";
var list = new List<string>;
```

There is no analogous keyword to de-allocate memory, as de-allocation
happens automatically when the garbage collector reclaims the memory
through its scheduled running.

Method variables normally go out of scope once a method completes, at
which point they can be collected. However, you can indicate to the GC
that a particular object is out of scope sooner than method exit using
the `using` statement.

``` csharp
using(FileStream stream = GetFileStream(context))
{
    //operations on the stream
}
```

Once the `using` block completes, the GC will know that the `stream`
object in the example above is free to be collected and its memory
reclaimed.

One of the less obvious but quite far-reaching features that a garbage
collector enables is memory safety. The invariant of memory safety is
very simple: a program is memory safe if it accesses only memory that
has been allocated (and not freed). Dangling pointers are always bugs,
and tracking them down is often quite difficult.

The .NET runtime provides additional services, to complete the promise
of memory safety, not naturally offered by a GC. It ensures that
programs do not index off the end of an array or accessing a phantom
field off the end of an object.

The following example will throw as a result of memory safety.

``` csharp
int[] numbers = new int[42];
int number = numbers[42]; // will throw (indexes are 0-based)
```

### Type Safety

Objects are allocated in terms of types. The only operations allowed for
a given object, and the memory it consumes, are those of its type. A
`Dog` type may have `Jump` and `WagTail` methods, but not likely a
`SumTotal` method. A program can only call the declared methods of a
given type. All other calls will result either in a compile-time error
or a run-time exception (in case of using dynamic features or `object`).

.NET languages can be object-oriented, with hierarchies of base and
derived classes. The .NET runtime will only allow object casts and calls
that align with the object hierarchy. Remember that every type defined
in any .NET language derives from the core `object` type.

``` csharp
Dog dog = Dog.AdoptDog(); // Returns a Dog type
Pet pet = (Pet)dog; // Dog derives from Pet
pet.ActCute();
Car car = (Car)dog; // will throw - no relationship between Car and Dog
object temp = (object)dog; // legal - a Dog is an object
car = (Car)temp; // will throw - the runtime isn't fooled
car.Accelerate() // the dog won't like this, nor will the program get this far
```

Type safety is also used to help enforce encapsulation by guaranteeing
the fidelity of the accessor keywords. Accessor keywords are artifacts
which control access to members of a given type by other code. These are
usually used for various kinds of data within a type that are used to
manage its behavior.

``` csharp
Dog dog = Dog._nextDogToBeAdopted; // will throw - this is a private field
```

Some .NET languages support **type inference**. Type inference means
that the compiler will deduce the type of the expression on the
left-hand side from the expression on the right-hand side. This doesn't
mean that the type safety is broken or avoided. The resulting type
**has** a strong type with everything that implies. Let's rewrite the
first two lines of the previous example to introduce type inference. You
will note that the rest of the example is completely the same.

``` csharp
var dog = Dog.AdoptDog();
var pet = (Pet)dog;
pet.ActCute();
Car car = (Car)dog; // will throw - no relationship between Car and Dog
object temp = (object)dog; // legal - a Dog is an object
car = (Car)temp; // will throw - the runtime isn't fooled
car.Accelerate() // the dog won't like this, nor will the program get this far
```

### Delegates and Lambdas

Delegates are like C++ function pointers, with a big difference that
they are type safe. They are a kind of disconnected method within the
CLR type system. Regular methods are attached to a class and only
directly callable through static or instance calling conventions.

Delegates are used in various APIs and places in the .NET world,
especially through lambda expressions, which are a cornerstone of Linq.

Read more about it in the delegates-lambdas document.

### Generic Types (Generics)

Generic types, a.k.a "generics" are a feature that was added in .NET
Framework 2.0. In short, generics allow the programmer to introduce a
"type parameter" when designing their classes, that will allow the
client code (i.e. the users of the type) to specify the exact type to
use in place of the type parameter.

Generics were added in order to help programmers implement generic data
structures. Before their arrival, in order for a, say, List type to be
generic, it would have to work with elements that were of type object.
This would have various performance as well as semantic problems, not to
mention possible subtle runtime errors. The most notorious of the latter
is when a data structure contains, for instance, both integers and
strings, and an InvalidCastException is thrown on working with the
list's members.

The below sample shows a basic program running using an instance of
List&lt;T&gt; types.

``` csharp
using System;
using System.Collections.Generic;

namespace GenericsSampleShort {
    public static void Main(string[] args){
        // List<string> is the client way of specifying the actual type for the type parameter T
        List<string> listOfStrings = new List<string> { "First", "Second", "Third" };

        // listOfStrings can accept only strings, both on read and write.
        listOfStrings.Add("Fourth");

        // Below will throw a compile-time error, since the type parameter
        // specifies this list as containing only strings.
        listOfStrings.Add(1);

    }
}
```

Read more about it in the generics document.

### Async Programming

Async is a first-class concept within .NET, with async support in the
runtime, the framework libraries and various .NET languages. Async is
based off of the `Task` concept, which encapsulates a set of operations
to be completed. Tasks are distinct from threads and may not rely on
threads or require CPU time much at all, particularly for I/O-bound
tasks.

TODO: Elaborate on Task concept.

C\# includes special treatment for async, including the special keyword
`await` for managing tasks. The following example demonstrates calling a
web endpoint as an async operation.

``` csharp
    string url = "http://someUrl";
    HttpClient client = new HttpClient();
    string json = await client.GetStringAsync(url);
```

The call to `client.GetStringAsync(url)` does not block, but instead
immediately yields by returning a `Task`. Computation resumes and the
call returns the requested string when the network activity has
completed.

### Language Integrated Query (LINQ)

.NET programs typically operate on some form of data. The data can be
database-resident or in the form of objects (sometimes called POCOs for
"Plain Old CLR Objects"). LINQ provides a language-integrated uniform
query model over data, independent of the source. Linq providers bridge
the gap between the uniform query model and the form of the data, such
as SQL Server tables, XML documents, standard collections like List and
more.

The follow examples demonstrate various uses of LINQ to query different
forms of data.

TODO: finish the section, link to a more detailed document.

### Dynamic language features

TODO: finish section

### Code contracts

TODO: finish section

### Native Interoperability

.NET provides low-level access to native APIs via the platform invoke or
P/Invoke facility. It enables a mapping of .NET types to native types,
which the .NET runtime marshalls before calling the native API.

TODO: Examples.

Higher-level native interop can be established with P/Invoke. The COM
and WinRT interop systems in the CLR are both built on top of P/Invoke.
The Java and Objective-C interop systems provided by Xamarin on top of
Mono are fundamentally the same.

#### Unsafe Code

The CLR enables the ability to acccess native memory and do pointer
arithmetic. These operations are needed for some algortithms and for
calling some native APIs. The use of these capabilities is discouraged,
since you no longer get the benefit of verifiability, nor will your code
be allowed to run in all environments. The best practice is to confine
unsafe code as much as possible and that the vast majority of code is
type-safe.

TODO: Examples.

Notes
-----

The term ".NET runtime" is used throughout the document to accomodate
for the multiple implementations of .NET, such as CLR, Mono, IL2CPP and
others. The more specific names are only used if needed.

This document is not intended to be historical in nature, but describe
the .NET platform as it is now. It isn't important whether a .NET
feature has always been available or was only recently introduced, only
that it is important enough to highlight and discuss.
