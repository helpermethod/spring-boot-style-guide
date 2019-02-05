# Spring Boot Style Guide

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

