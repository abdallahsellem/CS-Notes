# **Functional Interfaces**

A **functional interface** = interface with **exactly one abstract method**.

Examples:

- `Runnable` → `run()`
    
- `Callable<T>` → `call()`
    
- `Comparator<T>` → `compare(T a, T b)`
    
- `Function<T,R>`
    
- `Supplier<T>`
    
- `Consumer<T>`
    
- `Predicate<T>`
    

You must annotate with:

`@FunctionalInterface`

### ✔ Example:

```java 
@FunctionalInterface interface MathOp {   
  int operate(int a, int b); } 
   MathOp add = (a, b) -> a + b
```


