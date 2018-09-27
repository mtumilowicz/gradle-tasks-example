# gradle-tasks-example
The main goal of this project is to show simple example of gradle's tasks.

_Reference_: https://docs.gradle.org/current/dsl/index.html  
_Reference_: https://docs.gradle.org/current/userguide/more_about_tasks.html

# preface
Everything in Gradle sits on top of two basic concepts: projects and tasks.
* **Project** - Gradle build is made up of one or more projects.A project does not 
necessarily represent a thing to be built - it might represent a thing to 
be done.  
 **Example**: a library JAR, a web application, deploying application to 
 staging, production environment.
 
* **Task** -  A task represents some atomic piece of work which a build 
performs.  
  **Example**: compiling some classes, creating a JAR, generating Javadoc  
  _Remark_: we have custom tasks for java triggered by plugin:
    ```
    plugins {
        id 'java'
    }
    ```
  
* `build.gradle` - a build configuration script

# task
* tasks form a `Directed Acyclic Graph`
* tasks are code
    ```
    task upper {
            doLast {
            def list = [1, 2, 3, 4, 5]
            println "test task"
            list.findAll {it % 2 == 0} each {println it}
        }
    }
    ```
* tasks could depend on other tasks
    ```
    task hello {
        doLast{
            println 'hello'
        }
    }
    task afterHello(dependsOn: hello) {
        doLast {
            println 'after hello'
        }
    }
    ```
* extra task properties - you can add your own properties to a task
    ```
    task greeting {
        ext.shouldGreet = false
    }
    
    task greetingPrinter {
        doLast{
            greeting.shouldGreet ? println('hello') : println('bye bye')
        }
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
    * Task has an `onlyIf` predicate returns false.
* **NO-SOURCE** - Task did not need to execute its actions.
Task has inputs and outputs, but no sources. For example, 
source files are `.java` files for `JavaCompile`.

# incremental builds
Task takes some inputs and generates some outputs. For example:
`JavaCompile` - input: source files, output: generated class files. 
Other inputs might include things like whether debug information 
should be included.
* **task inputs** - it affects one or more outputs (example: source files,
java version)
* **internal task property** - no impact on outputs (example: maximum 
memory available for compilation)

As part of incremental build, Gradle tests whether any of the task 
inputs or outputs have changed since the last build. If they haven’t, 
Gradle can consider the task up to date and therefore skip executing 
its actions.

1. Gradle takes a snapshot of the inputs. This snapshot contains the 
paths of input files and a hash of the contents of each file. 
1. Gradle then executes the task. 
1. If the task completes successfully, Gradle takes a snapshot of the 
outputs. This snapshot contains the set of output files and a hash of 
the contents of each file. 
1. Gradle persists both snapshots for the next time the task is executed.
1. Each time after that, before the task is executed, Gradle takes a 
new snapshot of the inputs and outputs. 
1. If the new snapshots are the same as the previous snapshots, 
Gradle assumes that the outputs are up to date and skips the task. 
1. If they are not the same, Gradle executes the task and persists 
both snapshots for the next time the task is executed.

**The code of the task is a part of the inputs to the task.** 
When a task, its actions, or its dependencies change between 
executions, Gradle considers the task as out-of-date.

# excluding tasks from execution
* gradle anotherTask --exclude-task test
* gradle anotherTask -x test

# using a predicate
Use the `onlyIf()` method to attach a predicate to a task. 
The task’s actions are only executed if the predicate evaluates to true.
```
task task1 {
    doLast {
        println 'task1'
    }
}

task task2(dependsOn: task1) {
    onlyIf {false}
    doLast {
        println 'task2'
    }
}
```
then executing
* `gradle task1`
    ```
    > Task :task1
    task1
    ```
* `gradle task2`
    ```
    > Task :task1
    task1
    
    > Task :task2 SKIPPED
    ```

# manual
* defining task of specific type: `task taskName(type: Copy)`
* defining dependency on the other task: 
`task taskName(dependsOn: otherTaskName)`
* all predefined task types: https://docs.gradle.org/current/dsl/org.gradle.api.Task.html

# project description
We provide two tasks:
1. `Zip` type - to zip all sources
    ```
    task archive(type: Zip) {
        archiveName = "app.zip"
        destinationDir = file("${buildDir}/archive")
        
        from sourceSets.main.allSource
    }
    ```
1. `Copy` type - to copy zip files to backup folder
    ```
    task backup(type: Copy) {
        from archive
        into "backup"
    }
    ```
    _Remark_: when `Gradle` sees `from archive` it will know that task
    `backup` is `archive`-dependent.