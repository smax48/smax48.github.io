---
layout: post
title: My favourite trick with .NET generics
---

Sometimes you need to process information about child classes in the parent class on the static level.
For example, invoke some reflection methods to find out metadata about the derived type in the static constructor of the base type.

Although it sounds a bit weird, this can be achieved quite easily with generics.
Despite the fact that this is actually not the intended use of generics, it works very well.

All you need to do is to declare your base class in the following form
(field "SomeIntProp" here is just an example, see its possible usage below):

```
class MyBaseClass<T> where T : MyBaseClass<T> {
  static int SomeIntProp;
...
}
```

And now the derived classes:

```
class MyChildClassA : MyBaseClass<MyChildClassA> {
 public string StringProp1 {get; set; }
...
}
class MyChildClassB : MyBaseClass<MyChildClassB> {
 public string StringProp2 {get; set; }
...
}
```

So basically you tell the compiler that your generic parameter is a class that must derive from this base class.

This approach gives you the possibility to query metadata about the derived class:

```
typeof(T).GetProperties(); //will return {StringProp1} or {StringProp2} as a result depending on what type is <T>
```

Additional benefit: every generic specialization will have its own scope for static variables defined in the base class.

In the example above, the "SomeIntProp" field can hold different values for MyChildClassA and MyChildClassB!
