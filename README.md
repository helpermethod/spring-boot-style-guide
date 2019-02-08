# Spring Boot Style Guide

*A mostly reasonable approach to Spring Boot.*

[![Join the chat at https://gitter.im/helpermethod/spring-boot-style-guide](https://badges.gitter.im/helpermethod/spring-boot-style-guide.svg)](https://gitter.im/helpermethod/spring-boot-style-guide?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Dependency Injection

* Never use `field injection` or `setter injection`. Use `constructor injection` instead.

> Why? See [Why field injection is evil](
http://olivergierke.de/2013/11/why-field-injection-is-evil/) for an in-depth discussion.

```java
// really bad
@AutoWired
private PersonRepository personRepositoy;

// still bad
private PersonRepository personRespository;

@Autowired
public void setPersonRepository(PersonRepository personRepository) {
    this.personRepository = personRepository;
}

// good
private final PersonRepository personRepository;

public PersonService(PersonRepository personRepository) {
    this.personRepository = personRepository;
}
```

## Controllers

* Use `@GetMapping`, `@PostMapping` etc. instead of `@RequestMapping`.

> Why? `@GetMapping`, `@PostMapping` etc. are easier to read and force you to declare the request method.
 
```java
// bad
@RequestMapping(method = RequestMethod.GET, value = "/person/{id}")
public Person show(@PathVariable long id) {
}

// good
@GetMapping("/person/{id}")
public Person show(@PathVariable long id) {
}
```

## Serialization

* Do not map your JSON objects to JavaBeans.

> Why? JavaBeans are mutable and split object construction across multiple calls.

```java
// bad
public class Person {
    privat String firstname;
    private String lastname;

    public void setFirstname() {
        this.firstname = firstname;
    }

    public String getFirstname() {
        return firstname;
    }

    public void setLastname() {
        this.lastname = lastname;
    }

    public String getLastname() {
        return lastname;
    }
}

// good
public class Person {
    private final String firstname;
    private final String lastname;

    @JsonCreator
    public Person(String firstname, String lastname) {
        this.firstname = firstname;
        this.lastname = lastname;
    }

    public String getFirstname() {
        return firstname;
    }

    public String getLastname() {
        return lastname;
    }
}
```
