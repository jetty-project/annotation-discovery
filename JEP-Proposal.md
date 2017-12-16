```
Title: Annotation Discovery
Author: Greg Wilkins <gregw@webtide.com>
Organization: Webtide LLC
Created: 2017/12/01
Type: Feature
State: Draft
Exposure: Open
Component: modules
Scope: SE
Discussion: discuss at openjdk dot java dot net
Start: 2018/Q1 ?
Depends: 
Effort: M?
Duration: M?
Template: 1.0
```

Summary
-------

An API and implementation(s) for annotation discovery that abstracts away the
details of modules, classpaths, multi-release jars etc.

Goals
-----
- To develop an agreed specification for resolving precisely what classes should be checked for annotations 
when at runtime there are class paths, module paths, upgrade paths, multi release jars
- To develop an API that is widely accepted by multiple frameworks.
- To develop a reference implementation of the API suitable for inclusion in a future release of openJDK

Success Metrics
---------------

- The minimum metric for success is to achieve an agreed specification for annotation discovery that will be 
accepted by multiple frameworks & other specifications (Eg Servlets, JPA, Spring, CDI, etc.)

Motivation
----------

Prior to Java 9, many java technologies implemented annotation discovery by inspection of either the URLs of the 
classloader and/or the class path itself and tools such as [ASM](http://asm.ow2.org/) where then used to parse the 
byte code for annotations without loading the classes.   With the introduction of java 9, which included Modules, 
Layers, Upgrade paths, Jlink and Multi Release jars, the task of discovering class has become difficult to both 
specify and implement.

Efficient annotation discovery was an [identified requirement of project 
Jigsaw](http://openjdk.java.net/projects/jigsaw/spec/reqs/#efficient-annotation-detection):
```text
It must be possible to identify all of the class files in a module artifact in which a 
particular annotation is present without actually reading all of the class files. 
At run time it must be possible to identify all of the classes in a loaded module 
in which a particular annotation is present without enumerating all of the classes 
in the module, so long as the annotation was retained for run time. For efficiency it 
may be necessary to specify that only certain annotations need to be detectable in 
this manner.
```
However as explained in the mailing list, this requirement 
["didn't happen in Java SE 9"](http://mail.openjdk.java.net/pipermail/jigsaw-dev/2017-November/013363.html).
Thus currently multiple frameworks (servlets, EJB, JPA, CDI, Spring etc.) are currently
reverse engineering the algorithms implemented by the JVM for resolving the modules
available at runtime and which classes within those modules are available for loading
by the specific runtime platform.   Such duplicated effort is unlikely to be correct
given the complex nature of module resolution, nor future proof given the intention 
to release java 10 and 11 within the next year.

Description
-----------
At this state the description is very much open for contributions and debate.

#### Requirements
 0. To retrieve the class names of classes have specific class and/or method annotations.
 1. Correctness is the primary requirement, so that even without any compile phase optimisations the correct results will be returned.
 2. Efficiency is a secondary requirement, so that for given annotations efficient lookup can be done, perhaps using compile time optimisations. 
 3. Annotation information must be retrieved without loading classes.
 4. Annotation queries may be scoped by Module, Layer or Classloader
 5. Efficiently support multiple queries, so that the results of one query may be used as the basis of another query without excessive duplication of effort (eg. @HandlerTypes)
 6. Annotations must be discoverable via meta-annotations; e.g. when having a meta-annotation `@interface Qualifier {}` and an annotation `@Qualifier @interface Informal {}`, elements annotated with `@Informal` should be discoverable by querying for `@Qualifier`
 7. The API should allow to discover _declaration annotations_ as well as _type annotations_ (as usable since Java 8)
 8. Discovery of classes that extend classes or implement interfaces may also be supported.

#### Implementation

- Efficient discovery may be assisted by the optional inclusion of annotation metadata within META-INF

Alternatives
------------
At runtime:
- Query the module system for lists of resolved modules. 
- Use a ModuleReader to discover the classes with a module.
- Convert the discovered classes into `jrt://module` URIs
- Use a tools such as ASM to parse the bytes and discover annotations.

Dependencies
-----------
The algorithms required for annotation discovery are highly dependent on the algorithms already implemented 
by the module system and classloaders.  This effort can either develop in parallel with the existing algorithms
or attempt integrate to share the logic for module/classpath resolution.
