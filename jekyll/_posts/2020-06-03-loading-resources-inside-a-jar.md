---
layout: post
title: "Loading resources inside a jar"
---

Recently I encountered an issue that happens only when running inside a jar. Our non-spring test application has an arbitrary number of resouce files that need to be read and parsed. We've been using class loaders, to read either files or folders, and it worked pretty well, until we put our application inside a jar. We noticed that for some reason, the jar just can't read folders by simply using class loaders. In other words, it won't work without knowing the file names beforehand. 

In our case, the file stucture looks like this:
```
src
├───app
|   ├───a.java
|   └───b.java
└───resource
    ├───com.my.templates
    |   ├───base.html
    |   └───index.html
    └───com.my.documents
        ├───hi.json
        └───bye.json
```

I did a little search online because it should be something that others might have already run into. The closest thread I could find is this one: https://stackoverflow.com/questions/3923129/get-a-list-of-resources-from-classpath-directory

If it's a Spring application, it should be straightforward. `PathMatchingResourcePatternResolver` should solve the problem. But unfortunately this does not apply to this application. Fortunately though, we do find an existing wheel that addresses this in a breeze. It is already mentioned in that thread - Ronmamo Reflections.

All I need to do is add a dependency, e.g. in Gradle:
```
dependencies {
    implementation 'org.reflections:reflections:0.9.+'
}
```

Then use its `ResourcesScanner` to scan our files. 
```
Set<String> fileNames = new Reflections("com.my.documents", new ResourcesScanner()).getResources(Pattern.compile(".+\\.json"));
```

This gives you an absolute path for the resource files and I can then further read it using a class loader. Problem solved.

I find this library is really helpful. We also use it for test class scanning and discovery. The only caveat I see is to be careful when filtering by the package prefix. It can easily scan thousands of unnecessary files or classes if you have a large collection of jars on your dependency where the package names match the prefix. Our package name in this case is unique, hence no apparent negative impact on performance. 