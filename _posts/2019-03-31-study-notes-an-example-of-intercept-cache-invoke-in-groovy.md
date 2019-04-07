---
layout: post
title: "Extra Logging Levels using Groovy's Intercept-Cache-Invoke Design Pattern"
---

This article is inspired by, or basically copied from, a demo from [Ken Kousen](http://www.kousenit.com/) where he demonstrated the use of `metaClass` and the Intercept-Cache-Invoke design pattern of Groovy. With this design pattern, you can call methods on a class where the methods aren't even defined. The trick is to override the `methodMissing` method on the `metaClass`, and handle it in whatever ways you want.

An example here is to add more logging levels to the Java's default logging component. The existing levels include:
* SEVERE
* WARNING
* INFO
* CONFIG
* FINE
* FINER
* FINEST

What if you want to add more logging levels to it, like "GOOD", "OK", "TERRIBLE", etc. and you would want to use it like this:
```
logger.info("existing level info")
logger.ok("I guess I'm all right")
logger.terrible("holy shit")
```

First, you'll want to define a class for your custom logging levels. 
```
import groovy.transform.InheritConstructors;
import java.util.logging.Level;

@InheritConstructors
class CustomLevel extends Level {
}
```

Next, override the `methodMissing` method in the `metaClass` of `Logger` object. You can clearly see the three steps of intercept, cache and invoke in the code below. 

```
import java.util.logging.*

// Intercept (using methodMissing)
Logger.metaClass.methodMissing = { String name, args ->
    println "debug: inside methodMissing with $name"
    int val = Level.WARNING.intValue() +
        (Level.SEVERE.intValue() - Level.WARNING.intValue()) * Math.random()
    def level = new CustomLevel(name.toUpperCase(), val)
    def impl = { Object... varArgs ->
        delegate.log(level, varArgs[0])
    }
    // Cache the implementation on the metaClass
    Logger.metaClass."$name" = impl

    // Invoke the new implementation
    impl(args)
}
```

The smart part is how the implementation is created and cached for an unknown method, and how the `impl` Closure delegates the actual method invocation to the `Logger`'s log method in a functional programming way. 

Now, you can actually use the logger. For all undefined methods, this logger will create a new logging level and log it with this level.
```
Logger logger = Logger.getLogger(this.class.name)
logger.ok("I guess I'm all right")
logger.terrible("holy shit")
logger.sleeping("good night")
logger.sleeping("sweet dreams")
```

Note that when an undefined method is called twice, only the first time will the debug message appear. 

Code base: https://github.com/kousen/IntroGroovy/blob/master/src/main/groovy/metaprogramming/use_emc.groovy