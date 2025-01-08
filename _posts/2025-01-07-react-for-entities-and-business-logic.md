## React for Entities and Business Logic

## Introduction

Object models have been a cornerstone of software development for decades, providing a foundation for writing business logic in various programming languages. In this background information, we will delve into the evolution of object models, focusing on Java, C\#, Python, and TypeScript. We will also explore the backlash against heavy, rigid object models and the emergence of POJO (Plain Old Java Object) and similar concepts.

We borrow some [tree diffing ideas](https://www.dhiwise.com/post/a-deep-dive-into-react-reconciliation-algorithm) from React and apply it to object frameworks.

## The Rise of Object Models

In the 1990s and early 2000s, object models became increasingly popular as a way to encapsulate business logic in software applications. This was largely driven by the adoption of object-oriented programming (OOP) languages like Java, C++, and C\#. Object models provided a structured approach to organizing code, making it easier to maintain, reuse, and extend.

In Java, for example, the Enterprise JavaBeans (EJB) framework introduced a rigid, container-managed approach to building object models. Similarly, in C\#, the .NET Framework's Component Object Model (COM) and later, the .NET Framework's Entity Framework, promoted the use of object models for building business logic.

## The Backlash Against Heavy Object Models

However, as object models became more widespread, developers began to experience the drawbacks of heavy, rigid frameworks. These included:

* Steep learning curves: Complex frameworks require significant investment in training and expertise.
* Tight coupling: Object models were often tightly coupled to specific frameworks, making it difficult to switch or replace them.
* Inflexibility: Rigid frameworks limited the ability to adapt to changing business requirements.
* Performance overhead: Heavy object models introduced significant performance overhead, impacting application responsiveness.

## The Emergence of POJO and Similar Concepts

In response to these challenges, the concept of POJO (Plain Old Java Object) emerged in the early 2000s. POJO emphasized the use of simple, lightweight Java objects to encapsulate business logic, rather than relying on heavy, container-managed frameworks.

Similar concepts, such as POCO (Plain Old CLR Object) in .NET and Python's emphasis on simple, flexible data structures, also gained popularity. These approaches prioritized simplicity, flexibility, and ease of use over the rigid, framework-centric approaches of the past.

## Modern Object Models

Today, object models continue to evolve, incorporating lessons learned from the past. Modern object models prioritize simplicity, flexibility, and ease of use, while still providing a structured approach to organizing business logic.

In Java, for example, the Spring Framework's emphasis on simplicity, flexibility, and ease of use has become a de facto standard for building object models. Similarly, in C\#, the .NET Core Framework's lightweight, flexible approach to building object models has gained widespread adoption.

In Python, the emphasis on simple, flexible data structures continues, with popular frameworks like Django and Flask providing lightweight, modular approaches to building object models. In TypeScript, the growing popularity of frameworks like Angular and React has led to a renewed focus on building simple, flexible object models that prioritize ease of use and maintainability.

## Transaction Management in Modern Object Models

Modern object models provide robust transaction management capabilities, allowing developers to easily manage database transactions. In Java, for example, the Spring Framework provides a comprehensive transaction management system, which allows developers to annotate methods with `@Transactional` to enable transactional behavior.

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
    }
}
```

Similarly, in C\#  Entity Framework provides a `DbContext` class, which manages database transactions. Developers can use the `SaveChanges` method to commit changes to the database.

```cs
public class UserService {

    private readonly MyDbContext _context;

    public UserService(MyDbContext context) {
        _context = context;
    }

    public void CreateUser(User user) {
        _context.Users.Add(user);
        _context.SaveChanges();
    }
}
```

##

## Tracking Modifications to Objects

To generate SQL at commit time, modern object models use various techniques to track modifications to objects. One common approach is to use a concept called "change tracking." In this approach, the framework maintains a record of changes made to objects, which is then used to generate SQL statements at commit time.

For example, in the Entity Framework, the `DbContext` class uses a concept called "change tracking" to track modifications to objects. When an object is modified, the `DbContext` class updates its internal change tracking records, which are then used to generate SQL statements at commit time.

## First Generation Solution: Transaction Script and Unit of Work

As developers, we can take a more pragmatic approach to writing business logic by focusing on plain old objects (POJOs) and leveraging frameworks in a lazy and targeted manner. This approach allows us to decouple our business logic from the underlying framework, making it easier to test, maintain, and evolve our codebase over time. By keeping our business logic in POJOs, we can also avoid the overhead and complexity associated with heavy frameworks, and instead use them only when necessary, such as when interacting with the database or other external systems.

To achieve this, developers can use a technique called "[transaction script](https://martinfowler.com/eaaCatalog/transactionScript.html)" or "[unit of work](https://martinfowler.com/eaaCatalog/unitOfWork.html)," where the business logic is executed in a plain old object, and the framework is used only to manage the transaction and persist the changes to the database. This approach allows developers to write business logic that is decoupled from the framework, and yet still benefits from the framework's capabilities, such as transaction management and database persistence. By using frameworks in a lazy and targeted manner, developers can strike a balance between simplicity, flexibility, and maintainability, and build robust and scalable applications that meet the needs of their business.

While this is an improvement, it doesn’t go all the way. We present a different method below.

## Better Solution: React like Object Group Diffing

In python there is a [100x difference](https://github.com/adsharma/fquery/pull/4) in instantiation cost between a dataclass vs SQLModel. For POJO vs Entity, the difference is likely to be around 20-50x.

The idea is exemplified by this LLM generated code for [Java](https://gist.github.com/adsharma/4fdffa615e732b76ba0e314ad5061b81) and [Python](https://gist.github.com/adsharma/cfe9b151fc3ea0bdc0c31e297ea15a4a). Both transaction script and unit of work run on POJOs.

At the end, we diff and use an Entity framework to generate a transaction with the object modifications. For those who prefer NoSQL solutions, there may be a more direct path to persistence.

## Impact on Core Database Technology

The use of pessimistic concurrency is widespread among distributed databases. This limits how quickly transactions can be committed in a multi-region setup. A new generation of distributed databases that use optimistic concurrency control (OCC) are on the horizon. But they’re limited by the existence of the frameworks which implement very generalized transactions.

By adopting the ideas presented here, we can limit most applications to transactions where all “keys” or object-ids that are modified are known at transaction creation time. There would be no long lived transactions.

## Conclusion

In conclusion, object models have undergone significant evolution over the years, driven by the need for simplicity, flexibility, and ease of use. The backlash against heavy, rigid object models led to the emergence of POJO and similar concepts, which prioritized simplicity and flexibility over complexity and rigidity. Today, modern object models continue to prioritize these values, providing a structured approach to organizing business logic while emphasizing ease of use and maintainability.

The ideas presented here should unlock higher performance in the business logic layer and open the door to a new generation of databases.

