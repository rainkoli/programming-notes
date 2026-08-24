# Dependency Injection Detailed Notes

## Dependency

A dependency is an object that another object needs in order to perform
its responsibilities.

Example:

``` java
class UserService {

    private UserRepository repository;

    public void register(User user) {
        repository.save(user);
    }
}
```

Here:

-   `UserService` handles user business logic.
-   `UserRepository` handles data access.

`UserService` needs `UserRepository` to complete `register()`, so:

    UserService depends on UserRepository

Therefore:

    UserRepository is a dependency of UserService.

------------------------------------------------------------------------

## Dependency Direction

Dependency has direction.

Correct:

    UserService
          |
          ↓
    UserRepository

`UserService` uses the capability provided by `UserRepository`.

------------------------------------------------------------------------

## Dependency Injection

Dependency Injection means:

> An object receives the objects it depends on from an external source
> instead of creating them internally.

Without DI:

``` java
class UserService {

    private UserRepository repository =
            new UserRepository();

}
```

`UserService` creates its own dependency.

With DI:

``` java
class UserService {

    private UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }

}
```

The dependency is provided externally.

------------------------------------------------------------------------

## Constructor Injection

Constructor Injection provides dependencies through a constructor.

Example:

``` java
class UserService {

    private UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

The caller provides the dependency:

``` java
UserRepository repository =
        new UserRepository();

UserService service =
        new UserService(repository);
```

------------------------------------------------------------------------

## Method Injection

Method Injection provides dependencies through method parameters.

Example:

``` java
class UserService {

    public void process(UserRepository repository) {
        repository.save();
    }

}
```

The dependency only exists during method execution.

------------------------------------------------------------------------

## Dependency Injection Roles

Three important roles:

### Client

The object that needs a dependency.

Example:

    UserService

### Dependency

The object that provides required functionality.

Example:

    UserRepository

### Injector

The object responsible for creating and supplying dependencies.

Examples:

-   Spring: ApplicationContext / BeanFactory
-   JUnit 5: ParameterResolver

------------------------------------------------------------------------

## Dependency Injection and IoC

Dependency Injection is often used together with Inversion of Control.

Traditional:

    Application
        |
        ↓
    new Object()

IoC:

    Framework
        |
        ↓
    creates object

Spring manages object creation instead of application code.

------------------------------------------------------------------------

## JUnit 5 Dependency Injection

JUnit 5 supports constructor injection and method injection.

Method injection example:

``` java
@Test
void test(TestInfo info) {

}
```

JUnit provides `TestInfo`.

Flow:

    @Test
     ↓
    Find parameter
     ↓
    ParameterResolver
     ↓
    Create object
     ↓
    Invoke test method

Constructor injection example:

``` java
class UserTest {

    UserTest(TestInfo info) {

    }
}
```

JUnit creates the test object and supplies the constructor parameter.

------------------------------------------------------------------------

## JUnit DI vs Spring DI

                 JUnit 5                         Spring
  -------------- ------------------------------- ----------------------------
  Container      JUnit Engine                    ApplicationContext
  Injector       ParameterResolver               BeanFactory
  Dependencies   TestInfo, TestReporter          Beans
  Injection      Constructor/Method parameters   Constructor, Field, Setter

------------------------------------------------------------------------

## Summary

Dependency means:

> An object that another object needs to complete its responsibility.

Example:

    UserService
          |
          ↓
    UserRepository

Dependency Injection means:

> Dependencies are created and supplied externally instead of being
> created by the object itself.

Main forms:

1.  Constructor Injection
2.  Method Injection
3.  Setter Injection
