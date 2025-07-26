# __:simple-gradle: Gradle__

Gradle is an open-source build automation tool that streamlines the process of compiling, testing, and deploying software projects. It uses a flexible domain-specific language (DSL) based on Groovy or Kotlin to define build logic, manage dependencies, and support features like incremental builds and parallel task execution—making it especially popular for Java and Android development.

A simple Gradle project typically looks like this:

``` .css title="Project Structure"
# Single-module project
my-project/
├── build.gradle
├── settings.gradle
└── src
    ├── main
    │   └── java
    └── test
        └── java

# Multi-module project
my-app/
├── settings.gradle
├── build.gradle
├── core
│   └── build.gradle
├── api
│   └── build.gradle
└── web
    └── build.gradle
```

- `build.gradle`: Main build script. Supports 2 syntax __Groovy__(`build.gradle`) or __Kotlin__(`build.gradle.kts`)
- `settings.gradle`: Defines your project structure, modules/subprojects
    ``` .groovy
    rootProject.name = 'my-app'

    include 'core', 'api', 'web'
    ```

## [Directories](https://docs.gradle.org/current/userguide/gradle_directories.html)

<figure markdown="span">
    ![Directory example](./img/directories.png)
</figure>

### Gradle User Home directory

By default, the Gradle User Home (~/.gradle) stores global configuration properties, initialization scripts, caches, and log files.

It can be set with the environment variable `GRADLE_USER_HOME`.

- `caches`: Global cache directory (for everything that is not project-specific).
    - `x.x`: Version-specific caches (e.g., to support incremental builds).
    - `jars-3`, `modules-2`: Shared caches (e.g., for artifacts of dependencies).
- `daemon`: Registry and logs of the Gradle Daemon.
- `init.d`: Global initialization scripts.
- `jdks`: JDKs downloaded by the toolchain support.
- `wrapper`: Distributions downloaded by the Gradle Wrapper.
- `gradle.properties`: Global Gradle configuration properties.


### Project Root directory

``` .css title="multi-module project struc"
├── .gradle 
│   ├── 4.8 
│   ├── 4.9 
│   └── ⋮
├── build   
├── gradle
│   └── wrapper 
├── gradle.properties   
├── gradlew 
├── gradlew.bat 
├── settings.gradle.kts 
├── subproject-one  
|   └── build.gradle.kts    
├── subproject-two  
|   └── build.gradle.kts    
└── ⋮
```

## Build Scripts

<figure markdown="span">
    ![Build Script example](./img/build_scripts.png)
</figure>


### Defining Variables

Variables can be declared in several ways:

=== "Project-level Variables"

    Defined directly in your `build.gradle`. `ext` stands for "extra properties" and allows creating global project-level variables.

    ``` .groovy
    ext {
        springVersion = '6.1.3'
        junitVersion = '5.10.1'
    }

    dependencies {
        implementation "org.springframework:spring-web:$springVersion"
        testImplementation "org.junit.jupiter:junit-jupiter:$junitVersion"
    }
    ```

=== "Local Variables"

    Local variables defined within a task or method scope:

    ``` .groovy
    task greet {
        def message = "Hello from Gradle!"

        doLast {
            println message
        }
    }
    ```

=== "External Variables"

    Variables can be externalized in `gradle.properties`

    ``` .ini
    springVersion=6.1.3
    junitVersion=5.10.1
    ```

- Environment variables are accessible by `System.getenv('MY_ENV_VAR')`

https://docs.gradle.org/current/userguide/gradle_directories.html#project_root


## Common directives/blocks

### Tasks

A task in Gradle is a fundamental build block that represents a unit of work (compile, test, run, clean, etc.).

=== "Creating Task"

    ``` .groovy
    task taskName {
        doFirst {
            println 'Executed before!'
        }
        doLast {
            println 'Executed after!'
        }
    }

    task cleanLogs(type: Delete) {
        delete 'logs/'
    }
    ```

