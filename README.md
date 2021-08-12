Welcome to Refactoring from Anemic Domain Model Towards a Rich One
=====================

This is the source code for my Pluralsight course [Refactoring from Anemic Domain Model Towards a Rich One][L5].

How to Get Started
--------------

There are 2 versions of the source code: Before and After. You can go ahead and look at the After version but I would recommend that you follow the course and do all refactoring steps along with me.

In order to run the application, you need to create a database and change the connection string in the composition root.

[L2]: DBCreationScript.txt
[L3]: DddInPractice.UI/App.xaml.cs
[L1]: http://www.apache.org/licenses/LICENSE-2.0
[L4]: https://www.pluralsight.com/courses/domain-driven-design-in-practice
[L5]: http://www.pluralsight.com/courses/refactoring-anemic-domain-model

Divan's notes

(4.) Decoupling the Domain Model from the Data Contracts

Why?
===

Changes don't break contracts for clients.
Improve security of contracts: Clients can't change fields that they shouldnt.

How?
===

Create Dtos
Seperate the data contracts and the domain models.
Different contracts for lists, inputs and outputs.

(5.) Using Value Objects as Domain Model Building blocks

This makes the model more expressive and encapsulated.
Identify problems faster(during compilation)
More expressive, readable domain model. Makes concepts explicit.
Avoid domain logic duplication. Adhere to DRY principle.
Raise level of abstraction

Use the Value Objects as the buildign blocks of the domain model
Fully utilise the type system

Note: You can use the NuGet package CSharpFunctionalExtentions to get the generic Result or ValueObject classes. Or implement your own.

(6.) Pushing Logic down from Services to Domian Classes

Starts off with logic in entities and public setters => Refactor this.
Put domain logic close the the data it operates on in oerder to encapsulate. Transfer it from the services, to the domain entities.
Entitites have exposed methods, or "public APIs".
Define constructors with exception handling. You can place the check in the same line as the assignment. => This will be refactored later though, to adhere to seperation of concerns.
Important: Remove operations from the APIs that are illegal from the domain standpoint.
Encapsulate over-exposed collections, by introducing a read-only public collection, on top of a private mutable one. Create seperate methods for mutation. Only add them as you need them (YANGNI)
Remember: When you perform two actions to achive one goal, it's a sign of a leak in abstraction.
Essence of transforming Anemic Domain Model to a Rich one: Reduce the number of mutatuing APIs. Only leave ones that are absolutely neccecary.

When we move the logic from the services to the domain entities:  1. We can delete the services
                                                                  2. No longer have to pass services to the controller. We can use the entities directly.
Note: We cannot neccecarilly do this in all projects.
For eg. It is usefull to have a service if it points to an external resource like a 3rd party API. It is important to keep your domain model isolated from the external world.
Therefore, you should not attribute such responsibilities to domain classes. 
Another eg. Logic that does not fit any of the domain classes. This might be a sign of a missed abstraction.

In a entity, if the methods work with the same data and have similar logic, we can abstract that code.

After these refactorings, domain classes are encapsulated and extensible, no domain class can now reside in an invalid state.
Mive all operations upon domain classes to those classes themselves (richness)
Use constructors to ensure all new instances are valid.
Encapsulate the work with collections. Don't expose immutable collections. Introduce seperate methods to add/remove items from the collection.
Introduce new abstractions, this makes the code more readible.
Pushed domain logic from services to entities. This allowed us to remove mutating APIs.
Refactor switch statements, to a hierarchy of classes. Allowing OO polymorphism to do all the work for us.

(7.) Organising the Application Services Layer

Application services layer

(diagram) (Application services layer is in the controller)
All domain knowledge should be attributed to the domain classes
Can create seperate classes for application layer and not have that in the controller. This will help prevent the controllers to become bloated.
An application services layer does not contain any domain logic

Repositories and Unit of Work

It is not the repository's responsibility to commit the unit of work
Commit unit of work, when the controller's method returns 'OK' => Implement unit of work in a BaseController
Implement own Ok() method in the new BaseController. Now we can remove the SaveChanges() calls from the controller.
The result of this, is that the 'unit of work' is only committed when successful.
Repositories now, do not control the unit of work. All work is commited or discarded at the end of the web request. => To do that, we created our own base controller class.

Working with exceptions

Methods in controller have the same overarching try, catch statement. This violates DRY. We also don't catch all possible exceptions in the controller (exceptions can happen anywhere, not only in repository actions)
Create single exception handler to cover all controllers in the project. Implement as middleware in the application's pipeline. Hence, we must fullfill some contract there. (Middleware for all unhandled exceptions)
In order to use the new handler, we need to include it in the ASP.NET pipeline. We do this by adding the app.UseMiddleWare<ExceptionHandler>(); method call in the Configure method in Startup.cs => Now, we no longer need the try/catch statements!

Introducing: Envelope

Create a wrapper, that standardises the format of the response of the API. This wrapper will have a structure that eceryone can bing to. ("Envelope")
Now, return the envelope object, instead of the result, in the base controller and ExceptionHandler.

Simplifying the Controller

It's simpler to use domain classes, than IDs for checks (This utilises the inheritance from the Entity base class)
Move domain knowledge out of controller, to the underlying domain class.
Remove more responsibility from the client code by checking if a movie is already purchased(domain knowledge), in the Customer class(domain class) => This makes it impossible for the customer entity to enter an illegal state, with regards to the current movies.
Also, to be consistant, we should have all domain checks in the same place (domain class). Not some in the controller and others in the domain class.
In the Customer class (domain class): Return a Result class instead of a bool, as this is more descriptive.
Good to seperate the check and the actual operation in seperate methods in the domain class, as is done here. 
That way you adhere to the Command-query seperation principle(CQS) => any method either returns a value, or mutates the state. Never both. This is not always possible to adhere to. For instance, when the operation must occur immediately after the check.

Changing the Project Structure

You must have high cohesion of your code, in all aspects. Group files together, based on their business meaning (not implementation details)
Create folders for domain files.
Create Utils folder, for files unrelated to the domain model
Create Common folder, for base domain classes as well as value objects that are used by multiple aggrigates

(8.) Domain Model best practices

YAGNI

Business requirements change constantly.
Less is better. Code is not an asset, it's a liability. More code increases cost of ownership. This is the best predictor of project success.
Build a domain model specific to your problem as possible. Don't over-abstract the domain model.
Don't use a boilerplate set of domain classes. 
Don't bring complexity untill you really need it.

Sharing Domain Logic Between Projects

The DRY principle is for a single bounded context only.
Different bounded contexts have different perspectives on the same concepts. There is always knowledge that is important for one project, but not relevant to another.
Dycotomy: Coupling <==> Code Duplication. Remember: Coupling is MUCH worse than code duplication. It will hinder your ability to evolve your domain model.
Sharing utility code is usually OK.
Bounded contexts should always evolve indapendantly

Domain model Encapsulation

Expose the minimum mutable APIs as possible.
Don't allow the model to reside in an invalid state. Have almost no public setters, and less primitive types.

Domain Model Isolation

Serate Domain logic from external influence (eg. repo)
For eg. Domain model should not directly interact with the external influence (eg. BD) as this does not scale.
Attribute this class to the app services layer. This reduces complexity, increases testability.

