# An opinionated guide on application development with Spring Boot

## Dependency Injection

* Never use `field injection` or `setter injection`. Use `constructor injection` instead.

> TODO

```java
// bad
@AutoWired
private PersonRepository personRepositoy;
```

http://olivergierke.de/2013/11/why-field-injection-is-evil/
