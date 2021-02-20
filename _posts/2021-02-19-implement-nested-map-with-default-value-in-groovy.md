---
layout: post
title: "Implement nested Map with default value as empty Map in Groovy"
---

Groovy does not have support for nested Map with default value out of the box. So in order to insert values in a nested Map, where the ancestor keys do not exist yet, you have to initialize them first. Otherwise, you will get a `NullPointerException`, e.g.:

```
def m = [:]
m['a']['b']['c'] = 1
//java.lang.NullPointerException: Cannot get property 'b' on null object
//  at Script1.run(Script1.groovy:2)
```

Here, I find the workarounds below can be useful if you don't want to do the initialization or don't want to worry about key existance.

## Method 1: `withDefault` method
If you already know how deep your nested Map will be, you can directly use multiple `withDefault` methods and put an empty Map in closure.

```
def m = [:].withDefault { [:].withDefault { [:] } }
m['a']['b']['c'] = 1
​assert m == [a: [b: [c: 1]]]

// enhance withDefault method (or create a new one)
LinkedHashMap.metaClass.withDefault = { int i ->
  if (i <= 0) return [:]
  return MapWithDefault.newInstance([:], { delegate.withDefault(i - 1) })
}

def m = [:].withDefault(3)
m['a']['b']['c']['d'] = 1
assert m == [a: [b: [c: [d: 1]]]]
```

This of course is not ideal if you want the Map to go arbitrarily deep

## Method 2: Modify `getAt` method on `LinkedHashMap` MetaClass
This approach allows you to go as deep as you want, but the limitation is that all of your keys have to be either `String` or non-`String`. Also remember that the dot notation won't work.

```
// If you know your Map keys are always String:
LinkedHashMap.getMetaClass().getAt = { String k ->
  if (!delegate.containsKey(k)) {
    delegate.put(k, [:])
  }
  delegate.get(k)
}

def m = [:]
m['a']['b']['c'] = 1
assert m == [a: [b: [c: 1]]]

// If you know your Map keys do not contain String:
LinkedHashMap.getMetaClass().getAt = { k ->
  if (!delegate.containsKey(k)) {
    delegate.put(k, [:])
  }
  delegate.get(k)
}

def m = [:]
m[1][2][3.3] = 'a'
assert m == [1:[2:[3.3:'a']]]
```

## Method 3: Create a custom class that extends `LinkedHashMap`, then override `get` method
You can create a new class in exchange for some flexibility.
```
class DefaultMap extends LinkedHashMap {
    def get(k) {
        if (!super.containsKey(k)) {
            super.put(k, new DefaultMap())
        }
        super.get(k)
    }
}

def m = new DefaultMap()
m['a']['b']['c'] = 1
assert m == [a: [b: [c: 1]]]
```

## Method 4: Add a new method on `LinkedHashMap` MetaClass
This works fine if you don't need subscript operator or dot notation.
```
LinkedHashMap.metaClass.customGet = { k ->
  if (!delegate.containsKey(k)) {
    delegate.put(k, [:])
  }
  delegate.get(k)
}


def m = [:]
m.customGet('a').customGet('b').put('c', 1)
assert m == [a: [b: [c: 1]]]
```

Any other methods you can think of? Which method do you prefer?
