# gradle-tasks-example
The main goal of this project is to show simple example of gradle's tasks.

_Reference_: https://docs.gradle.org/current/dsl/index.html  
_Reference_: https://docs.gradle.org/current/userguide/more_about_tasks.html

# preface
Everything in Gradle sits on top of two basic concepts: projects and tasks.
* **Project** - Gradle build is made up of one or more projects. A project 
does not necessarily represent a thing to be built. A project does not 
necessarily represent a thing to be built - it might represent a thing to 
be done.
 **Example**: a library JAR, a web application, deploying application to 
 staging, production environment
 
* **Task** -  A task represents some atomic piece of work which a build 
performs.  
  **Example**: compiling some classes, creating a JAR, generating Javadoc  
  _Remark_: we have custom tasks for java, add `apply plugin: java`
  
* `build.gradle` - a build configuration script

# task
* tasks form a `Directed Acyclic Graph`
* tasks are code
    ```
    task upper {
        def list = [1, 2, 3, 4, 5]
        println "test task"
        list.findAll {it % 2 == 0} each {println it}
    }
    ```
* tasks could depend on other tasks
    ```
    task hello {
        println 'hello'
    }
    task afterHello(dependsOn: hello) {
        println 'after hello'
    }
    ```
* extra task properties - you can add your own properties to a task
    ```
    task greeting {
        ext.shouldGreet = false
    }
    
    task greetingPrinter {
        greeting.shouldGreet ? println('hello') : println('bye bye')
    }
    ```
_Remark_: Gradle has a configuration phase and an execution phase. 
After the configuration phase, Gradle knows all tasks that should 
be executed. **Gradle builds the complete dependency graph before any 
task is executed.**

# task outcomes
* **(no label) or EXECUTED** - Task executed its actions.
* **UP-TO-DATE** - Task’s outputs did not change:
    * Task has outputs and inputs and they have not changed.
    * Task has actions, but the task tells Gradle it did not 
    change its outputs.
* **FROM-CACHE** - Task’s outputs could be found from a previous 
execution. Task has outputs restored from the build cache.
* **SKIPPED** - Task did not execute its actions.
    * Task has been explicitly excluded from the command-line.
    * Task has an onlyIf predicate return false.
* **NO-SOURCE** - Task did not need to execute its actions.
Task has inputs and outputs, but no sources. For example, 
source files are `.java` files for `JavaCompile`.

# incremental builds

# excluding tasks from execution

# using predicate

# manual