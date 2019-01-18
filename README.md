# An opinionated guide on developing web applications with Spring Boot

## Dependency Injection

* Never use `field injection` or `setter injection`. Use `constructor injection` instead.

> TODO

```java
// bad
@AutoWired
private PersonRepository personRepositoy;
```
