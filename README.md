# Spring Boot Style Guide

An opinionated guide on developing web applications with Spring Boot. Inspired by [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript).

[![Join the chat at https://gitter.im/helpermethod/spring-boot-style-guide](https://badges.gitter.im/helpermethod/spring-boot-style-guide.svg)](https://gitter.im/helpermethod/spring-boot-style-guide?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/helpermethod/spring-boot-style-guide/master/LICENSE)

## Table of Contents

* [Dependency Injection](#dependency-injection)
* [Controllers](#controllers)
* [Serialization](#serialization)
* [Testing](#testing)
* [Monitoring](#monitoring)

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

    // if the class has only one constructor, the @Autowired annotation can be omitted
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
    List<Person> getPersons();
}

public class PersonServiceImpl implements PersonService {
    public List<Person> getPersons() {
        // more code
    }
}

// good
public class PersonService {
    public List<Person> getPersons() {
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
public class PersonDto {
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
public class PersonDto {
    private final String firstname;
    private final String lastname;

    // requires Spring Boot 2.0 or higher to work out of the box
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
public class PersonDto {
    private final String firstname;
    private final String lastname;

    // if the class has a only one constructor, the @JsonCreator annotation can be omitted
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
    void testGetPersons() {
        // given
        PersonRepository personRepository = mock(PersonRepository.class);
        when(personRepository.findAll()).thenReturn(List.of(new Person("Oliver", "Weiler")));

        PersonService personService = new PersonService(personRepository);

        // when
        List<Person> persons = personService.getPersons();

        // then
        assertThat(persons).extracting(Person::getFirstname, Person::getLastname).containsExactly("Oliver", "Weiler");
    }
}
```

* Use [AssertJ](http://joel-costigliola.github.io/assertj/). Avoid [Hamcrest](http://hamcrest.org/).

> Why? `AssertJ` is more actively developed, requires only one static import, and allows you to discover assertions through code completion.

```java
// bad
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.not;
import static org.hamcrest.Matchers.empty;

// more code
assertThat(List.of(new Person("Oliver", "Weiler")), is(not(empty())));

// good
import static org.assertj.core.api.Assertions.assertThat;

// more code

assertThat(persons).isNotEmpty();
```

**[⬆ back to top](#table-of-contents)**

## Monitoring

TBD

**[⬆ back to top](#table-of-contents)**
