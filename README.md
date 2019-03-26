# Spring Boot Style Guide

An opinionated guide on developing web applications with Spring Boot. Inspired by [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript).

[![Join the chat at https://gitter.im/helpermethod/spring-boot-style-guide](https://badges.gitter.im/helpermethod/spring-boot-style-guide.svg)](https://gitter.im/helpermethod/spring-boot-style-guide?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/helpermethod/spring-boot-style-guide/master/LICENSE)

## Table of Contents

* [Dependency Injection](#dependency-injection)
* [Controllers](#controllers)
* [Serialization](#serialization)
* [Testing](#testing)

## Dependency Injection

* Use `constructor injection`. Avoid `field injection`.

> Why? Constructor injection makes dependencies explicit and forces you to provide all mandatory dependencies when creating instances of your component.

```java
// bad
public class PersonService {
    @AutoWired
    private PersonRepository personRepositoy;
}

// good
public class PersonService {
    private final PersonRepository personRepository;

    public PersonService(PersonRepository personRepository) {
        this.personRepository = personRepository;
    }
}    
```

* Avoid single implementation interfaces.

> Why? A class already exposes an interface: its public members. Adding an identical `interface` definition makes the code harder to navigate and violates [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it).
>
> What about testing? Earlier mocking frameworks were only capable of mocking interfaces. Recent frameworks like [Mockito](https://site.mockito.org/) can also mock classes. 

```java
// bad
public interface PersonService {
    Person createPerson(String firstname, String lastname);
}

public class PersonServiceImpl implements PersonService {
    public Person createPerson(String firstname, String lastname) {
        // more code
    }
}

// good
public class PersonService {
    public Person createPerson(String firstname, String lastname) {
        // more code
    }
}
```

**[⬆ back to top](#table-of-contents)**

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
@RestController
public class PersonController {
    @RequestMapping(method = RequestMethod.GET, value = "/person/{id}")
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

**[⬆ back to top](#table-of-contents)**

## Serialization

* Do not map your JSON objects to `JavaBeans`.

> Why? JavaBeans are mutable and split object construction across multiple calls.

```java
// bad
public class Person {
    private String firstname;
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

    // requires at least Spring Boot 2.0 to work out of the box
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

// best
public class Person {
    private final String firstname;
    private final String lastname;

    // if the class has a only one visible constructor, the @JsonCreator annotation can be ommitted
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

**[⬆ back to top](#table-of-contents)**

## Testing

* Keep Spring out of your unit tests.

```java
class PersonServiceTests {
    @Test
    void testfindPersonById() {
        // given
        PersonRepository personRepository = mock(PersonRepository.class);
        when(personRepository.findById(1L)).thenReturn(Optional.of(new Person("Oliver", "Weiler")));

        PersonService personService = new PersonService(personRepository);

        // when
        Person person = personService.findPersonById(1L);

        // then
        assertThat(person).extracting("firstname", "lastname").containsExactly("Oliver", "Weiler");
    }
}
```

* Use [AssertJ](http://joel-costigliola.github.io/assertj/). Avoid [Hamcrest](http://hamcrest.org/).

**[⬆ back to top](#table-of-contents)**
