### **Composite Pattern** – (Structural Pattern)

**Purpose**:

- Allows you to compose objects into tree-like structures (i.e., part-whole hierarchies) and work with them as if they were individual objects.

**Key Idea**:

- **Composite** treats both **individual objects** and **compositions of objects** uniformly.

**Structure**:

- **Component**: Defines the interface for all objects in the composition (both composite and leaf objects).
- **Leaf**: Represents the end objects of a composition. Leaf objects do not have children.
- **Composite**: Contains children and implements child-related operations (add, remove, etc.).

**When to Use**:

- When you need to treat individual objects and compositions (groups of objects) the same way.

**Real-World Example**:

- A file system where files are leaf objects, and folders (composite) can contain files or other folders.

**Example UML**:
![[Pasted image 20240925160018.png]]

---
### **Builder Pattern** – (Creational Pattern)

**Purpose**:

- Separates the construction of a complex object from its representation so that the same construction process can create different representations.

**Key Idea**:

- Build complex objects step-by-step. The construction process is flexible, allowing different combinations and optional parts.

**Structure**:

- **Builder**: Interface or abstract class defining steps to build the product.
- **Concrete Builder**: Implements the builder, constructs specific parts of the product.
- **Director** (optional): Controls the order in which the builder's steps are executed.
- **Product**: The final object constructed using the builder.

**When to Use**:

- When creating complex objects with lots of optional parts or configurations (e.g., constructing a house with various rooms, windows, etc.).
- When you want more control over the creation process than constructors provide.

**Real-World Example**:

- Building a house: You may choose different materials, add specific rooms, and customize the final product.

**Example UML**:

![[Pasted image 20240925160510.png]]



---

### **Factory** vs **Builder** :
#### 1. **Factory Pattern:**

**Purpose**: The **Factory pattern** is used when the main goal is to **create objects** without exposing the complex logic or the class name to the client. It abstracts the **object creation** process, so the client doesn’t need to know how the objects are created, just that it gets the correct one.

- **When to Use**: You use the Factory pattern when the client only needs an **instance of an object**, but **doesn’t care about the specific creation details**. The client just asks for a car, and the factory decides which car to give based on the input.

**Example in Real Life**: Think of a **pizza shop**. You place an order for a "Veggie Pizza" or a "Pepperoni Pizza", but you don’t care how it’s made or what ingredients they use exactly — you just care that you get the pizza type you asked for. The pizza shop is the factory that takes your request and gives you the appropriate pizza.

**Main Purpose**: To **simplify object creation** by hiding the instantiation process from the client.

---

#### 2. **Builder Pattern:**

**Purpose**: The **Builder pattern** is used when you need to **construct complex objects** step by step. It gives you the **control** to customize the object as it's being built, allowing different configurations.

- **When to Use**: Use Builder when the **creation of the object involves multiple steps** or when you have a lot of options to set up the object. It’s not just about creating the object but also about controlling how each piece of the object is put together.

**Example in Real Life**: Imagine you’re **building a computer**. You choose the processor, the RAM, the graphics card, and the storage. It’s a step-by-step process, and each step can vary depending on what kind of computer you want to build. The Builder pattern lets you **assemble the parts** in different ways to create different configurations of the computer.

**Main Purpose**: To provide **more control over object construction** for complex objects that need to be built in a series of steps.

---

#### **Key Difference in Purpose**:

1. **Factory**:
    
    - The main focus is on **what kind of object to create**.
    - The client doesn't need to know the details; they just get an object of the desired type.
    - Ideal for **simple objects** where creation logic can be encapsulated and hidden.
2. **Builder**:
    
    - The main focus is on **how to create the object**, step by step.
    - The client can influence the **construction process** and decide which parts to include.
    - Ideal for **complex objects** where each part can vary or have optional configurations.

---

#### **Clearer Analogy**:

- **Factory** is like a **restaurant** where you just order a meal. You don't care how it’s cooked or prepared; you just get your meal based on what you ordered.
    
- **Builder** is like **building your own custom sandwich** at a sandwich shop. You choose the bread, meat, vegetables, and condiments step-by-step. You care about **each step of the process** because you want to build the sandwich to your taste.
### **Adapter Pattern** – (Structural Pattern)

**Purpose**:

- Converts the interface of a class into another interface that the client expects. It allows incompatible classes to work together.

**Key Idea**:

- "Adapts" one interface to another to make existing classes work with new code.

**Structure**:

- **Target**: The interface the client expects.
- **Adapter**: Adapts the interface of an Adaptee to the Target interface.
- **Adaptee**: The existing class with an incompatible interface.

**When to Use**:

- When you want to integrate a class that has an incompatible interface into your system.
- When you want to reuse existing legacy code without modifying it.

**Real-World Example**:

- A power adapter that allows a 3-prong plug to fit into a 2-prong outlet.

**Example UML**:

![[Pasted image 20240925160315.png]]

