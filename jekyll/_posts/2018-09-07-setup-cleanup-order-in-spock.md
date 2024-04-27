---
layout: post
title: Running order of setup and cleanup methods in spock framework 
---

So I've been using spock test framework for a while. From time to time I fall into the tricky part of the running order of the setup and cleanup fixture methods. 

Spock's documentation explains pretty well on what each of its available fixture methods does in the Spock Primer section.
```groovy
def setup() {}              // run before every feature method
def cleanup() {}            // run after every feature method
def setupSpec() {}          // run before the first feature method
def cleanupSpec() {}        // run after the last feature method
``` 

> If fixture methods are overridden in a specification subclass then `setup()` of the superclass will run before `setup()` of the subclass. `cleanup()` works in reverse order, that is `cleanup()` of the subclass will execute before `cleanup()` of the superclass. `setupSpec()` and `cleanupSpec()` behave in the same way. There is no need to explicitly call `super.setup()` or `super.cleanup()` as Spock will automatically find and execute fixture methods at all levels in an inheritance heirarchy.

Then in Data Driven Testing section, it emphasizes that `setup()` and `cleanup()` will be run in each iteration inside a where block. 
> Iterations are isolated from each other in the same way as separate feature methods. Each iteration gets its own instance of the specification class, and the `setup` and `cleanup` methods will be called before and after each iteration, respectively.

Before I actually read its behavior in data driven approach, I wrote a comprehensive experiment to verify that. Here's the child and parent class and their fixture and feature methods:
```groovy
class ParentClass extends Specification {
    def setupSpec() { println 'Parent Class setupSpec()' }
    def cleanupSpec() { println 'Parent Class cleanupSpec()' }
    def setup() { println 'Parent Class setup()' }
    def cleanup() { println 'Parent Class cleanup()' }

    def "test"() {
        setup: println '   Parent Class feature method setup(), a = ' + a
        expect: 1 == 1
        cleanup: println '   Parent Class feature method cleanup(), a = ' + a
        where: a << [1, 2]
    }
}

class ChildClass extends ParentClass {
    def setupSpec() { println 'Child Class setupSpec()' }
    def cleanupSpec() { println 'Child Class cleanupSpec()' }
    def setup() { println '  Child Class setup()' }
    def cleanup() { println '  Child Class cleanup()' }

    def "test"() {
        setup: println '   Child Class feature method setup(), a = ' + a
        expect: 1 == 1
        cleanup: println '   Child Class feature method cleanup(), a = ' + a
        where: a << [1, 2]
    }
}
```
And here's the output when I run the ChildClass specification.
```
Parent Class setupSpec()
Child Class setupSpec()
Parent Class setup()
  Child Class setup()
   Parent Class feature method setup(), a = 1
   Parent Class feature method cleanup(), a = 1
  Child Class cleanup()
Parent Class cleanup()
Parent Class setup()
  Child Class setup()
   Parent Class feature method setup(), a = 2
   Parent Class feature method cleanup(), a = 2
  Child Class cleanup()
Parent Class cleanup()
Parent Class setup()
  Child Class setup()
   Child Class feature method setup(), a = 1
   Child Class feature method cleanup(), a = 1
  Child Class cleanup()
Parent Class cleanup()
Parent Class setup()
  Child Class setup()
   Child Class feature method setup(), a = 2
   Child Class feature method cleanup(), a = 2
  Child Class cleanup()
Parent Class cleanup()
Child Class cleanupSpec()
Parent Class cleanupSpec()
```

This experiment really clears things out for me. As you can see, it is quite straightforward and intuitive. The order for setup is always the class level method first then feature level, and cleanup is in reverse order. 