A set of extensions to the open-source SCons build tool (www.scons.org)

# Introduction #

What is the Software Construction Toolkit?
  * A toolkit for building applications.
  * Open-source, so you can use it for building open-source products.
  * Based on the open-source SCons build tool.
    * Written in python; easily extended by writing tool modules.
    * Documented.
    * Tested.
    * Your friend!

A sample of features:
  * Build Windows, Linux, Mac desktop platforms with a single set of build files.
  * Support for parallel (multi-processor) builds on all platforms.
  * Support for distributed builds via distcc and Incredibuild.
  * Generate Visual Studio solution and project files.

Why is the only tool in the toolkit a hammer?
  * If you have a hammer, what other tools do you need?
  * Once you start using it, everything looks like a nail - it's flexible enough you want to use it for all your build needs.
  * Getting hammered is just as much fun for project builds as it is for developers.
  * Because typing 'software\_construction\_toolkit.bat' would be too painful.

Why is this project separate from SCons?  Why not just change SCons itself?
  * SCons is a very flexible build tool.  Where we can improve SCons while maintaining its flexibility, we're contributing changes directly back to SCons.
  * Software Construction Toolkit provides a framework of additional builders and tools for SCons to make cross-platform development and testing easier.  This framework does make some assumptions about the structure of a build - for example, that all libraries have distinct names.  To preserve SCons's flexibility, we've kept the parts of the framework which rely on those assumptions in a separate project.

# Documentation #

[Introduction to the Software Construction Toolkit](http://code.google.com/p/swtoolkit/wiki/Introduction)

[Glossary of all functions, variables, and methods added by the Software Construction Toolkit](http://code.google.com/p/swtoolkit/wiki/Glossary)

[Examples of common tasks](http://code.google.com/p/swtoolkit/wiki/Examples)

Developing Software Construction Toolkit:
  * [Writing and running tests](http://code.google.com/p/swtoolkit/wiki/Testing)