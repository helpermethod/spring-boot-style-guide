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

    // if the class has only one constructor, @Autowired can be omitted
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
    @GetMapping("/persons/{id}")
    public Person show(@PathVariable long id) {
        // more code
    }
}

// good
@RestController
public class PersonController {
    @GetMapping("/persons/{id}")
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
    @RequestMapping(method = RequestMethod.GET, value = "/persons/{id}")
    public Person show(@PathVariable long id) {
        // more code
    }
}

// good
@RestController
public class PersonController {
    @GetMapping("/persons/{id}")
    public Person show(@PathVariable long id) {
        // more code
    }
}
```

* Simplify your controller keeping it thin

> Why? To avoid [SRP](https://en.wikipedia.org/wiki/Single-responsibility_principle#:~:text=The%20single%2Dresponsibility%20principle%20(SRP,it%20should%20encapsulate%20that%20part.)) violations;

> Where should I put my business logic? Keep the bussines logic [encapsulated](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)) into your services or specialized classes;

```java
// bad
@PostMapping("/users")
public ResponseEntity<Void> postNewUser(@RequestBody UserRequest userRequest) {
    if (userRequest.isLessThanEighteenYearsOld()) {
        throw new IllegalArgumentException("Sorry, only users greater or equal than 18 years old.");
    }

    if (!userRequest.hasJob()) {
        throw new IllegalArgumentException("Sorry, only users working.");
    }

    if (!this.userService.hasUsernameAvailable(userRequest.getUsername())) {
        throw new IllegalArgumentException(String.format("Sorry, [%s] is not an available username.", userRequest.getUsername()));
    }

    this.userService.createNewUser(userRequest);

    return ResponseEntity.status(HttpStatus.CREATED).build();
}

// good
@PostMapping("/users")
public ResponseEntity<Void> postNewUser(@RequestBody UserRequest userRequest) {
    this.userService.createNewUser(userRequest);
    return ResponseEntity.status(HttpStatus.CREATED).build();
}

public class UserService {

    // variables declaration

    public void createNewUser(UserRequest userRequest) {
        this.validateNewUser(userRequest);
        UserEntity newUserEntity = this.userMapper.mapToEntity(userRequest);
        this.userRepository.save(newUserEntity);
    }

    private void validateNewUser(UserRequest userRequest) {
        // business validations
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

    // requires your code to be compiled with a Java 8 compliant compiler 
    // with the -parameter flag turned on
    // as of Spring Boot 2.0 or higher, this is the default
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

    // if the class has a only one constructor, @JsonCreator can be omitted
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

> Why? `AssertJ` is more actively developed, requires only one static import, and allows you to discover assertions through autocompletion.

```java
// bad
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.not;
import static org.hamcrest.Matchers.empty;

assertThat(persons), is(not(empty())));

// good
import static org.assertj.core.api.Assertions.assertThat;

assertThat(persons).isNotEmpty();
```

**[⬆ back to top](#table-of-contents)**
