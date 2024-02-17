## Chapter One 

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



## Chapter Two :