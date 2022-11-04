# saucelabs-fat-jars

This repository contains several Java projects to illustrate how to create _fat-jars_ (i.e., Java archives containing all the compiled classes and all dependencies) for executing standalone [Selenium](https://www.selenium.dev/) tests. These projects have been implemented using [Maven](https://maven.apache.org/) and [Gradle](https://gradle.org/) as build tools, and they contain tests based on different frameworks: [JUnit 4](https://junit.org/junit4/), [JUnit 5](https://junit.org/junit5/docs/current/user-guide/), and [TestNG]( https://testng.org/doc/). The required setup for these alternatives is described in the following sections.

## Maven

First, the local dependencies (if any) should be installed in the Maven local repository (`.m2/repository`). In these examples, this dependency is simulated with the project [saucelabs-selenium-diy](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-diy), used as a dependency in the rest ([saucelabs-selenium-junit4](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-junit4), [saucelabs-selenium-junit5](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-junit5), and [saucelabs-selenium-testng](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-testng)). The Maven command to make this installation is:

```
mvn install
```

Then, to create the _fat-jar_ file with Maven, we need a Maven plugin called `maven-assembly-plugin`. The setup of this plugin in the `pom.xml` should be as follows:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.4.2</version>
    <configuration>
        <archive>
            <manifest>
                <mainClass>com.saucelabs.test.RunSuiteMain</mainClass>
            </manifest>
        </archive>
        <descriptors>
            <descriptor>src/test/resources/assembly.xml</descriptor>
        </descriptors>
    </configuration>
</plugin>
```

As you can see, there is an additional XML file (called [`assembly.xml`](https://github.com/bonigarcia/saucelabs-fat-jars/blob/main/saucelabs-selenium-junit5/src/test/resources/assembly.xml) in this example) and a Java main class. This class is required to select the tests to be executed by the _fat-jar_. The implementation of this class is specific to each testing framework (JUnit 4, JUnit 5, or TestNG). See the sections below for further details.

Then, to build the _fat-jar_ file, we need to invoke the following command:

```
mvn compile assembly:single
```

The  _fat-jar_ will be created in the `target` folder. This file is executable throw the following command:

```
java -jar filename-fat.jar
```

## Gradle

First, the local dependencies (if any) should be installed in the Maven local repository (`.m2/repository`). In these examples, this dependency is simulated with the project [saucelabs-selenium-diy](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-diy), used as a dependency in the rest ([saucelabs-selenium-junit4](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-junit4), [saucelabs-selenium-junit5](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-junit5), and [saucelabs-selenium-testng](https://github.com/bonigarcia/saucelabs-fat-jars/tree/main/saucelabs-selenium-testng)). The Gradle command to make this installation is:

```
gradle publishToMavenLocal
```

Then, to create the _fat-jar_ file with Gradle, we need the following setup in the `build.gradle` file:

```
task fatJar(type: Jar) {
    manifest {
        attributes( 
                "Main-Class": "com.saucelabs.test.RunSuiteMain"
        )
    }
    classifier = "fat"
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
    from { 
        sourceSets.test.output
    }
    from { 
        configurations.testRuntimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } 
    } {
        exclude "META-INF/*.SF"
        exclude "META-INF/*.DSA"
        exclude "META-INF/*.RSA"
    }
    with jar
}
```

As you can see, this setup points to a Java main class. This class is required to select the tests to be executed by the _fat-jar_. The implementation of this class is specific to each testing framework (JUnit 4, JUnit 5, or TestNG). See the sections below for further details.

Then, to create the _fat-jar_ file, we need the following command:

```
gradle fatJar
```

The  _fat-jar_ will be created in the `build\libs` folder. This file is executable throw the following command:

```
java -jar filename-fat.jar
```

## JUnit 4

JUnit 4 allows executing tests programmatically using the class `JUnitCore`, for instance, as follows (see the whole [main class](https://github.com/bonigarcia/saucelabs-fat-jars/blob/main/saucelabs-selenium-junit4/src/test/java/com/saucelabs/test/RunSuiteMain.java) example):

```
JUnitCore junit = new JUnitCore();
junit.addListener(new TextListener(System.out));
junit.run(ChromeJUnit4Test.class);
```

## JUnit 5

JUnit 5 allows executing tests programmatically using the [JUnit Launcher API](https://junit.org/junit5/docs/current/user-guide/#running-tests). To use this API, first, we need to declare the `org.junit.platform:junit-platform-launcher` in Maven/Gradle. Then, we can use the following snippet to run the tests in a [Java class](https://github.com/bonigarcia/saucelabs-fat-jars/blob/main/saucelabs-selenium-junit5/src/test/java/com/saucelabs/test/RunSuiteMain.java):

```
// Discover and filter tests
LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder
        .request().selectors(selectPackage("com.saucelabs.test"))
        .filters(includeClassNamePatterns(".*Test")).build();
Launcher launcher = LauncherFactory.create();
launcher.discover(request);

// Executing tests
TestExecutionListener listener = new SummaryGeneratingListener();
launcher.registerTestExecutionListeners(listener);
launcher.execute(request, listener);
```

## TestNG

TestNG allows executing tests programmatically using the class `TestNG`, for instance, as follows (see the whole [main class](https://github.com/bonigarcia/saucelabs-fat-jars/blob/main/saucelabs-selenium-testng/src/test/java/com/saucelabs/test/RunSuiteMain.java) example):

```
TestNG testNG = new TestNG();
testNG.setTestClasses(new Class[] { ChromeNGTest.class });
testNG.run();
```

## About
WebDriverManager (Copyright &copy; 2022) is a project created by [Boni Garcia](https://bonigarcia.dev/) and licensed under the terms of the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).
