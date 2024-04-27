---
layout: post
title: "Writing and using custom annotations for Spock tests"
---

Spock framework, just like Groovy, is hightly extensible. In our integration test suite, there's a need of categorizing the tests into different types, say "smoke test", "functional test", "deployment test", etc. Although it saves a lot of trouble if different types of tests are written in different specification files, tagging some tests inside one specification better fits our needs. It is actually not that difficult to implement. Spock already has a great documentation on how to write custom annotations. There are two types, global extensions and annotation driven local extensions. Here I'm going to focus on local extensions.

The goal is to create `@SmokeTest`, `@FunctionalTest`, `@Deployment` annotations and we get to choose what type of tests to run. Like this,

```
class SampleSpec extends Specification {
    @SmokeTest
    def 'smoke test'() {
        expect: 1 == 1
    }
    @FunctionalTest
    def 'functional test'() {
        expect: 1+1 == 2
    }
    @DeploymentTest
    def 'deployment test'() {
        expect: 1*1 == 1
    }
}
```

When this specification is run, we can pipe in a "tests" parameter to specify which test to run, e.g. `--tests=smoke,functional`. In cases where the whole specification is tagged with `@FunctionalTest` and only a few feature methods are tagged with `@SmokeTest`, you can even use `--tests=-smoke,functional` to run non-smoke functional tests. Here's how. 

First, define these tags. Functional test is a broad category. I choose to assign a value called `subLevel` as some functional test are more involving while some are quite straightforward. There are also tests on services of different natural languages. A `language` variable is added as well. 

```
@Retention(RetentionPolicy.RUNTIME)
@Target([ElementType.TYPE, ElementType.METHOD])
@ExtensionAnnotation(SmokeTestExtension)
@interface SmokeTest {}

@Retention(RetentionPolicy.RUNTIME)
@Target([ElementType.TYPE, ElementType.METHOD])
@ExtensionAnnotation(FunctionalTestExtension)
@interface FunctionalTest {
    String subLevel() default '1'
    String language() default 'en'
}

@Retention(RetentionPolicy.RUNTIME)
@Target([ElementType.TYPE, ElementType.METHOD])
@ExtensionAnnotation(StressTestExtension)
@interface StressTest {}
```

Then, create an abstract base class that the spcific annotations can extend from.
```
abstract class BaseTestExtension<T extends Annotation> extends AbstractAnnotationDrivenExtension implements IAnnotationDrivenExtension<T> {

    String alias

    @Override
    void visitSpecAnnotation(T annotation, SpecInfo spec) {
        for (FeatureInfo feature : spec.features) doVisit(annotation, feature)
    }

    @Override
    void visitFeatureAnnotation(T annotation, FeatureInfo feature) {
        doVisit(annotation, feature)
    }

    // decide if a feature method should be visited or skipped
    private void doVisit(T annotation, ISkippable skippable) {
        def queries = extractValues(filteredTests)
        skippable.setSkipped(!queries.every { k, v -> annotation."$k"() in v })
    }

    // parse an input test string, e.g. smoke,functional(subLevel=1|2&language=en|fr)
    // define the syntax yourself
    @Memoized
    private def extractValues(String tests) {
        if (!tests) return
        String query = tests.split(',').find { it.startsWith(getAlias()) }
        query = StringUtils.strip(query ? query.substring(getAlias().length()) : '', '()')
        if (!query) return
        def queries = query.split('&').inject([:]) { map, kv ->
            def (key, value) = kv.split('=')
            map[key] = StringUtils.split((String) value, '|')
            return map
        }
        return queries
    }
}
```
The way I control which test to skip or to run is through the `doVisit` method. The `extractValues` method take in an input string from the user, e.g. `smoke,functional` and parse it to decide if a feature method should be visited. 

The actual annotations are defined as simple as this:
```
class SmokeTestExtension extends BaseTestExtension<SmokeTest> {
    String alias = 'smoke'
}
```
```
class FunctionalTestExtension extends BaseTestExtension<FunctionalTest> {
    String alias = 'functional'
}
```
```
class DeploymentTestExtension extends BaseTestExtension<DeploymentTest> {
    String alias = 'deployment'
}
```

Finally, create a Spock configuration file "SpockConfig.groovy" in the classpath to map user input into actual annotations.
```
runner {

    String testsRawString = System.getProperty('tests', '')
    List tests = testsRawString
            .plus(testsRawString.contains('deployment') ? '' : ',-deployment')
            .split(',')
            .collect { def m = it =~ /(-?\w+)(\(\S*\))?/; if (m) m.group(1) }
            .findAll { StringUtils.stripStart((String) it, '-') in TEST_DEFINITIONS }

    def includedTests = tests
            .findAll { !it.startsWith('-') }
            .collect { TEST_DEFINITIONS[it] }
    def excludedTests = tests
            .findAll { it.startsWith('-') }
            .collect { TEST_DEFINITIONS[it.substring(1)] }

    include(*includedTests)
    exclude(*excludedTests)

}
```

Our deployment tests need to be run in a different environment, thus it is turned off by default by appending `-deployment` to the input string. Of course, the `TEST_DEFINITIONS` that wires up the input and annotations needs to be defined as well:
```
public static TEST_DEFINITIONS = [
    'smoke'      : SmokeTest,
    'functional' : FunctionalTest,
    'deployment' : DeploymentTest
]
```

When everything is put together, you can use option `tests=smoke,functional` to filter out custom groups of tests that are tagged with the corresponding annotations. 