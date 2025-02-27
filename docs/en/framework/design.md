# Framework — Design Approach

The Spiral Framework is a PHP framework that aims to simplify the development of complex web applications. One of the
key principles of the framework is to adhere to the principle of pragmatic design, which includes following the
principles of Keep It Simple, Stupid (KISS) and SOLID.

## Principles

The KISS principle states that software should be designed to be as simple as possible, with a minimal number of moving
parts. This makes the code easier to understand, test, and maintain. The SOLID principles, on the other hand, are a set
of five principles of object-oriented design that are intended to make software more maintainable and scalable.

### 1. Avoid cross dependencies when possible

One of the core components of the Spiral Framework is the use of dependency injection, which helps to avoid
cross-dependencies between components and promotes loose coupling. This makes it easy to replace one component with
another, without affecting the rest of the system.

### 2. Prioritize composition over inheritance

Another important aspect of the Spiral Framework is its focus on composition over inheritance. This approach emphasizes
the use of object composition to create new objects, rather than relying on class inheritance. This allows for greater
flexibility and makes it easier to change the behavior of an object at runtime.

### 3. Prefer smaller but richer interfaces

The Spiral Framework also prioritizes smaller, richer interfaces. This means that interfaces should be kept as simple as
possible, with a minimal number of methods. This makes it easier to implement the interface and increases the chances
that a class implementing the interface has the correct behavior.

### 4. Avoid magic

The Spiral Framework also encourages developers to avoid "magic" in their code. This means that the code should be as
explicit as possible and avoid using hidden side effects or other tricky constructs. The aim is to make the code as easy
to understand as possible.

### 5. Java code style

Finally, the Spiral Framework follows the Java code style which is considered as a high-quality standard. This means
that the code should be consistently formatted and follow established conventions and best practices.

## Hybrid Runtime

The framework relies on the [RoadRunner](https://roadrunner.dev) application server to run some of its services. PHP
codebase is mostly centered around quick delivery of efficient business logic. The application server, Golang based, is
focused on efficiently solving infrastructure tasks.

![High Level Architecture Diagram](https://user-images.githubusercontent.com/773481/180764832-d91daec4-36fb-4651-ace3-64eac6f289c8.png)

> **Note**
> Read about the application lifecycle [here](../framework/lifecycle.md).
