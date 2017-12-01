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

- Design an algorithm and API that will allow the efficient discovery of classes
annotated with specified annotations without loading those classes. 
- The discovery may be scoped to an individual module or classloader.
- Efficient discovery may be assisted by the inclusion of annotation metadata within META-INF

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