=== "Task Dependencies"

    Tasks can depend on other tasks:

    ``` .groovy
    task compile {
        doLast {
            println 'Compiling source...'
        }
    }

    task test(dependsOn: compile) {
        doLast {
            println 'Running tests...'
        }
    }
    ```

=== "Task Lifecycle"

    - `doFirst`: Executes at the beginning of the task.
    - `doLast`: Executes at the end of the task.

    ``` .groovy
    task demo {
        doFirst {
            println 'This runs first'
        }
        doLast {
            println 'This runs last'
        }
    }
    ```

=== "Customizing Tasks"

    You can modify existing tasks:

    ``` .groovy
    tasks.withType(JavaCompile) {
        options.encoding = 'UTF-8'
    }
    ```

### Plugins

Define build behavior and add functionalities like Java, Spring Boot, or Android support. Plugins extend Gradle’s capabilities.

``` .groovy
plugins {
    id 'java'           // Adds Java compilation.
    id 'application'    // Adds support for running apps.
}
```

__Common Plugins__:

??? info "`java`"

    - Automatically configures standard source sets like main and test along with their respective directories (src/main/java, src/test/java, etc.).
    - Provides tasks such as compileJava, processResources, classes, jar, and test to handle compiling code, processing resources, packaging into a JAR, and running tests.
    - Establishes configurations like implementation, api, compileOnly, and runtimeOnly to manage dependencies effectively.

??? info "`base`"

    - Provides a set of foundational tasks that are common to many builds.
    - Introduces tasks like clean (to delete build directories) and assemble (a placeholder for producing build artifacts). These tasks form the base workflow of many projects.
    - Doesn’t impose language-specific conventions, it is ideal for multi-module builds or projects where you want to build a custom workflow without the assumptions that come with language-specific plugins.
    
??? info "`application`"
    - Is designed for projects that produce executable applications, particularly command-line applications.

??? info "`org.springframework.boot`"
    - Spring Boot specific gradle tasks
    - Auto-versioning for spring boot starter dependencies through [Spring Boot BOM(bill of materials)](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-dependencies?smo=true).

??? info "`io.spring.dependency-management`"
    - Enables auto-versioning based on BOM defined by other plugins.


### Repositories

Defines where Gradle fetches dependencies from:

``` .groovy
repositories {
    maven {
        // URL to your Artifactory repository (release or snapshot)
        url "https://artifactory.example.com/artifactory/libs-release-local"
        credentials {
            // Credentials for accessing the repository
            username = project.findProperty("artifactoryUsername") ?: System.getenv("ARTIFACTORY_USERNAME")
            password = project.findProperty("artifactoryPassword") ?: System.getenv("ARTIFACTORY_PASSWORD")
        }
    }
    // Fallback to Maven Central for dependencies not hosted on Artifactory
    mavenCentral()
}
```

