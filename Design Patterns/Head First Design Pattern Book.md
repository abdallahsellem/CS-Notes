## Chapter One (strategy)

### design pattern rules :

1) identify objects and components of your system that vary and separate them form what stays the same 
2) Program to Interface not to implementation 
3) Favor composition over inheritance 
	- creating systems using composition gives you a lot more flexibility. Not only does it let you encapsulate a family of algorithms into their own set of classes, but it also lets you change behavior at runtime as long as the object you’re composing with implements the correct behavior interface.
4) **The Strategy Pattern** defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.


### design pattern use cases :

- inheritance doesn't work well for system that has classes that have a changing behavior and not all classes has the same behavior so you will be face 2 options : 
	- Just add the function that change in the super class and override function in all the child that you want the behavior change in it and if the class doesn't have that behavior you will do some this `class dog { public void hunting(){//do nothing }}`
	- don't add the function in the super class but implement it in each class (No code reuse) also (if you want to change a little behavior in all classes you will require to change it in all classes so if the 43 child class you have you to change the behavior in each one of 43)

- applying polymorphism to be more abstract and concise: 
	- by using this we don't know which animal i have , all i care is the functionality of the method i want like makesound() in below example :
	- ![[PolyMorePhism.png]]
- **first rule of design pattern identify aspects of your system that vary or change and separate them from components that stay the same of constant overtime** 
- to solve the problem of reusability and flexibility of changing code we can do that : 
	- make an interface for each behavior 
	- create a behavior class which implement the interface 
	- if a subclass of the superclass wants to use this behavior it will make this behavior through an object variable  that implemented in the subclass , so now we don't care about the implementation of the behavior inside our class , all we care about is how to perform the behavior 
	- now lets see the code of what we explained : ![[BehaviorChangeImplementation.png]]
	
	- initialization of the behaviors : ![[BehaviorChangeintilization.png]]
	- also we can use setter and getter to change the behavior of the class on runtime 
- except Java interfaces typically have no implementation code, so no code reuse. In either case, whenever you need to modify a behavior, you’re often forced to track down and change it in all the different subclasses where that behavior is defined

### why Patterns ? 
- Shared pattern vocabularies are POWERFUL. When you communicate with another developer or your team using patterns, you are communicating not just a pattern name but a whole set of qualities, characteristics, and constraints that the pattern represents.
- Patterns allow you to say more with less. When you use a pattern in a description, other developers quickly know precisely the design you have in mind
- Talking at the pattern level allows you to stay “in the design” longer. Talking about software systems using patterns allows you to keep the discussion at the design level, without having to dive down to the nitty-gritty details of implementing objects and classes
- Shared vocabularies can turbo-charge your development team. A team well versed in design patterns can move more quickly with less room for misunderstanding.

> Design patterns don’t go directly into your code, they first go into your BRAIN. Once you’ve loaded your brain with a good working knowledge of patterns, you can then start to apply them to your new designs, and rework your old code when you find it’s degrading into an inflexible mess

- Knowing the OO basics does not make you a good OO designer. Good OO designs are reusable, extensible, and maintainable. Patterns show you how to build systems with good OO design qualities. Patterns are proven object-oriented experience.

- Patterns don’t give you code, they give you general solutions to design problems. You apply them to your specific application.

- Patterns aren’t invented, they are discovered.

- Most patterns and principles address issues of change in software.

- Most patterns allow some part of a system to vary independently of all other parts.

- We often try to take what varies in a system and encapsulate it.

- Patterns provide a shared language that can maximize the value of your communication with other developers.
### Links :
understand polymorphism : https://chat.openai.com/share/a88e1e22-394b-48d9-b95d-203e8b998a37



## Chapter Two(observer) :

### definition :
- Publishers + Subscribers = Observer Pattern
- **The Observer Pattern** defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically
- ![[Pasted image 20240220162630.png]]
- 


### Rules:
- When two objects are loosely coupled, they can interact, but they typically have very little knowledge of each other. As we’re going to see, loosely coupled designs often give us a lot of flexibility (more on that in a bit)
	- the only thing the subject knows about an observer is that it implements a certain interface (the Observer interface)
	- We can add new observers at any time
	- We never need to modify the subject to add new types of observers
	- We can reuse subjects or observers independently of each other
	- Changes to either the subject or an observer will not affect the other
- Strive(try to achieve ) for loosely coupled designs between objects that interact 

- in observer pattern it's better to make observers pull data from subject instead of subject push data to observers 
	- ![[Pasted image 20240220144758.png]]
	- ![[Pasted image 20240220144822.png]]

### Observer Design and Implementation  :
- ![[Pasted image 20240220164151.png]]

-  ![[Pasted image 20240220164305.png]]
- ![[Pasted image 20240220164336.png]]
- ![[Pasted image 20240220164402.png]]


### Main Notes :
- The Observer Pattern defines a one-to-m any relations hip between objects .
- Subjects update Observers using a com m on interface.
- Observers of any concrete type can participate in the pattern as long as they implement the Observer interface.
- Observers are loosely coupled in that the Subject knows nothing about them , other than that they implement the Observer interface.
- You can pus h or pull data from the Subject when using the pattern (pull is considered more “correct”).

- The Observer Pattern is related to the Publish/Subscribe Pattern, which is for m ore complex situations with multiple Subjects and/or multiple m es s age types .
- The Observer Pattern is a com m only used pattern, and we’ll see it again when we learn about Model-View-Controller.



## Chapter Three (Decorator):

### Decorator Properties and definition:
The Decorator Pattern attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

- Decorators have the same supertype as the objects they decorate.
- The decorator adds its own behavior before and/or after delegating to the object it decorates to do the rest of the job.
- Objects can be decorated at any time, so we can decorate objects dynamically at runtime with as many decorators as we like




### Decorator Design and Implementation :






### Main Notes :
-  applying the Open-Closed Principle EVERYWHERE is wasteful and unnecessary, and can lead to complex, hard-to-understand code.
- inheritance is one form of extension, but not necessarily the best way to achieve flexibility in our designs .
- In our designs we should allow behavior to be extended without the need to modify existing code.
- Composition and delegation can often be us ed to add new behaviors at runtime.
- The Decorator Pattern provides an alternative to sub-classing for extending behavior.
- The Decorator Pattern involves a set of decorator class es that are used to wrap concrete components .
- Decorator class es mirror the type of the components they decorate. (In fact, they are the s am e type as the components they decorate, either through inheritance or interface implementation.)
- Decorators change the behavior of their components by adding new functionality before and/or after (or even in place of) method calls to the component.
- You can wrap a component with any number of decorators .
- Decorators are typically trans parent to the client of the component—that is , unless the client is relying on the component’s concrete type.
- Decorators can res ult in m any s m all objects in our design, and overuse can be complex.