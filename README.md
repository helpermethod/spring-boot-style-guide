# Spring Boot Style Guide

A mostly reasonable approach to Spring Boot.

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

> Why? They are easier to read and force you to declare the request method.
 
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

## Serialization and Deserialization

```java
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