??? info "Cache"

    Gradle caches downloaded dependencies, which helps improve build performance.

    Gradle caches artifacts, plugins, and dependencies in the following location by default:
    
    ``` .bash
    rm -rf ~/.gradle/caches`
    ```

    Clearing only the build cache (without removing dependencies):

    ``` .bash
    gradle cleanBuildCache
    ```

??? info "Proxy Configurations"

    Proxy configurations can be done in several ways

    ``` .groovy title="gradle.properties"
    systemProp.http.proxyHost=proxy.example.com
    systemProp.http.proxyPort=8080
    systemProp.http.proxyUser=yourProxyUsername
    systemProp.http.proxyPassword=yourProxyPassword
    systemProp.http.nonProxyHosts=localhost|*.internal.example.com
    systemProp.https.nonProxyHosts=localhost|*.internal.example.com
    ```

    ``` .bash
    export HTTP_PROXY=http://username:password@proxy.example.com:8080
    export HTTPS_PROXY=http://username:password@proxy.example.com:8080
    ```

    ``` .groovy
    repositories {
        maven {
            ...
        }
    }

    System.setProperty('http.proxyHost', 'proxy.example.com')
    System.setProperty('http.proxyPort', '8080')
    System.setProperty('http.proxyUser', 'user')
    System.setProperty('http.proxyPassword', 'password')
    ```

### Dependencies

Specifies project dependencies. Dependecy can have different scopes, which define how dependencies are used in different parts of the project.

``` .groovy
dependencies {
    implementation 'org.apache.commons:commons-lang3:3.14.0'
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.1'
}
```

<div id="spring-boot-bom"></div>   
??? info "Bill Of Materials(BOM)"

    A BOM is a special kind of Maven POM file that defines versions for multiple dependencies so that you don’t have to specify them manually.

    - Spring Boot provides the `spring-boot-dependencies` BOM, which manages all core Spring dependencies.
    - By default, when you apply the Spring Boot Gradle Plugin, it automatically includes the default BOM for that Spring Boot version.

    ``` .groovy
    plugins {
        id 'org.springframework.boot' version '3.2.0' // Applies the default BOM
        id 'io.spring.dependency-management' version '1.1.4' // Manages dependencies
    }

    dependencies {
        implementation 'org.springframework:spring-core' // Version is managed automatically
    }
    ```

??? info "Dependency Managment"

    Sometimes, you may want to override the Spring Boot BOM version to use a different set of dependency versions.
    
    Two approaches to override verions:

    === "Using `platform()`"

        ``` .groovy
        dependencies {
            // Override Spring Boot BOM
            // all dependencies defined here references 3.2.0 BOM for version resolution
            implementation platform("org.springframework.boot:spring-boot-dependencies:3.2.0")

            // Initially inherits version from above BOM
            // But, gets overriden by a specific version
            implementation 'org.springframework:spring-core:6.1.2'   
        }
        ```

        - The `platform(...)` method tells Gradle to use Spring Boot `3.2.0` BOM for dependency resolution.
        - Only dependencies declared in this block will inherit versions from the BOM. i.e. `dependencies` listed along with `platform()`
        - If another module depends on this project, the BOM will not be passed transitively.

        __Best For__:

        - Projects where only specific dependencies need to be controlled via the BOM.
        - Cases where you don't need the BOM to affect transitive dependencies.
        - ==__Keeping Gradle's native dependency resolution behavior.__==


    === "Using `dependencyManagement`"

        ``` .groovy
        dependencyManagement {
            imports {
                mavenBom "org.springframework.boot:spring-boot-dependencies:3.2.0"
            }
        }

        dependencies {
            implementation 'org.springframework:spring-core:6.1.2' // Override a single dependency
        }
        ```

        - The mavenBom(...) inside dependencyManagement makes the BOM apply transitively.
        - All dependencies in the project automatically inherit versions from `spring-boot-dependencies:3.2.0`.
        - If another module depends on this project, it will also inherit these versions.

        __Best For__:
        
        - Multi-module projects where you want all modules to use the same dependency versions.
        - Ensuring consistency across all dependencies in the project.
        - ==__Spring Boot projects using the `io.spring.dependency-management` plugin.__==



__Common Scopes__:

??? info "`implementation`"

    - For dependencies used only in your own code.
    - Does not expose the dependency to other modules.

??? info "`api`"
    
    (For Libraries & Shared Dependencies): Exposes the dependency to other modules that use your library.
    - If another module depends on your module, it also gets this dependency.
    - Don’t use in regular apps, only in libraries.

??? info "`runtimeOnly`"

    - The dependency is only needed when running the application. e.g., JDBC Drivers (MySQL, PostgreSQL).

??? info "`compileOnly`"
    
    - (Used at Compile-Time, Not at Runtime): Used when you only need the dependency at compile time but not at runtime. e.g., Lombok(Annotation processors).
    - Interfaces that will be implemented by another dependency at runtime.

??? info "`testImplementation`"

    - Used for dependencies needed during test compilation and execution. e.g., JUnit, Mockito.

??? info "`testRuntimeOnly`"

    - (Only Needed for Running Tests): Used for dependencies that are needed at test execution but not at test compilation. e.g., JUnit engine


### Test

The `test` block primarily customizes test execution behavior and integrates seamlessly with testing frameworks.

``` .groovy
plugins {
    id 'java'
}

