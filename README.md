# Spring Boot Style Guide

An opinionated guide on developing web applications with Spring Boot.

[![Join the chat at https://gitter.im/helpermethod/spring-boot-style-guide](https://badges.gitter.im/helpermethod/spring-boot-style-guide.svg)](https://gitter.im/helpermethod/spring-boot-style-guide?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Dependency Injection

* Use `constructor injection`. Avoid `field injection`.

> Why? Constructor injection makes dependencies explicit and forces you to provide all mandatory dependencies when creating instances of your component.

```java
// bad
@AutoWired
private PersonRepository personRepositoy;

// good
private final PersonRepository personRepository;

public PersonService(PersonRepository personRepository) {
    this.personRepository = personRepository;
}
```

* Avoid single implementation interfaces.

> Why? TBD

```java
// bad
public interface PersonService {
    Person createPerson(String firstname, String lastname);
}

public class PersonServiceImpl implements PersonService {
    public Person create(String firstname, String lastname) {
        // more code
    }
}

// good
public class PersonService {
    public Person create(String firstname, String lastname) {
        // more code
    }
}
```

## Controllers

* Use `@RestController` when providing a RESTful API.

```java
// bad
@Controller
public class PersonController {
    @ResponseBody
    @GetMapping("/person/{id}")
    public Person show(@PathVariable long id) {
        // more code
    }
}

// good
@RestController
public class PersonController {
    @GetMapping("/person/{id}")
    public Person show(@PathVariable long id) {
        // more code
    }
}
```

* Use `@GetMapping`, `@PostMapping` etc. instead of `@RequestMapping`.
 
```java
// bad
@RequestMapping(method = RequestMethod.GET, value = "/person/{id}")
public Person show(@PathVariable long id) {
    // more code
}

// good
@GetMapping("/person/{id}")
public Person show(@PathVariable long id) {
    // more code
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

    // this requires at least Spring Boot 2.0 to work out of the box
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