dependencies {
    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform() // Enables running tests with JUnit 5

    // Sets how many parallel test JVM processes Gradle will run simultaneously.
    // Improves execution speed by running tests concurrently.
    maxParallelForks = Runtime.runtime.availableProcessors()

    // Customize the Java Virtual Machine arguments for test execution.
    // Adjust memory settings or environment-specific system properties.
    jvmArgs '-Xms256m', '-Xmx1024m', '-Dfile.encoding=UTF-8'

    // Configure environment variables accessible within your tests
    environment 'ENV', 'test'

    testLogging {
        events "passed", "skipped", "failed"
        exceptionFormat "full"
        showStandardStreams true
    }
}
```

### Group, Version, and Application metadata

Defines project-level metadata.

``` .groovy
group = 'com.example.myapp'
version = '1.0.0'

application {
    mainClass = 'com.example.myapp.Main'
}
```

### Java version and compiler settings

Specifies compatibility settings for Java projects.

``` .groovy
// Include only major versions(11, 22). No minor, and patch versions(11.0.5, 22.0.2)
java {
    toolchain {
//        Allows you to specify which JDK to use for building and testing
//        Gradle downloads and uses the specified JDK version if not available locally.
//        Downloaded JDKs are cached in the Gradle user home directory(~/.gradle/jdks)
//        ex: The build uses JDK 11
        languageVersion = JavaLanguageVersion.of(17)
    }

//    Specifies the Java version of the source code. It defines which Java language features are allowed
//    To restrict the language features to a specific Java version.
//    Ensures the code compatibility for older versions.
//    ex: The source code must adhere to Java 11 features.
    sourceCompatibility = JavaVersion.VERSION_11
    
//    Defines the version of the Java bytecode generated by the compiler
//    Ensures compiled .class files can run on a specific Java version or newer.
//    ex: The compiled bytecode will be compatible with Java 11.
    targetCompatibility = JavaVersion.VERSION_11
}
```

### Allprojects / Subprojects

Configurations applied to all or specific subprojects in multi-module setups.

``` .groovy
allprojects {
    repositories {
        mavenCentral()
    }
}

subprojects {
    apply plugin: 'java'
    sourceCompatibility = '17'
}

# To iterate over projects
allprojects {
    println it.name
}

subprojects.each { subproject ->
    println subproject.name
}
```


## Gradle Wrapper (`gradlew`)

- Explain usage of the wrapper script (`gradlew`) which ensures consistent builds across environments.

``` .bash title="Generate gradle wrapper"
gradle wrapper --gradle-version=8.6
./gradlew build
```

## Gradle Configuration

``` .groovy title="gradle.properties"

// Enable incremental builds and parallel task execution.
org.gradle.parallel=true
org.gradle.caching=true

// Gradle Daemon & Performance Tuning
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx2g -XX:+HeapDumpOnOutOfMemoryError
```

## Conditional Logic & Profiles

``` .groovy
def isProduction = project.hasProperty('prod')

if (isProduction) {
    println 'Running in production mode'
} else {
    println 'Running in development mode'
}
```

``` .bash
// Use the -P flag to pass properties
./gradlew build -Pprod
```


## rootProject

``` .groovy title="rootProject v/s project"
// module: core/build.gradle
println "Project Name: ${project.name}"

// module: ./build.gradle
println "Root Project Name: ${rootProject.name}"
```

``` .groovy title="accessing rootProject tasks"
// module: ./build.gradle
task rootTask {
    doLast {
        println 'Root task executed'
    }
}

// module: core/build.gradle
task coreTask(dependsOn: rootProject.tasks.rootTask) {
    doLast {
        println 'Web task executed after root task'
    }
}
```
