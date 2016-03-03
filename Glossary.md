For general SCons environment variables and methods see:
http://scons.org/doc/latest/HTML/scons-man.html.

**Contents**


**Contents (full listing)**



---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Global Methods and Variables #

The following global methods and variables are available outside of
environments.

## AddSiteDir ##
Add a site directory, as if passed to the --site-dir option.

Usage: `AddSiteDir(site_dir)`

Args:
  * site\_dir: Site directory path to add, relative to the location of the SConstruct file.

This may be called from the SConscript file to add a local site scons directory
for a project.
This does the following:
  * Adds site\_dir/site\_scons to sys.path.
  * Imports site\_dir/site\_init.py.
  * Adds site\_dir/site\_scons to the SCons tools path.

Example:
```
AddSiteDir('$SOURCE_ROOT/foo/site_scons')
```

## AddTargetGroup ##
Adds a target group, used for printing help.

Usage: `AddTargetGroup(name, description)`

Args:
  * name: Name of target group.  This should be the name of an alias which points to other aliases for the specific targets.
  * description: Description of the target group.  Should read properly when appended to 'The following ' - for example, 'programs can be built'.

Target groups can be used in the following ways:
  * Automatically printed in hammer help ('hammer -h').
  * Can be specified on the command line to build all targets in the group.
  * Can be referenced in a SConscript as env.Alias(name).
  * Can be passed to the Visual Studio solution and source project builders.

Example:
```
# Add the 'dinner' target group.  'hammer dinner' will build all targets in
# the group.
AddTargetGroup('dinner', 'dinner targets can be built.')
AddTargetGroup('appetizers', 'appetizers can be built.')

# Tell env.ComponentProgram() to add its targets to the 'dinner' group
env.Append(COMPONENT_PROGRAM_GROUPS=['dinner'])

# Build some programs.  They will be added to the 'dinner' group.
env.ComponentProgram('beef', beef_inputs)
env.ComponentProgram('cheese', cheese_inputs)

# Whenever we build dinner, we also want to build all the appetizers
env.Alias('dinner', env.Alias('appetizers'))
```

## AddTargetHelp ##

Usage: `AddTargetHelp()`

Adds SCons help for the targets, groups, and modes.  This is called
automatically by BuildEnvironments(); normally, you don't need to call
this.

## BuildEnvironments ##
Build a collection of SConscripts under a collection of environments.

Usage: `BuildEnvironments(environments)`

Args:
  * environments: List of SCons environments.

Returns:
  * List of environments which were actually evaluated (built).

Only environments with $HOST\_PLATFORMS containing the platform specified by
--host-platform (or the native host platform, if --host-platform was not
specified) will be matched.

Each matching environment is checked against the modes passed to the --mode
command line argument (or 'default', if no mode(s) were specified).  If any
of the modes match the environment's $BUILD\_TYPE or any of the environment's
$BUILD\_GROUPS, all the $BUILD\_SCONSCRIPTS (and for legacy reasons,
$BUILD\_COMPONENTS) in that environment will be built.

Example:
```
root_env = Environment(tools = ['component_setup'])

windows_env = root_env.Clone(
    tools = ['target_platform_windows'],
    BUILD_SCONSCRIPTS = ['build.scons'],
)

windows_dbg_env = windows_env.Clone(
    tools = ['target_debug'],
    BUILD_TYPE = 'dbg',
    BUILD_DESCRIPTION = 'Windows debug',
    BUILD_GROUPS = ['default'],
)

windows_opt_env = windows_env.Clone(
    tools = ['target_optimized'],
    BUILD_TYPE = 'opt',
    BUILD_DESCRIPTION = 'Windows optimized',
)

BuildEnvironments([
    windows_dbg_env,
    windows_opt_env,
])
```

## DeclareBit ##
Declares and describes the bit.

Usage: `DeclareBit(bit_name, desc, exclusive_groups=None)`

Args:
  * bit\_name: Name of the bit being described.
  * desc: Description of bit.
  * exclusive\_groups: Bit groups which this bit belongs to.  At most one bit may be set in each exclusive group.  May be a string, list of string, or None.

Adds a description for the bit in the global dictionary of bit names.  All
bits must be described before being used in env.!Bit() / env.AllBits() /
env.AnyBits().

It is ok to declare a bit multiple times, as long as the description is
identical.  Redeclaring a bit with a different description will cause an error
(as well it should, since each bit should mean exactly one thing).

By custom, bit names are lowercase-separated-by-dashes.  This is particularly
useful when using env.SetBitFromOption(), since that prefixes '--' and '--no-'
to the bit name when creating the command line options.

The target\_platform tools declare a default set of bits for the platforms. For
example, target\_platform\_linux declares 'linux' and 'posix' bits.

Bits for a project are most commonly declared in main.scons.

Tools which use bits (by calling the component\_bits methods) should declare
them in the tool's generate() function.

Example:
```
# Declare the 'foo' bit
DeclareBit('foo', 'Enable foo options')

# Ok to declare the same bit somewhere else, as long as it has the same
# description.
DeclareBit('foo', 'Enable foo options')

# The following will fail because the description is different
DeclareBit('foo', 'Do something else')

# Now that the bit is declared, you can use it with the other component_bits
# methods.  For example, to customize an environment in main.scons before you
# pass that environment to BuildEnvironments().

# Subclass the windows build environment to make a 'win-foo' variant
windows_foo_env = windows_env.Clone(BUILD_TYPE='win-foo',
    BUILD_DESCRIPTION='Windows build, foo variant')
windows_foo_env.SetBits('foo')
```
Then down in any SConscript, you can check for the 'foo' bit:
```
# Only include these inputs for 'foo' mode
if env.Bit('foo'):
  inputs += ['foo_only.cpp']
```

## FilterEnvironments ##
Filters a list of environments to those matching the current platform and
mode.

Usage: `filtered_envs = FilterEnvironments(environments)`

Args:
  * environments: List of SCons environments.

Returns:
  * List of environments which match the current platform and mode.

Only environments with $HOST\_PLATFORMS containing the platform specified by
--host-platform (or the native host platform, if --host-platform was not
specified) will be matched.

Each matching environment is checked against the modes passed to the --mode
command line argument (or 'default', if no mode(s) were specified).  If any
of the modes match the environment's $BUILD\_TYPE or any of the environment's
$BUILD\_GROUPS, all the $BUILD\_SCONSCRIPTS (and for legacy reasons,
$BUILD\_COMPONENTS) in that environment will be returned.

FilterEnvironments() is called internally by BuildEnvironments().

## GetTargetGroups ##
Returns the dict of target groups, indexed by group name.

Usage: `groups = !GetTargetGroups()`

The values in this dict are TargetGroup objects, which contain the
following members:
  * TargetGroup.name: Name of the target group.
  * TargetGroup.description: Description of the group.
  * TargetGroup.GetTargetNames(): Returns a list of target names in the group.

Note that GetTargetGroups() will return incomplete information if called
before BuildEnvironments().

GetTargetGroups() is used internally by the component\_targets\_msvc tool.
It might be useful for short-term debugging or in your own tool, but think hard
before calling this from a SConscript - there's probably a better way to do
what you want.

## GetTargetModes ##
Returns the dict of target modes, indexed by mode name.

Usage: `modes = !GetTargetModes()`

The values in this dict are TargetMode objects, which contain the
following members:
  * TargetMode.name: Name of the target mode.
  * TargetMode.GetTargetNames(): Returns a list of target names in the mode.

In this context, 'mode' means one of the build environments which
BuildEnvironments() executed.  We should have called this method
GetTargetBuildEnvironments() - and probably will rename it to that in the
future.

Note that GetTargetModes() will return incomplete information if called
before BuildEnvironments().

GetTargetModes() is used internally by the component\_targets\_msvc tool.
It might be useful for short-term debugging or in your own tool, but think hard
before calling this from a SConscript - there's probably a better way to do
what you want.

## GetTargets ##
Returns the dict of targets, indexed by target name.

Usage: `targets = !GetTargets()`

The values in this dict are !Target objects, which contain the

following members:
  * Target.name: Name of the target.
  * Target.properties: Dict of global properties for the target, indexed by property name.  The values are the property values.
  * Target.mode\_properties: Dict of mode-specific properties for the target, indexed by mode name, for only the modes where the target is built.  The values of this dict are mode-specific dicts of property values, indexed by property name.  (This is a dict of dicts.)

In this context, 'mode' means one of the build environments which
BuildEnvironments() executed.

Note that GetTargets() will return incomplete information if called
before BuildEnvironments().

GetTargets() is used internally by the component\_targets\_msvc tool.
It might be useful for short-term debugging or in your own tool, but think hard
before calling this from a SConscript - there's probably a better way to do
what you want.

## HOST\_PLATFORM ##
Global variable containing the current host platform, currently one of
  * 'WINDOWS'
  * 'MAC'
  * 'LINUX'
This is used internally by the component\_bits tool.  In general, you should
use the host\_windows, host\_linux, and host\_mac component bits rather than
this variable.


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Environment Methods #

The tools in Software Construction Toolkit add the following methods to SCons
environments.

## AllBits ##
Checks if the environment has all the bits.

Usage: `env.AllBits(bits)`

Args:
  * bits: One or mode bits to check (though if you're only checking a single bit, env.Bit() is cheaper)

Returns True if every bit listed is present in the environment.

Example:
```
if env.AllBits('windows', 'debug'):
  inputs += ['foo_debug_only.cpp']
```

## AnyBits ##
Checks if the environment has at least one of the bits.

Usage: `env.AnyBits(bits)`

Args:
  * bits: One or mode bits to check (though if you're only checking a single bit, env.Bit() is cheaper)

Returns True if at least one bit listed is present in the environment.

Example:
```
if env.AnyBits('linux', 'mac'):
  inputs += ['linux_or_mac.cpp']
```

## ApplySConscript ##
Applies a SConscript to the current environment.

Usage: `env.ApplySConscript(sconscript_file)`

Args:
  * sconscript\_file: Name of SConscript file to apply.

Returns the return value from the call to SCons's SConscript().

env.ApplySConscript() should be used when an existing SConscript which sets up
an environment gets too large, or when there is common setup between multiple
environments which can't be reduced into a parent environment which the
multiple child environments Clone() from.  The latter case is necessary
because env.Clone() only enables single inheritance for environments.

env.ApplySConscript() is NOT intended to replace the env.Tool() method.  If
you need to add methods or builders to one or more environments, do that as a
tool (and write unit tests for them).

env.ApplySConscript() is equivalent to the following SCons call:
```
SConscript(sconscript_file, exports={'env':self})
```

The called SConscript should import the 'env' variable to get access to the
calling environment:
```
Import('env')
```

Changes made to env in the called SConscript will be applied to the environment
calling env.ApplySConscript() - that is, env in the called SConscript is a
reference to the calling environment.

If you need to export multiple variables to the called SConscript, or return
variables from it, use the existing SConscript() function.

Example:
```
# Add include paths, etc. for net utils
env.ApplySConscript('$MAIN_DIR/netutils/using_netutils.scons')

# (after the call, env has been modified; in this example, 'USING_NETUTILS' has
# been added to CPPDEFINES.
```
Sample code for a SConscript caled by env.ApplySConscript():
```
# Import a copy of the calling environment
Import('env')

# Changes made here affect the calling environment
env.Append(CPPDEFINES=['USING_NETUTILS'])
```

## Archive7zip ##
7zip archive builder.

Usage: `env.Archive7zip(target, source)`

Stores the sources in the target archive without compression.

Use env.Compress7zip() to create an archive which uses compression.

Example:
```
env.Archive7zip('foo.7z', ['foo.exe'])
```

## Bit ##
Checks if the environment has the bit.

Usage: `env.Bit(bit_name)`

Args:
  * bit\_name: Name of the bit to check.

Returns True if the bit is present in the environment.

Example:
```
if env.Bit('windows'):
  inputs += ['windows.cpp']
```

## BuildSConscript ##
Builds a SConscript based on the current environment.

Usage: `env.BuildSConscript(sconscript_file)`

Args:
  * sconscript\_file: Name of SConscript file to build.  If this is a directory, this method will look for sconscript\_file+'/build.scons', and if that is not found, sconscript\_file+'/SConscript'.

Returns the return value from the call to SConscript().

env.BuildSConscript() should be used when an existing SConscript which builds a
project gets too large, or when a group of SConscripts are logically related
but should not directly affect each others' environments (for example, a
library might want to build a number of unit tests which exist in
subdirectories, but not allow those tests' SConscripts to affect/pollute the
library's environment.

env.BuildSConscript() is NOT intended to replace the env.Tool() method.  If you
need to add methods or builders to one or more environments, do that as a tool
(and write unit tests for them).

env.BuildSConscript() is equivalent to the following SCons call:
```
SConscript(sconscript_file, exports={'env':self.Clone()})
```
or if sconscript\_file is a directory:
```
SConscript(sconscript_file+'/build.scons', exports={'env':self.Clone()})
```

The called SConscript should import the 'env' variable to get access to the
calling environment:
```
Import('env')
```

Changes made to env in the called SConscript will NOT be applied to the
environment calling env.BuildSConscript() - that is, env in the called
SConscript is a clone/copy of the calling environment, not a reference to that
environment.

If you need to export multiple variables to the called SConscript, or return
variables from it, use the existing SConscript() function.

Examples:
```
# Call sub-SConscript to build export lib
env.BuildSConscript('export_lib.scons')

# Look in the tests subdirectory for 'build.scons' or 'SConscript, and build it
env.BuildSConscript('tests')
```
Sample code for a SConscript caled by env.BuildSConscript():
```
# Import a copy of the calling environment
Import('env')

# Changes made here do not affect the calling environment
env.Append(CPPDEFINES=['FOO', 'BAR'])

# Build the shared/dynamic export library
env.ComponentLibrary('foo_export', 'foo_export.cpp', COMPONENT_STATIC=False)
```

## ClearBits ##
Clears the bits in the environment.

Usage: `env.ClearBits(bits)`

Args:
  * bits: One or mode bits to clear.

Example:
```
env.ClearBits('use-that-dll')
```

## CommandOutput ##
Runs a command and captures its output.  Returns a list containing the node for
the output file.

Usage: `env.CommandOutput(target, source)`

Example:
```
# Run foo.exe and capture output to foo.out.
env.CommandOutput('foo.out', 'foo.exe')

# Run bar.exe with the options '-a -b', in the directory './bardir'
env.CommandOutput('bar.out', 'foo.exe',
                  COMMAND_OUTPUT_CMDLINE='$SOURCE -a -b',
                  COMMAND_OUTPUT_RUN_DIR='./bardir')

# Run baz.exe.  Don't echo the output to stdout when running.  Timeout after 30
# seconds.
env.CommandOutput('baz.out', 'baz.exe',
                  COMMAND_OUTPUT_ECHO=False,
                  COMMAND_OUTPUT_TIMEOUT=30)
```

Uses variables:
  * $COMMAND\_OUTPUT\_CMDLINE
  * $COMMAND\_OUTPUT\_ECHO
  * $COMMAND\_OUTPUT\_RUN\_DIR
  * $COMMAND\_OUTPUT\_TIMEOUT
  * $COMMAND\_OUTPUT\_TIMEOUT\_ERRORLEVEL

## ComponentLibrary ##
Builds a library.

Usage: `env.ComponentLibrary(lib_name, source)`

Args:
  * lib\_name: Name of library.
  * sources: List of input files (sources, objects, etc.)
  * (any other args which are valid for the SCons env.Library() builder)

In hammer, use env.ComponentLibrary() instead of SCons's env.Library(),
env.StaticLibrary(), or env.SharedLibrary().  env.ComponentLibrary() differs
in the following ways:
  * Uses $COMPONENT\_PLATFORM\_SETUP to do target-platform-specific setup.
  * Uses $COMPONENT\_STATIC to determine whether the object should be compiled static or shared.
  * Uses $INCLUDES to set up include files which should be specified on the compiler command line.
  * Sets up target aliases, so if you env.ComponentLibrary('foo', ...) you can build the library via 'hammer foo'.
  * Publishes library outputs for use by env.ComponentProgram() and env.ComponentPackage()
  * Replicates the library output to $LIB\_DIR.

Examples:
```
# Compile either a shared or static library, depending on the default value of
# COMPONENT_STATIC
env.ComponentLibrary('foo', ['foo.cpp'])

# Compile a shared library
env.ComponentLibrary('foo2', ['foo.cpp'], COMPONENT_STATIC=False)

# Compile a static library
env.ComponentLibrary('foo3', ['foo.cpp'], COMPONENT_STATIC=True)
```

## ComponentObject ##
Compiles one or more source files to object files.

Usage: Same as SCons's env.Object(): `env.ComponentObject(target, source)`

In hammer, use env.ComponentObject() instead of SCons's env.Object() or
env.SharedObject().  env.ComponentObject() differs in the following ways:
  * Uses $COMPONENT\_PLATFORM\_SETUP to do target-platform-specific setup.
  * Uses $COMPONENT\_STATIC to determine whether the object should be compiled static or shared.
  * Uses $INCLUDES to set up include files which should be specified on the compiler command line.

Examples:
```
# Compile foo.cpp to either a shared or static object, depending on the
# default value of COMPONENT_STATIC:
env.ComponentObject('foo.cpp')

# Compile foo.cpp to a shared object, and include bar.h on the command line
env.Append(INCLUDES=['bar.h'])
env.ComponentObject('foo.cpp', COMPONENT_STATIC=False)

# Compile a.c, b.cpp, and d.cc into objects
env.ComponentObject(['a.c', 'b.cpp', 'd.cc'])

# Compile foo.cpp to an object file named oof.  The suffix (file extension) of
# the object file depends on the target platform and whether it's shared or
# static.
env.ComponentObject('oof', 'foo.cpp')
```

## ComponentPackage ##
Creates a package (that is, a collection of resources from one or more
components) and copies the resources for those components to the destination
directory.

Usage: `env.ComponentPackage(package_name, dest_dir)`

Args:
  * package\_name: Name of package.
  * dest\_dir: Destination directory for package.

Returns the alias node for the package.

Example:
```
env.ComponentProgram('foo', foo_inputs)
env.ComponentProgram('bar', bar_inputs)
env.ComponentPackage('install',
                     '$TARGET_ROOT/install',
                     COMPONENTS=['foo', 'bar'])
```

## ComponentProgram ##
Compiles and links a program.

Usage: `env.ComponentProgram(prog_name, source)`

Args:
  * prog\_name: Name of the program.
  * source: List of input files (sources, objects, etc.)
  * (any other args which are valid for the SCons env.Program() builder)

This builder does the following:
  * Compiles and links the program.
  * Replicates the program and any shared libraries used by the program to $STAGING\_DIR (or to somewhere else, if you've changed $COMPONENT\_PROGRAM\_RESOURCES).

Example:
```
# Create a program 'foo' which links against lib 'bar'
env.Append(LIBS=['bar'])
env.ComponentProgram('foo', ['fooa.cpp', 'foob.cpp', 'fooc.cpp'])
```

## ComponentTestOutput ##
Describes test output which is generated by some other program.

Usage: `env.ComponentTestOutput(test_name, sources)`

Args:
  * test\_name: Name of the test.
  * sources: List of files/nodes output by the test.

This is called internally by env.ComponentTestProgram() to set up its 'run'
alias.

The 'run' alias will be added to the following target groups:
  * If $COMPONENT\_TEST\_ENABLED=False, added to 'run\_disabled\_tests' only; the steps below are not performed.
  * If $COMPONENT\_TEST\_OUTPUT\_GROUPS, added to those groups.
  * If $COMPONENT\_TEST\_SIZE is set, added to 'run\_COMPONENT\_TEST\_SIZE\_tests' (for example, 'run\_large\_tests').


Example:
```
# Running 'precompiled_tests/foo.exe -o foo.dat', creates foo.dat.
test_out = env.Command(['foo.dat'], ['precompiled_tests/foo.exe'],
                       '$SOURCE -o $TARGET')

# Add 'run_foo' as an alias for running that test, which is a medium test.
env.ComponentTestOutput('run_foo', test_out, COMPONENT_TEST_SIZE='medium')
```

## ComponentTestProgram ##
Compiles and links a test program.

Usage: `env.ComponentTestProgram(program_name, sources)`

Args:
  * program\_name: Name of the program.
  * sources: List of input files (sources, objects, etc.)
  * (any other args which are valid for the SCons env.Program() builder)

This builder does the following:
  * Compiles and links the test program.
  * Replicates the test program, its test input, and any shared libraries used by the test program to $TESTS\_DIR (or to somewhere else, if you've changed $COMPONENT\_TEST\_PROGRAM\_RESOURCES).
  * If $COMPONENT\_TEST\_RUNNABLE (the default), sets up a 'run' alias to run the test program.  That is, if your program is 'foo', this creates a 'run\_foo' alias for hammer to run the test if it's been recompiled or its inputs have changed.  (See also the --retest command line option).  See env.ComponentTestOutput() for variables which affect setting up this alias.

Examples:
```
# Create a test program 'foo', and a 'run_foo' target to run the test.
env.ComponentTestProgram('foo', ['foo.cpp'])

# Create a small test program 'foo2' which needs 'a.jpg' in the same directory
# to run the test.
env.ComponentTestProgram('foo2', ['foo.cpp'], COMPONENT_TEST_SIZE='small')
env.Publish('foo2', 'test_input', 'a.jpg')

# Create a test program 'bar', which the 'foo' test program needs (maybe bar is
# a mock server for the foo test program).  Don't add a 'run_bar' alias.
p = env.ComponentTestProgram('bar', ['bar.cpp'], COMPONENT_TEST_RUNNABLE=False)
env.Depends(env.Alias('foo'), p)

# Create a test program 'foo3' which is known to be flaky, so shouldn't be run
# by 'hammer run_all_tests'
env.ComponentTestProgram('foo3', ['foo.cpp'], COMPONENT_TEST_DISABLED=True)
```

## ComponentVSProject ##
Visual Studio target project builder.

Usage: `env.ComponentVSProject(target_name)`

Args:
  * target\_name: Target name to build a project for,.

Builds a Visual Studio project file for the target, which can be used to
build, run, and debug the target in Visual Studio.

The build configurations which this project will support are those specified
on the command line to hammer at the time the project is generated.  That is,
if you want to generate a project file which can be used to build the 'dbg'
and 'opt' build environments, build the project with
'hammer --mode=dbg,opt your\_project\_alias'.  Or perhaps more usefully,
'hammer --mode=all your\_project\_alias'.

Note that this project contains no source code.  This makes it very fast to
build and also means it does not need to be rebuilt unless the path to the
output file changes.  It is recommended to have one target project per target,
then a single source code project from env.ComponetVSDirProject() with the
source for your entire project.  This is very handy at avoiding the
never-ending stream of may-I-reload-your-project dialog boxes from Visual
Studio when the project files are updated.

Usually, it's easier to use env.ComponentVSSolution() to build all of the
target projects for your project at once.

Example:
```
env.Program('foo', ['foo.cpp'])

p = env.ComponentVSProject('foo')
# Add the project file to the 'projects' alias, so that
# 'hammer projects --mode=all' will rebuild all your projects.
env.Alias('projects', p)
```

## ComponentVSSolution ##
Visual Studio solution file builder.

Usage: `env.ComponentVSSolution(self, solution_name, target_names, projects)`

Args:
  * solution\_name: Name of the solution.
  * target\_names: Names of targets or target groups to include in the solution.  This will automatically build projects for them.
  * projects: List of aditional projects not generated by this solution to include in the solution.  May be omitteded if not needed.

This builds a Visual Studio solution file which contains the listed targets,
and builds project files for those targets.  Additional project files
(generated by other component\_targets\_msvs project builder calls, hand-built,
etc.) specified in the projects option will be included in the solution.

If target\_names contains any target groups (see AddTargetGroup()), the
solution will create projects for each of the targets in the group, and place
them all in a solution folder named for the group.

Example:
```
env.Tool('component_targets_msvs')

# Build a source code project
p = env.ComponentVSDirProject('project_source', ['$MAIN_DIR'])

# Build target projects and solution, and include the source code project.
env.ComponentVSSolution('hammer_solution', ['all_programs', 'all_libraries'],
                        projects=[p])
```

## ComponentVSSourceProject ##
Visual Studio source project builder.

Usage: `env.ComponentVSSourceProject(project_name, target_names)`

Args:
  * project\_name: Name of the project.
  * target\_names: List of target names to include source for.

This builder scans the dependency graph to find all sources for the targets,
including the sources and headers for any libraries used to build the targets,
then produces a Visual Studio project file containing that source.  It uses
the $COMPONENT\_VS\_SOURCE\_FOLDERS variable to determine how to map the sources
it finds into folders in the project file.

While this builder generates the most correct list of source files, it can also
be painfully slow to run.  You can get almost as good a result in orders of
magnitude less time using env.ComponentVSDirProject().

Example:
```
env.ComponentVSSourceProject('foo_source', ['foo'])
```

## ComponentVSDirProject ##
Visual Studio directory-based source project builder.

Usage: `env.ComponentVSDirProject(project_name, source)`

Args:
  * project\_name: Name of the project.
  * source: List of source files and/or directories.

This builder scans the directories from the source list to find all files with
suffixes listed in $COMPONENT\_VS\_SOURCE\_SUFFIXES, then produces a Visual Studio
project file containing that source.  It uses
the $COMPONENT\_VS\_SOURCE\_FOLDERS variable to determine how to map the sources
it finds into folders in the project file.

Example:
```
env.ComponentVSDirProject('foo_source', ['$MAIN_DIR'])
```

## Compress7zip ##
7zip compressed archive builder.

Usage: `env.Compress7zip(target, source)`

Stores the sources in the target archive with maximum compression.

Use env.Archive7zip() to create an archive which does not use compression.

Example:
```
env.Compress7zip('foo.7z', ['foo.exe'])
```

## ConcatSource ##
Source concatenation builder.

Usage: `env.ConcatSource(target, source)`

Args:
  * target: Target source-concatenation file.  Will be given the '$CXXFILESUFFIX' suffix if no file suffix is specified.
  * source: List of source files to include.

The source concatenation builder picks out the input source files which have
file suffixes listed in $CONCAT\_SOURCE\_SUFFIXES, and replaces them with a
single source file which uses #includes to reference them.

Benefits:
  * Dramatic speedup of compile times, since the compiler is invoked fewer times on bigger files, and header files are preprocessed fewer times.  Some projects have seen 5x speedup using env.ConcatSource().
  * Better cross-file optimization resulting in smaller, faster binaries.
  * Reduces duplicate instantiation of static template classes.

Risks:
  * Since the source files are all effectively concatenated, there can be symbol name conflicts if you reuse the same static variable or function names in multiple source files.
  * Header files included in one source file can affect other source files.  The most common failure mode is that you include code in b.cc which relies on the header file included in a.cc, so when you compile without source concatenation, b.cc fails with undefined symbols.

Example:
```
inputs = [
    env.ConcatSource('concat_b', ['b1.cc', 'b2.cc', 'b4.cc', 'foo.mm']),
    # b3.cc defines static variables which conflict with b2.cc, so they can't
    # be in the same ConcatSource() call.
    'b3.cc',
]
```

If $CONCAT\_SOURCE\_ENABLE=True, the resulting inputs will be
['concat\_b.cc', 'foo.mm', 'b3.cc'], where concat\_b.cc contains something like:
```
#include "b1.cc"
#include "b2.cc"
#include "b4.cc"
```
Note that 'foo.mm' is not included in concat\_b.cc because '.mm' is not one of
the suffixes listed in $CONCAT\_SOURCE\_SUFFIXES.

If $CONCAT\_SOURCE\_ENABLE=False, the resulting inputs will be
['b1.cc', 'b2.cc', 'b4.cc', 'foo.mm', 'b3.cc'].  That is, the inputs are the
same as if the env.ConcatSource() call was not there.  This makes it easy to
sanity-check or disable the effect of env.ConcatSource().

## Defer ##
Adds a deferred function or modifies defer dependencies.

Usage: `env.Defer(function, name, after)`

Args:
  * function: Function to defer.  May be omitted if this call is to add a relationship between other deferred functions.
  * name: Name of the function or group to defer.  If this is omitted, the name of the function is used.
  * after: Function, name, or list of functions/names which should run before the deferred function/name.

The deferred function should take an env argument, which will be passed the
environment used to call env.Defer(), and will be executed in the same working
directory as the calling SConscript.
(Exception: if this environment is cloned and the clone calls
env.SetDeferRoot() and then env.ExecuteDefer(), the function will be passed
the root environment, instead of the environment used to call env.Defer().)

All deferred functions run after all SConscripts.  Additional dependencies
may be specified with the after= keyword.

Examples:
```
def func(env):
  """Deferred function to do something.

  Args:
    env: Environment context used in env.Defer() call.
  """
  ...

# Defer func() until after all SConscripts
env.Defer(func)

# Defer func() until otherfunc() runs
env.Defer(func, after=otherfunc)

# Defer func() until after SConscripts, put in group 'bob'
env.Defer(func, 'bob')

# Defer func2() until after all funcs in 'bob' group have run
env.Defer(func2, after='bob')

# Defer func3() until after SConscripts, put in group 'sam'
env.Defer(func3, 'sam')

# Defer all functions in group 'bob' until after all functions in group 'sam'
have run.
env.Defer('bob', after='sam')

# Defer func4() until after all functions in groups 'bob' and 'sam' have run.
env.Defer(func4, after=['bob', 'sam'])
```

## ExecuteDefer ##
Executes deferred functions.

Usage: `env.ExecuteDefer()`

This is called automatically by BuildEnvironments().

## Extract7zip ##
Extract from a 7zip archive.

Usage: `env.Extract7zip(target_directory_dummy, source)`

7-zip behaves like an odd combination of tar and gzip. As result this builder
has two ways it can be used.

If you want to extract one layer of a 7-zip archive, you would do:
```
  out_file = env.Extract7zip('outdir/dummy_file', 'archive.7z')
```

This builder has a proper generator and emitter, at SCons time you will get a
list of archive contents.  Be aware this can have performance overhead, since
7zip archives are being explored to generate the DAG.

In typical usage, 7-zip archives often contain a collection of files which has
been archive, and the resulting 7-zip file then compressed (similar to
xyz.tar.gz, but essentially xyz.7z.7z). To handle this gracefully (and be able
to get file nodes from it), the Extract7zip() builder has the
$SEVEN\_ZIP\_PEEL\_LAYERS flag. If you wish to extract the contents of a doubly
7-zipped file you can do this:
```
  many_files = env.Extract7zip('outdir/dummy_file', 'archive.7z', SEVEN_ZIP_PEEL_LAYERS=True)
```

## FilterOut ##
Removes values from existing construction variables in an Environment.  This
is the inverse of SCons's env.Append().

Usage: `env.FilterOut(variable-values pairs)`

For each variable, the values to remove should be a list.

It is not necessary to check if the variables and/or values are already in the
environment before calling env.FilterOut().

Examples:
```
# Remove the SOME_DEFINE flag from CPPDEFINES, if it is present.
env.FilterOut(CPPDEFINES=['SOME_DEFINE'])

# Remove multiple values from variables
env.FilterOut(
    CPPPATH=['$MAIN_DIR/include1'],
    CPPDEFINES=['FLAG1', 'FLAG2'],
)
```

## GatherInputs ##
Find all (non-generated) input files used for a target.

Usage: `env.GatherInputs(target, groups, exclude_pattern)`

Args:
  * target: a target node to find source files for.  For example: `env.File('bob.exe')`
  * groups: a list of patterns to use as categories. For example: `['.*\\.c$', '.*\\.h$']`.  If not specified, uses `['.*']`, which includes all inputs.
  * exclude\_pattern: optional pattern to exclude from the search. For example: `'.*third_party.*'`

Returns a list of lists of files for each category.  Each file will be placed
in the first category which matches, even if categories overlap.  For example:
`[['bob.c', 'jim.c'], ['bob.h', 'jim.h']]`

Note that if the dependency graph is large (that is, you've got lots of
targets, libraries, headers, etc.), env.GatherInputs() can be very slooooow.

Example:
```
p = env.Program('bob', inputs)
bob_headers = env.GatherInputs(p, ['\\.h$'])[0]
```
bob\_headers contains a list of all the header files included by any of
the 'bob' program's inputs, including any headers included by sources
compiled into libs which bob links against.  Note the `[0]` to return the first
list in the list-of-lists returned by env.GatherInputs().

## GetDeferRoot ##

Usage: `env.GetDeferRoot()`

Returns the root environment for defer.  If one of this environment's parents
called env.SetDeferRoot(), returns that environment.  Otherwise returns the
current environment (that is, env).

## GetPublished ##
Returns a list of the published resources of the specified type.

Usage: `env.GetPublished(group_name, resource_type)`

Args:
  * group\_name: Name of resource group, or a list of names of resource groups.
  * resource\_type: Type of resources (string), or a list of resource types.

Returns a flattened list of the source nodes from calls to Publish() for the
specified group and resource type.  Returns an empty list if there are no
matching resources.

env.GetPublished() is usually used in a function which is deferred via
env.Defer(), so that all other SConscripts have a chance to publish resources
first.

Example:
```
def DeferGetTestXml(env):
   xml_files = env.GetPublished('test', 'xml')

env.Defer(DeferGetTestXml)
```

## GetPublishedWithSubdirs ##
Returns a list of the published resources of the specified type, with the
subdir for each resource.

Usage: `env.GetPublishedWithSubdirs(group_name, resource_type)`

Args:
  * group\_name: Name of resource group, or a list of names of resource groups.
  * resource\_type: Type of resources (string), or a list of resource types.

Returns a flattened list of 2-tuples with the source nodes from calls to
Publish() for the specified group and resource type.  Each entry in the list
contains (source, subdir).  Returns an empty list if there are no matching
resources.

env.GetPublishedWithSubdirs() is usually used in a function which is deferred
via env.Defer(), so that all other SConscripts have a chance to publish
resources first.

Example:
```
def DeferGetTestXml(env):
  xml_files = env.GetPublishedWithSubdirs('test', 'xml')
  for src, subdir in xml_files:
    if subdir:
      print 'test: %s goes in subdir %s' % (src, subdir)
    else:
      print 'test: %s goes in the main resource dir' % src

env.Defer(DeferGetTestXml)
```

## Overlap ##
Checks for overlap between the values.

Usage: `env.Overlap(values1, values2)`

Args:
  * values1: First value(s) to compare.  May be a string or list of strings.
  * values2: Second value(s) to compare.  May be a string or list of strings.

Returns the list of values in common after substitution, or an empty list if
the values do not overlap.

Converts the values to a set of plain strings via env.SubstList2() before
comparison, so SCons $ variables are evaluated.

If you're thinking of using this to conditionally evaluate parts of your
SConscript based on what flags are in CPPDEFINES, it is better to use
the component\_bits functions instead.  Those are more explicit and less prone
to errors.  For example, `env.Overlap('$CPPDEFINES', ['OS_WINDOOWS'])` will
silently fail, but `env.Bit('windoows')` will print an error message that the
'windoows' bit has not been defined.

Examples:
```
# Find directories in common between CPPPATH and LIBPATH
common_dirs = env.Overlap('$CPPPATH', '$LIBPATH')

# Check if a.cc, b.cc, or c.cc is in the inputs list
if env.Overlap(inputs, ['a.cc', 'b.cc', 'c.cc']):
  ...
```

## PrintDefer ##
Prints the current defer dependency graph for debugging.

Usage: `env.PrintDefer(print_functions)`

Args:
  * print\_functions: If True (the default), print individual functions in defer groups.

## Publish ##
Publishes resources for use by other scripts.

Usage: `env.Publish(group_name, resource_type, source, subdir)`

Args:
  * group\_name: Name of resource group.
  * resource\_type: Type of resources (string).
  * source: Source file(s) to publish.  May be a string, Node, or a list of mixed strings or Nodes.  Strings will be passed through env.Glob() to evaluate wildcards.  If a source evaluates to a directory, the entire directory will be recursively published.
  * subdir: Optional subdirectory to which the resources should be published, relative to the primary directory for that resource type.  If omitted or None, resources are published in the primary directory for that resource type.

Most component builders automatically publish resources of type 'run' and
'debug'.

Examples:
```
# Publish input.txt and expected.out as test inputs for the 'foo' unit test.
env.ComponentTestProgram('foo', foo_inputs)
env.Publish('foo', 'test_input', ['input.txt', 'expected.out'])

# Publish xml templates for the navigate component.  These should go into the
# 'nav' subdirectory of the templates directory.
env.Publish('navigate', 'templates', 'xml/*.xml', subdir='nav')

# Publish the entire res subdirectory tree as resources for navigate.
env.Publish('navigate', 'resources', 'res')
```

## RelativePath ##
Returns the relative path from source to target.

Usage: `env.RelativePath(source, target, sep, source_is_file)`

Args:
  * source: Source path or node.
  * target: Target path or node.
  * sep: Path separator to use in returned relative path.  Defaults to the separator for the host platform if not specified.
  * source\_is\_file: If True, calculates the relative path from the directory containing the source, rather than the source itself.  Note that if source is a node, you can pass in source.dir instead, which is shorter.

Examples:
```
# Find relative path from $MAIN_DIR to $SOURCE_ROOT/foo
relpath = env.RelativePath('$MAIN_DIR', '$SOURCE_ROOT/foo')

# Find relative path from 'foo.idl' to the obj directory matching the current
# directory
relpath = env.RelativePath('foo.idl', '.', source_is_file=True)
```

## ReplaceStrings ##
Replace Strings builder, does regex substitution on files.

Usage: `env.ReplaceStrings(target, source)`

Args:
  * target: A single target file name or node.
  * source: A single input file name or node.

Uses the $REPLACE\_STRINGS environment variable to determine what strings to
replace.

Example:
```
env.ReplaceStrings('out', 'in', REPLACE_STRINGS = [('a*', 'b'), ('b', 'CCC')])
```
If 'in' has the contents `Haaapy`, this will create a file 'out' with contents
`HCCCpy`.

## Replicate ##
Replicates (copies) source files/directories to the target directory.  Use this
instead of env.Install() or env.Copy().

Usage: `env.Replicate(target, source)`

Args:
  * target: Destination(s) for copy.  Must evaluate to a directory via env.Dir(), or a list of directories.  If more than one directory is passed, the entire source list will be copied to each target directory.
  * source: Source file(s) to copy.  May be a string, Node, or a list of mixed strings or Nodes.  Strings will be passed through env.Glob() to evaluate wildcards.  If a source evaluates to a directory, the entire directory will be recursively copied.

Returns a list of the destination nodes from the calls to env.Install()

env.Replicate is much like env.Install(), with the following differences:
  * Uses hard linking when safely possible.
  * If the source is a directory, recurses through it and calls env.Install() on each source file, rather than copying the entire directory at once.  This provides more opportunity for hard linking, and also makes the destination files/directories all writable.
  * Can take sources which contain env.Glob()-style wildcards.
  * Can take multiple target directories; will copy to all of them.
  * Handles duplicate requests.
  * More flexible support for renaming destination files using the $REPLICATE\_REPLACE variable.

Examples:
```
# Copy all the html files from html/ to the artifacts directory.
env.Replicate('$ARTIFACTS_DIR', 'html/*.html')

# Copy two files.
env.Replicate('dst', ['file1', 'file2'])

# Copy footext.txt to distdir/footxt.bar.
env.Replicate('destdir', ['footxt.txt'],
              REPLICATE_REPLACE = [('\\.txt', '.bar'), ('est', 'ist')])
```
In the example above, note the use of '\\' to escape the '.' character,
so that it doesn't act like the regexp '.' and match any character.

## ReplicatePublished ##
Replicate published resources for the group to the target directory.

Usage: `env.ReplicatePublished(target, group_name, resource_type)`

Args:
  * target: Target directory for resources.
  * group\_name: Name of resource group, or a list of names of resource groups.
  * resource\_type: Type of resources (string), or a list of resource types.

Uses the subdir parameter passed to env.Publish() when replicating source nodes
to the target.

Returns the list of target nodes from the calls to env.Replicate().

Since this calls env.Replicate(), it will also use the $REPLICATE\_REPLACE
variable, if it's set in the calling environment.

Many builders automatically defer their own calls to env.ReplicatePublished()
to replicate the files needed to run the builder output.  Examples include
env.ComponentProgram(), env.ComponentPackage(), and
env.ComponentTestProgram().

It's often easier to use env.ComponentPackage() to gather and publish multiple
resource types from multiple components, instead of using multiple calls to
env.ReplicatePublished().

Example:
```
# (somewhere earlier)
env.Append(COMPONENTS=['foo', 'bar'])

# Defer copying the templates, so that all the other SConscripts get a chance
# to publish them.
def CopyTemplates(env):
  # Copy the templates for all components to $STAGING_DIR/templates.
  env.ReplicatePublished('$STAGING_DIR/templates', '$COMPONENTS', 'templates')

env.Defer(CopyTemplates)
```

## SetBitFromOption ##
Sets the bits in the environment.

Usage: `env.SetBitFromOption(bit_name, default)`

Args:
  * bit\_name: Name of the bit to set from a command line option.
  * default: Default value for bit if command line option is not present (True or False).

This creates two command line options - one to set the bit, one to clear the
bit.  If bit\_name='foo', this will create the options '--foo' and '--no-foo'.

Example:
```
# This checks the command line options.  If '--dice' is specified, sets the
# 'dice' bit.  If '--no-dice' is specified, clears the 'dice' bit.  If neither
# is specified, uses the default value of True (meaning set the 'dice' bit).
env.SetBitFromOption('dice', True)
```

## SetBits ##
Sets the bits in the environment.

Usage: `env.SetBits(bits)`

Args:
  * bits: One or mode bits to set.

Example:
```
env.SetBits('use-that-dll')
```

## SetDeferRoot ##
Sets the current environment as the root environment for defer.

Usage: `env.SetDeferRoot()`

This is called automatically by BuildEnvironments().

Functions deferred by environments cloned from the root environment (that is,
function deferred by children of the root environment) will be executed when
env.ExecuteDefer() is called from the root environment.

Functions deferred by environments from which the root environment was cloned
(that is, functions deferred by parents of the root environment) will be
passed the root environment instead of the original parent environment.
(Otherwise, they would have no way to determine the root environment.)

## SetTargetDescription ##
Convenience function to set a target's global DESCRIPTION property.

Usage: `env.SetTargetDescription(target_name, description)`

Args:
  * self: Environment context.
  * target\_name: Name of the target.
  * description: Description of the target.

This is equivalent to calling
```
env.SetTargetProperty(target_name, all_modes=True, DESCRIPTION=description)
```

## SetTargetProperty ##
Sets one or more properties for a target.

Usage: `env.SetTargetProperty(self, target_name, all_modes, properties)`

Args:
  * target\_name: Name of the target.
  * all\_modes: If True, property applies to all modes.  If false, it applies only to the current mode (determined by self['BUILD\_TYPE']).
  * properties: One or more name-value pairs for the properties to set.  Properties will be converted to strings via env.subst().

This is called automatically by component builders.

Common properties:
| **Property** | **all\_modes** | **Description** |
|:-------------|:---------------|:----------------|
| DESCRIPTION  | True           | Description of the target. |
| EXE          | False          | Name of the executable file created by the target. |
| RUN\_TARGET  | False          | Name of the target associated with running the target. |
| RUN\_CMDLINE | False          | Command line to use when running the target. |
| TARGET\_PATH | False          | Path to the primary output file created by the target. |
These properties are used by the component\_target\_msvs and component\_target\_xml
tools.

Example:
```
env.ComponentPackage('appbundle', ...)

# Since component packages can contain multiple executables, need to specify
# which one should be executed when the package is built and run from Visual
# Studio.  Note that here we're defining a SCons variable to hold the exe
# name so we can use it in multiple places in the call to SetTargetProperty()
# without needing to do python string operations.
env['PACKAGE_EXE'] = '$STAGING_DIR/appbundle/main.$PROGSUFFIX'
env.SetTargetProperty('appbundle',
                      EXE='$PACKAGE_EXE',
                      RUN_CMDLINE='$PACKAGE_EXE --some-option')
```

## SignedBinary ##
Signs a binary file (EXE/DLL) on Windows.

Usage: `env.SignedBinary(target, source)`

Example:
```
env.Tool('code_signing')
env.SignedBinary('hello_signed.exe', 'hello.exe',
                 CERTIFICATE_PATH='bob.pfx',
                 CERTIFICATE_PASSWORD='123',
                 TIMESTAMP_SERVER='')
```

If no certificate file is specified, the input exe will simply be copied to the
output exe.

If an empty timestamp server string is specified, there will be no timestamp.

Uses variables:
  * $CERTIFICATE\_PATH
  * $CERTIFICATE\_PASSWORD
  * $SIGNTOOL
  * $TIMESTAMP\_SERVER

## SubstList2 ##
Replacement subst\_list designed for flags/parameters, not command lines.

Usage: `env.SubstList2(args)`

Args:
  * args: One or more strings or lists of strings.

Returns a flattened, substituted list of strings.

This is primarily used by tool modules.  If you find yourself using this in
your SConscript files, you're getting pretty procedural; you're likely better
off refactoring that code into a method in a tool in your project's site\_scons
directory, and calling that method from your SConscript.  This makes your
procedural code more testable.

SCons's built-in env.subst\_list evaluates (substitutes) variables in its
arguments, and returns a list of lists (one per positional argument).  Since
it is designed for use in command line expansion, the list items are
SCons.Subst.CmdStringHolder instances.  These instances can't be passed into
env.File() (or subsequent calls to env.subst(), either).  The returned
nested lists also need to be flattened via env.Flatten() before the caller
can iterate over the contents.

env.SubstList2() does a subst\_list, flattens the result, then maps the
flattened list to strings.

Example:
```
# It is better to do:
for x in env.SubstList2('$MYPARAMS'):
# than to do:
for x in env.get('MYPARAMS', []):
# and definitely better than:
for x in env['MYPARAMS']:
# which will throw an exception if MYPARAMS isn't defined.
```


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Environment Variables #

The following environment variables are defined by Software Construction
Toolkit.

## ATLMFC\_VC80\_DIR ##
The location of the Windows ATL MFC for VC80 (Visual Studio 2005) tool for
SCons.  Normally set automatically by the atlmfc\_vc80 tool.

## ARTIFACTS\_DIR ##
The location where final artifacts will be stored. (For publication in Pulse).
Typically in $TARGET\_ROOT/artifacts.

## `_`BITS ##
Used internally by the component\_bits tool to track the bits for an
environment.  Do **not** access this directly; use methods provided by the
component\_bits tool instead.

## BUILD\_GROUPS ##
A list of build groups to which the build environment belongs.
By default, all environments belong to the 'all' group.

If an environment has the 'default' build group, it will be built by default
if the user does not explicitly specify a build mode using --mode.

## BUILD\_SCONSCRIPTS ##
The list of SConscripts which BuildEnvironments() will call for each build
environment.  An entry in this list may be file or directory.  If a directory,
BuildEnvironments() will first look for a file named `build.scons` in that
directory, then `SConscript`.

## BUILD\_TYPE ##
The short name of the build type for a build environment.  Common examples:
  * dbg
  * opt
  * prod
This is what the `--mode` command line option matches against when selecting a
build type.  It is also used by default to construct the $TARGET\_ROOT path.

Multiple environments can have the same $BUILD\_TYPE, as long as they are built
on different host platforms.

## BUILD\_TYPE\_DESCRIPTION ##
A longer description for the build environment.  For example,
'Optimized Windows build'.

## CCFLAG\_INCLUDE ##
The compiler option which specifies a header file which should be included on
the command line.  This is '-include' for gcc and '/FI' for msvc.

This is normally set up by the target\_platform tool.

## CCFLAGS\_DEBUG ##
List of flags to pass to the C and C++ compilers if the build environment is
a debug build.

## CCFLAGS\_OPTIMIZED ##
List of flags to pass to the C and C++ compilers if the build environment is
an optimized build.

## CERTIFICATE\_PASSWORD ##
Password for the signing certificate.  Used by the code\_signing tool.

## CERTIFICATE\_PATH ##
Path to the signing certificate.  Used by the code\_signing tool.

## COLLADA\_DIR ##
Path to Collada DOM.

## COLLADA\_DOM\_VERSION ##
Version of Collada DOM.  Set by the collada\_dom tool.

## COMMAND\_OUTPUT\_CMDLINE ##
Command line executed by the env.CommandOutput() method.  May contain $SOURCE,
which will be replaced by the first input passed to env.CommandOutput().  May
also contain other SCons variables; see the SCons help for env.Command().

## COMMAND\_OUTPUT\_ECHO ##
If set to True, output from commands run via env.CommandOutput() will be
echoed to the console.  If set to False, output will not be echoed.  Default is
True.

## COMMAND\_OUTPUT\_RUN\_DIR ##
Directory in which env.CommandOutput() will run its command.  If not
specified, the command will be run in $MAIN\_DIR.

## COMMAND\_OUTPUT\_TIMEOUT ##
Timeout in seconds for the command run via env.CommandOutput().  If the
command has not finished by this timeout, it and all its subprocesses will be
killed.

If 0 or None, there will be no timeout.

## COMMAND\_OUTPUT\_TIMEOUT\_ERRORLEVEL ##
Errorlevel which the env.CommandOutput() internal builder will return if the
command times out.  Normally you shouldn't need to change this; listed here for
completeness of documentation.  (Note that this is not what
env.CommandOutput() returns; it returns a node which points to the output
file.)

## COMPONENT\_LIBRARY\_GROUPS ##
The list of target groups which libraries created by env.ComponentLibrary()
should be added to.  Set to ['all\_libraries'] by default.

## COMPONENT\_LIBRARY\_DEBUG\_SUFFIXES ##
The list of file suffixes for library outputs which are necessary to debug
the library.  Normally set up by the appropriate target\_platform tool.

## COMPONENT\_LIBRARY\_LINK\_SUFFIXES ##
The list of file suffixes for library outputs which are necessary to link
against the library.  Normally set up by the appropriate target\_platform tool.

## COMPONENT\_LIBRARY\_PUBLISH ##
Normally, shared libraries created by env.ComponentLibrary() are not published
directly to the staging directory, unless they are referenced directly or
indirectly by a program or package (env.ComponentProgram() or
env.ComponentPackage()).

Set $COMPONENT\_LIBRARY\_PUBLISH to True for DLLs that
should be published to the staging directory even if no programs linked against
them (for example, a resource DLL).

## COMPONENT\_PACKAGE\_FILTER ##
List of components which should be filtered out of the component list when
creating a package.  This is useful if a package should not include certain
shared libraries, even though other programs or libraries in the package link
against them.

Usually, it's better to mask off these components at the source (that is,
don't link against libraries which you're not going to include), since
filtering them out at this stage runs the risk that the program won't run
because it won't be able to find a shared library or resource it needs.

## COMPONENT\_PACKAGE\_GROUPS ##
The list of target groups which packages created by env.ComponentPackage()
should be added to.  Set to ['all\_packages'] by default.

## COMPONENT\_PACKAGE\_RESOURCES ##
Map (dict) of resource type to destination directory, for resources which are
copied by env.ComponentPackage().  By default, copies the 'run' and 'debug'
resources to $PACKAGE\_DIR.

## COMPONENT\_PLATFORM\_SETUP ##
The function called by a component builder to modify the environment before
calling the underlying SCons builder.  Normally set up by the target\_platform
tool.

## COMPONENT\_PROGRAM\_GROUPS ##
The list of target groups which programs created by env.ComponentProgram()
should be added to.  Set to ['all\_programs'] by default.

## COMPONENT\_PROGRAM\_RESOURCES ##
Map (dict) of resource type to destination directory, for resources which are
copied by env.ComponentProgram().  By default, copies the 'run' and 'debug'
resources to $STAGING\_DIR.

## COMPONENT\_STATIC ##
Determines whether object files created by env.ComponentObject() and libraries
created by env.ComponentLibrary() are shared (dynamic) or static.  If
$COMPONENT\_STATIC is True, they will be static.  If False, they will be shared.

Example:
```
# Force foo to be a static library (.a or .lib), not a DLL
env.ComponentLibrary('foo', 'foo.cpp', COMPONENT_STATIC=True)
```

## COMPONENT\_TEST\_CMDLINE ##
The command line used for the 'run' target created by
env.ComponentTestProgram().  This is also used by the env.ComponentVSProject()
builder to construct the command line for the Visual Studio target project.

Set to '${PROGRAM\_NAME}' by default.

## COMPONENT\_TEST\_ENABLED ##
Provides an easy way to prevent a flaky or broken test from being run by the
build system.

If set to True, the 'run' target created by env.ComponentTestProgram() will be
placed into the groups in $COMPONENT\_TEST\_OUTPUT\_GROUPS and the group for the
test size ('run\_large\_tests', 'run\_medium\_tests', or 'run\_small\_tests').

If set to False, the 'run' target will be placed only in the
'run\_disabled\_tests' group.

## COMPONENT\_TEST\_OUTPUT\_GROUPS ##
The list of target groups which test output (test running) created by
env.ComponentTestProgram() and/or env.ComponentTestOutput() should be added
to.
Set to ['run\_all\_tests'] by default.

See env.ComponentTestOutput() for other variables which affect which group(s)
a test is added to.

## COMPONENT\_TEST\_PROGRAM\_GROUPS ##
The list of target groups which test programs created by
env.ComponentTestProgram() should be added to.  Set to ['all\_test\_programs'] by
default.

## COMPONENT\_TEST\_PROGRAM\_RESOURCES ##
Map (dict) of resource type to destination directory, for resources which are
copied by env.ComponentTestProgram().  By default, copies the 'run', 'debug',
and 'test\_input' resources to $TESTS\_DIR.

## COMPONENT\_TEST\_RUNNABLE ##
Provides a way to keep a test program from being directly runnable at all.
This is useful if you have a test which requires another sub-program to be
present (for example, if you have a named pipe test which needs to start
another process to talk to, but hammer should not attempt to run that
sub-program directly).

If set to True, env.ComponentTestProgram() will create a 'run' target to run
the test program.  Defaults to True.

## COMPONENT\_TEST\_SIZE ##
The size for tests created by env.ComponentTestProgram() and/or
env.ComponentTestOutput().  Defaults to 'large'.

## COMPONENT\_TEST\_SUBSYSTEM\_WINDOWS ##
Controls whether tests on Windows are console or windowed applications.

If set to False (the default), tests are console apps.  This does the following in target\_platform\_windows.py:
```
    env.FilterOut(
        CPPDEFINES=['_WINDOWS'],
        LINKFLAGS=['/SUBSYSTEM:WINDOWS'],
    )
    env.Append(
        CPPDEFINES=['_CONSOLE'],
        LINKFLAGS=['/SUBSYSTEM:CONSOLE'],
    )
```

If set to True, the settings for $CPPDEFINES and $LINKFLAGS are not modified.

This provides a way for your applications to be windowed by default (that is, set
`CPPDEFINES=['_WINDOWS']` and `LINKFLAGS=['/SUBSYSTEM:WINDOWS']`) while keeping your test apps console apps.

Admittedly this isn't the cleanest workaround.

## COMPONENT\_TEST\_TIMEOUT ##
A dict containing the timeouts in seconds for component tests by size.

## COMPONENT\_VS\_ENABLED\_PROJECTS ##
List of projects which should be enabled in the Visual Studio solution file
generated by !ComponentVSSolution().

Defaults to ['$COMPONENT\_VS\_SCONS\_DEFAULT\_TARGETS'], meaning that building the
Visual Studio solution will build the same targets as running 'hammer' without
explicitly specifying any targets on the command line.

## COMPONENT\_VS\_PROJECT\_DIR ##
Directory where project files created by the builders in component\_target\_msvs
will be placed.

Defaults to '$COMPONENT\_VS\_SOLUTION\_DIR/projects'.

## COMPONENT\_VS\_PROJECT\_SCRIPT\_NAME ##
Name of the script which executes hammer.  Normally, projects have a hammer.bat
file in the main project directory which chains to Software Construction
Toolkit's hammer.bat.

Defaults to 'hammer.bat'.

## COMPONENT\_VS\_PROJECT\_SCRIPT\_PATH ##
Visual Studio relative path from the project file to the script which executes
hammer.

Defaults to
`'$$(!ProjectDir)/$VS_PROJECT_TO_MAIN_DIR/$COMPONENT_VS_PROJECT_SCRIPT_NAME'`.
Note the double $$ so that the `env.subst('$COMPONENT_VS_PROJECT_SCRIPT_PATH')`
call done inside component\_targets\_msvs will produce something starting with
`'$(!ProjectDir)/'`, instead of trying to evaluate `'$ProjectDir'` as a SCons
environment variable.

## COMPONENT\_VS\_PROJECT\_SUFFIX ##
File suffix for Visual Studio project files.  Defaults to '.vcproj'.

## COMPONENT\_VS\_SCONS\_DEFAULT\_TARGETS ##
Used internally by the component\_targets\_msvs tool to hold the value of the
SCons DEFAULT\_TARGETS global variable at the time !ComponentVSSolution() is
called.

## COMPONENT\_VS\_SOLUTION\_DIR ##
Directory where solution files created by !ComponentVSSolution() will be
placed.

Defaults to '$DESTINATION\_ROOT/solution'.

## COMPONENT\_VS\_SOLUTION\_SUFFIX ##
File suffix for Visual Studio solution files.  Defaults to '.vcproj'.

## COMPONENT\_VS\_SOURCE\_FOLDERS ##
List of (folder\_name, folder\_path) pairs for organizing source code in projects
built by the component\_targets\_msvs tool.

Source code will be placed into the folder name correspnding to the first
matching path.  If a folder\_name is `None`, source code in the corresponding
folder\_path will not be added to the project.

Example:
```
env['COMPONENT_VS_SOURCE_FOLDERS'] = [
   (None, '$MAIN_DIR/some_devkit'),
   ('project', '$MAIN_DIR'),
   ('thirdparty', '$SOURCE_ROOT/foo'),
   ('other', '$SOURCE_ROOT'),
]
```
In this example,
  * $MAIN\_DIR/some\_devkit/foo.cpp would not be included in the project file, since it matches the first entry.
  * $MAIN\_DIR/bar/baz.cpp would be placed in 'project/bar'.
  * $SOURCE\_ROOT/foo/aaa/bbb.cpp would be placed in 'thirdparty/aaa'.
  * $SOURCE\_ROOT/bar/base.h would be placed in 'other/bar'.

## COMPONENT\_VS\_SOURCE\_SUFFIXES ##
List of file suffixes which will be included in the source file list in Visual
Studio project files created by the builders in component\_targets\_msvs.

Defaults to `['$CPPSUFFIXES', '.rc', '.scons']`.

## COMPONENT\_VS\_SOURCE\_TARGETS ##
Used internally by the component\_targets\_msvs tool.

## COMPONENTS ##
List of components to include in a package created with env.ComponentPackage().

## CONCAT\_SOURCE\_COMSTR ##
Format for what the env.ConcatSource() builder prints when it's about to run.

## CONCAT\_SOURCE\_ENABLE ##
Enables the env.ConcatSource() builder.  If this evaluates to False,
env.ConcatSource() simply passes through its list of input files.

This is most commonly used to make a no-concat build environment derived from
the main build environment:
```
opt_no_concat = optimized_env.Clone(
    CONCAT_SOURCE_ENABLE = False,
    BUILD_TYPE = 'opt-no-concat,
    BUILD_TYPE_DESCRIPTION='Optimized build, source concatenation disabled',
)
```

## CONCAT\_SOURCE\_SUFFIXES ##
List of file suffixes which can be concatenated together using
env.ConcatSource().

Defaults to
`['.c', '.C', '.cxx', '.cpp', '.c++', '.cc', '.h', '.H', '.hxx', '.hpp', '.hh']`

## COVERAGE\_ANALYZER ##
The location of coverage\_analyzer.exe, a Windows specific tool used to convert
the output of VsPerfCmd.exe to lcov format.  You generally will not need to
use or modify this directly.  Used by the code\_coverage tool.

## COVERAGE\_CCFLAGS ##
Platforms specific flags to be added to compiler lines for coverage runs.
Modify this if you need special compile flags when building for coverage.
Typically you will not need to do this.
Used by the code\_coverage tool.

## COVERAGE\_DIR ##
The directory where coverage data and any generated coverage reports will be
stored.
Used by the code\_coverage tool.

By default: $TARGET\_DIR/coverage

## COVERAGE\_ENABLED ##
This variable will be True if the current environment is being built for code
coverage.
Use this if you need different build behavior during coverage.
Set by the code\_coverage tool.

## COVERAGE\_EXTRA\_PATHS ##
Platform specific additional paths that need to be appended to env['PATH'] when
running coverage.
Typically you will not need to modify this.
Used by the code\_coverage tool.

## COVERAGE\_GENHTML ##
Location of a Linux/Mac specific tool to generate html from lcov files.
Typically you will not need to use or set this directly.
Used by the code\_coverage tool.

## COVERAGE\_HTML\_DIR ##
The location where any html reports on coverage will be output.
Currently html is only generated on Linux and Mac.
Used by the code\_coverage tool.

By default: $COVERAGE\_DIR/html

## COVERAGE\_INSTRUMENTATION\_PATHS ##
Paths which coverage tests will be run from.  This only matters for coverage on
Windows, because files must be instrumented in the location in which they will
eventually run.
Used by the code\_coverage tool.

By default: `['$TESTS_DIR', '$ARTIFACTS_DIR']`

## COVERAGE\_LIBS ##
Platform specific libraries to be linked to executables and dlls built for
coverage. Modify this if you need something other than the typical coverage
libraries. Typically you will not need to do this.
Used by the code\_coverage tool.

## COVERAGE\_LINKCOM\_EXTRAS ##
Platform specific commands to be executed after each link step on executables
and dlls built for coverage.  Modify this if you need to do in place
modification of each coverage target.  Typically you will not need to do this.
Used by the code\_coverage tool.

## COVERAGE\_LINKFLAGS ##
Platform specific flags to the linker for executables and dlls built for
coverage.  Modify this if you need something other than the typical coverage
link flags.  Typically you will not need to do this.
Used by the code\_coverage tool.

## COVERAGE\_MCOV ##
The path to mcov, a Linux/Mac tool used to convert gcov data to lcov format for
publication to a build dashboard.
Typically you will not need to set or use this directly.
Used by the code\_coverage tool internally.

## COVERAGE\_START\_CMD ##
Platform specific commands to be executed before any coverage tests are run.
Change this to alter what happens before coverage it run.
Typically the automatic settings are appropriate.
Used by the code\_coverage tool.

## COVERAGE\_START\_FILE ##
Name of a dummy target for the $COVERAGE\_START\_COMMAND.
You should not need to change this.
Used by the code\_coverage tool.

## COVERAGE\_STOP\_CMD ##
Platform specific commands to be executed after all coverage tests are run.
Change this to alter what happens after coverage is run, perhaps to alter
post-processing.
Typically the automatic settings are appropriate.
Used by the code\_coverage tool.

## COVERAGE\_OUTPUT\_FILE ##
The file where final lcov coverage data will be collected.
This file is usually uploaded to a coverage dashboard to publish code coverage
numbers.
Used by the code\_coverage tool.

By default: '$COVERAGE\_DIR/coverage.lcov'

## COVERAGE\_TARGETS ##
A list of targets that will have coverage run on them. These may included the
name of aliases.
You would typically adjust this to exclude tests from coverage runs.
Used by the code\_coverage tool.

NOTE: when running under coverage (particularly on windows) you need to invoke with --mode=MYCOVMODE coverage, in order for vspermon to be started at the right time.

By default: ['run\_all\_tests']

## COVERAGE\_VSINSTR ##
The path to vsinstr.exe, a Windows specific tool used to instrument an
executable or dll for profiling or coverage.
Typically you will not need to set or use this directly.
Used by the code\_coverage tool internally.

## COVERAGE\_VSPERFCMD ##
The path to VsPerfCmd.exe, a Windows specific tool run as a server to capture
profiling or coverage data.
It typically needs be started and stopped, before and after test runs.
Typically you will not need to set or use this directly.
Used by the code\_coverage tool.

## CPPDEFINES\_DEBUG ##
List of definitions to pass the C and C++ compilers if the build environment is
a debug build.

## CPPDEFINES\_OPTIMIZED ##
List of definitions to pass the C and C++ compilers if the build environment is
an optimized build.

## `_`DEFER\_GROUPS ##
Used internally by the defer tool to track the groups of deferred functions.
Do **not** access this directly; use methods provided by the defer tool instead.

## `_`DEFER\_ROOT\_ENV ##
Used internally by the defer tool to track the current root environment for
defer.  Do **not** access this directly; use env.GetDeferRoot().

## DESTINATION\_ROOT ##
The location where build output will be placed.
It is typically $MAIN\_DIR/scons-out, unless redefined or --host-platform is
specified.

## DIRECTX9\_0\_C\_DIR ##
Directory containing Windows !DirectX 9.0c.

## DIRECTX9\_18\_944\_0\_PARTIAL\_DIR\_0\_C\_DIR ##
Directory containing Windows !DirectX 9.18.944.0.

## DIRECTX9\_DIR ##
Directory containing the current version of Windows !DirectX 9.

Set by the directx\_9\_0\_c or directx\_9\_18\_944\_0\_partial tool.

## DISTCC ##
Name of the distcc tool.  Defaults to 'distcc'.

## DISTCC\_COMPILERS ##
Compilers which support distcc.

By default, this is set to ['cc', 'gcc', 'c++', 'g++'].

## DYLIB\_INSTALL\_NAME\_FLAGS ##
Flags to pass to the linker when compiling a shared library (.dylib) for
mac.

By default, this is set to
`['-install_name', '@executable_path/${TARGET.file}']`,
which means that dylibs are expecting to be located in the same directory as
the executables which use them.

## ENABLE\_EXCEPTIONS ##
If set to True, sets the component builders to generate code which uses
exceptions.  Defaults to False.

## GNU\_BINUTILS\_DIR ##
Path to directory containing GNU binutils.

## HOST\_PLATFORM\_SUFFIX ##
Specifies the host platform suffix, if --host-platform option was specified on
the command line.

Used in the definition of $DESTINATION\_ROOT, to keep output
from different host platforms in their own destination directories.  This is
useful if you are building source from a single client in your home directory
on multiple different platforms (not recommended, due to the potential for
confusion).

## HOST\_PLATFORMS ##
The list of host platforms on which a build environment can run.
Usually set up by including the proper target\_platform tool, so you don't
usually need to configure this manually.

An entry of `'*'` will match all host platforms.

## INCLUDES ##
List of header files to include on the command line when a component builder
invokes the C or C++ compiler.

Each header file will be prefixed with $CCFLAG\_INCLUDE.

Use this instead of hand-coding the include files with /FI or -include, since
$INCLUDES properly sets up depdendency information, and hand-coding doesn't.

## LIB\_DIR ##
The directory in which the linkable output libraries compiled by
env.ComponentLibrary() will be placed.
(That is, files of type .lib, .so, .a, .dylib)

$LIB\_DIR is appended to $LIBPATH and $RPATH
so that the linker can find libraries without specifying the full path to each
library (which is problematic to do properly cross-platform).

By default, $LIB\_DIR is set to `$TARGET_ROOT/lib`.

## LINKFLAGS\_DEBUG ##
List of flags to pass to the linker if the build environment is a debug build.

## LINKFLAGS\_OPTIMIZED ##
List of flags to pass to the linker if the build environment is an optimized
build.

## MAIN\_DIR ##
This is the directory in which main.scons or SConstruct is located.
Generally the root of the project. This environment variable should be used in
preference to `#` (hash), because it is expanded as a path even in non-path
contexts and is therefore more flexible.

## MANIFEST\_COM ##
On Windows, command line for running the manifest tool to add a manifest to a
program.

## MANIFEST\_COMSTR ##
On Windows, string to echo to the console when running the manifest tool on a
program.

## MANIFEST\_FILE ##
On Windows, manifest file to use as input to mt.exe.  Override this variable to
pass in a pregenerated manifest file.
  * Defaults to $MANFEST\_FILE\_GENERATED\_BY\_LINK.
  * Set to None to skip adding a manifest.

## MANFEST\_FILE\_GENERATED\_BY\_LINK ##
On Windows, the name of the manifest file generated by the Visual Studio linker
when it is passed the -manifest option.

## MSVC\_BLOCK\_ENVIRONMENT\_CHANGES ##
Controls how target\_platform\_windows modifies its environment for the Visual
Studio shell environment variables PATH, INCLUDE, and LIB.
  * If False, these variables are brought in from the shell environment in which hammer was called.  This is the default when hammer is run in a standalone environment.
  * If True, these variables are preserved around the env.Tool() invocations for the msvc, mslib, and mslink tools.  Those tools attempt to read the location of Visual Studio from the registry, so if their changes to the environment aren't blocked, it prevents hermetic builds from working properly.  This should be used if hammer is run in a hermetic build environment where build tools are checked in rather than locally installed.

## NACL\_SDK\_VALIDATE ##
Determines whether the naclsdk tool validates the presence of the Native
Client SDK.
  * If '1' (the default), checks for the SDK.
  * If '0', does not check.

## OBJ\_ROOT ##
The root location where object files and other intermediate build artifacts are
stored. It is typically `$TARGET_ROOT/obj` == `scons-out/$BUILD_TYPE/obj`,
unless redefined.

By default, generated files are placed in this directory in the same relative
path from $OBJ\_ROOT as their source files are from $MAIN\_DIR.

## PACKAGE\_DIR ##
Used internally by env.ComponentPackage().

## PACKAGE\_NAME ##
Used internally by env.ComponentPackage().

## PLATFORM\_SDK\_DIR ##
Path to preferred Platform SDK.
Currently you must explicitly set this up yourself.
(Not auto-detected)

## PLATFORM\_SDK\_VC80\_DIR ##
Path to Visual Studio VC 8.0 SDK.
Currently you must explicitly set this up yourself.
(Not auto-detected)

## PRE\_EVALUATE\_DIRS ##
List of directory variables which BuildEnvironments() should evaluate for
each build environment before calling all of that environment's
$BUILD\_SCONSCRIPTS.  This is a performance optimization.

By default, this is set by component\_setup to the following list:
```
[
    'ARTIFACTS_DIR',
    'DESTINATION_ROOT',
    'OBJ_ROOT',
    'SOURCE_ROOT',
    'TARGET_ROOT',
    'TOOL_ROOT',
]
```
(Note that the variable names in this list are NOT prefixed with $)

## PROGRAM\_BASENAME ##
Used internally by the component\_builders tool.

## PROGRAM\_NAME ##
Used internally by the component\_builders tool.

## PROJECT\_NAME ##
Used internally by the component\_targets\_msvs tool.

## PROJECT\_SOURCES ##
Used internally by the component\_targets\_msvs tool.

## PYTHON ##
The python executable used to run hammer/SCons.

## REPLACE\_STRINGS ##
List of pairs of regex search and replacement strings for env.ReplaceStrings().
The body of the source file has substitution performed on each pair
(search\_regex, replacement) in order.

SCons variables in the replacement strings (but not regex search strings) are
evaluated.

## REPLICATE\_REPLACE ##
A list of pairs of regex search and replacement strings for env.Replicate().
Each full destination path has substitution performed on each pair
(search\_regex, replacement) in order.

Example:
```
env.Replicate('destdir', ['footxt.txt'],
              REPLICATE_REPLACE = [('\\.txt', '.bar'), ('est', 'ist')])
```
will copy to 'distdir/footxt.bar'

In the example above, note the use of `\\` to escape the '.' character,
so that it doesn't act like the regexp '.' and match any character.

## SDL\_CPPPATH ##
Platform specific header include paths required for the Simple DirectMedia
Layer Library.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SDL\_DIR ##
Platform specific location of the Simple DirectMedia Layer Library.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SDL\_FRAMEWORKPATH ##
Framework paths required by the Simple DirectMedia Layer Library on MacOS.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SDL\_FRAMEWORKS ##
Frameworks required by the Simple Direct-Media Layer Library on MacOS.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SDL\_LIBPATH ##
Platform specific library include paths required for the Simple DirectMedia
Layer Library.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SDL\_LIBS ##
Platform specific libraries required for the Simple DirectMedia Layer Library.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SDL\_MODE ##
Mode for running SDL.
  * If 'local', looks for a local install of SDL.
  * If 'hermetic' (the default), assumes SDL is installed in one of
> > $SDL\_HERMETIC\_WINDOWS\_DIR, $SDL\_HERMETIC\_MAC\_DIR, or
> > $SDL\_HERMETIC\_LINUX\_DIR, depending on the current platform.
  * If 'none', assumes SDL is not present at all.

## SDL\_VALIDATE\_PATHS ##
A list of string pairs. This first element of each pair is checked for
existence to verify the presence of particular Simple DirectMedia Layer
components.
Typically set automatically by the sdl tool.
Used by the sdl tool.

## SEVEN\_ZIP ##
Path to the 7zip executable.

## SEVEN\_ZIP\_ARCHIVE\_OPTIONS ##
Command line options passed to 7zip by env.Archive7zip()

## SEVEN\_ZIP\_COMPRESS\_OPTIONS ##
Command line options passed to 7zip by env.Compress7zip()

## SEVEN\_ZIP\_DIR ##
Directory where 7zip is installed.

## SEVEN\_ZIP\_OUT\_DIR ##
Used internally by the seven\_zip tool.

## SEVEN\_ZIP\_PEEL\_LAYERS ##
If this variable is set to True, the SevenZipEmitter pseudo-builder will
extract the contents of a 7-zip archive compressed with 7-zip.
If it is False, the archive will be uncompressed, but not extracted.
Used by the seven\_zip tool.

## SHMANIFEST\_COM ##
On Windows, command line for running the manifest tool to add a manifest to a
DLL.

## SHMANIFEST\_COMSTR ##
On Windows, string to echo to the console when running the manifest tool on a
DLL.

## SIGNTOOL ##
The path to the Microsoft signing tool.  Used by the code\_signing tool.

## SOLUTION\_NAME ##
Used internally by the component\_targets\_msvs tool.

## SOLUTION\_TARGETS ##
Used internally by the component\_targets\_msvs tool.

## SOURCE\_ROOT ##
The top-most source directory.

This is frequently the base directory for your SCM client (for example,
`//depot` for a perforce server).

When referencing files from outside your project, reference them down from
$SOURCE\_ROOT, not up from your project using '..'.

Example:
```
# Good
env.Append(CPPPATH=['$SOURCE_ROOT/projects/foo'])

# Bad
env.Append(CPPPATH=['../foo'])
```

## STAGING\_DIR ##
The directory in which programs compiled by env.ComponentProgram() will be
output.

By default, $STAGING\_DIR is set to `$TARGET_ROOT/staging`.

## TARGET\_DEBUG ##
Set to True if the current build environment is derived from target\_debug.

This is used internally by several hammer tools which can not access the
component\_bits methods.

If you need to do something different in your SConscripts based on whether
you're building in debug mode, use `env.Bit('debug')` instead.
```
# Good
if env.Bit('debug'):

# Bad
if env.get('TARGET_DEBUG'):
# It's particularly bad because it will silently fail if typo'd:
if env.get('TARGET_DEBBUG'):
# Worse (will crash if TARGET_DEBUG wasn't defined)
if env['TARGET_DEBUG']:
```

## TARGET\_NAME ##
Used internally by component\_targets\_msvs.

## TARGET\_PATH ##
Used internally by component\_targets\_msvs.

## TARGET\_PLATFORM ##
Target platform name.

This is used internally by several Software Construction Toollit tools which
can not access the component\_bits methods.

If you need to do something different in your SConscripts based on the target
platform, use the component\_bits methods instead.
```
# Good
if env.AnyBits('windows', 'mac'):
if env.Bit('posix'):

# Bad
if env.get('TARGET_PLATFORM') in ('WINDOWS', 'MAC'):
# It's particularly bad because it will silently fail if either the name or
# value doesn't exactly match.  The following fails both ways:
if env.get('TARGET_PLATFORMM') == 'Windows':
# Worse (will crash if TARGET_PLATFORM wasn't defined)
if env['TARGET_PLATFORM'] == 'LINUX':
```

## TARGET\_ROOT ##
The root location where build output for a particular BUILD\_TYPE is placed.
It is typically
`$DESTINATION_ROOT/$BUILD_TYPE` == `scons-out/$BUILD_TYPE`, unless
redefined.

## TEST\_OUTPUT\_DIR ##
When env.ComponentTestProgram() creates a 'run' target for a test program to
run the test, the output of that target will be placed in $TEST\_OUTPUT\_DIR

By default, $TEST\_OUTPUT\_DIR is set to `$TARGET_ROOT/test_output`.

## TESTS\_DIR ##
The directory in which test programs compiled by env.ComponentTestProgram()
will be output.

By default, $TESTS\_DIR is set to `$TARGET_ROOT/tests`.

## THIRD\_PARTY ##
Directory containing third-party client code.

This is usually defined relative to $SOURCE\_ROOT, to make adding source or
include paths from third-party code more portable, for example:
`$SOURCE_ROOT/client/third_party`.

## TIMESTAMP\_SERVER ##
Timestamp server for code signing.  Used by the code\_signing tool.

## TOOL\_ROOT ##
Common root location for tools outside of the current project directory.
Normally set to $SOURCE\_ROOT.

(This is overridden by the internal hammer unit tests.  Normally, you don't
need to alter this.)

## VC80\_DIR ##
Path to directory containing Visual C++ 8.0.


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Tools #

Tools may be added to an environment when cloning it:
```
win_env = root_env.Clone(tools=['target_platform_windows'])
```
or explicitly by calling env.Tool():
```
win_env.Tool('replace_strings')
```

## atlmfc\_vc80 ##
Sets up environment to use Windows ATL MFC for VC80 (Visual Studio 2005).

## code\_coverage ##
Code coverage tool. This tool can be used to add code coverage to you project
(currently C/C++ specific).  Assuming your project is fairly generic, adding
code coverage is easy. Simply clone your debug environments and apply the
coverage tool.

For example:
```
windows_coverage_env = windows_debug_env.Clone(
   tools = ['code_coverage'],
   BUILD_TYPE = 'coverage-win',
   BUILD_TYPE_DESCRIPTION = 'Windows code coverage build',
)
```
The coverage output will be in lcov format at: $COVERAGE\_OUTPUT\_FILE.
Additionally on Linux/Mac, a coverage html report will be generated at
$COVERAGE\_HTML\_DIR.

Note that on Windows, only some versions of Visual Studio (such as the
team edition) come with the VsPerfCmd.exe utility required to process
coverage data.

## code\_signing ##
Code signing build tool.  Adds the env.SignedBinary() method.

## collada\_dom ##
Sets up environment to use Collada DOM 1.3.0.

## command\_output ##
Command output builder.  Adds the env.CommandOutput() method.

This tool is automatically included by component\_setup.

## component\_bits ##
Environment bit support.

Adds the following global methods
  * DeclareBit()
Adds the following environment methods:
  * env.AllBits()
  * env.AnyBits()
  * env.Bit()
  * env.ClearBits()
  * env.SetBitFromOption()
  * env.SetBits()
Declares the following bits using DeclareBit():
  * debug
  * host\_linux
  * host\_mac
  * host\_windows
  * linux
  * mac
  * posix
  * windows

These functions are the preferred way to conditionally include parts of your
SConscript files, since they do parameter-checking and are thus less
susceptible to typos.

Automatically sets the proper host bit (either host\_linux, host\_mac, or
host\_windows) based on the current host platform.

This tool is automatically included by component\_setup.

## component\_builders ##
Builder methods.

Adds the following environment methods:
  * env.ComponentPackage()
  * env.ComponentObject()
  * env.ComponentLibrary()
  * env.ComponentProgram()
  * env.ComponentTestProgram()
  * env.ComponentTestOutput()

This tool is automatically included by component\_setup.

## component\_setup ##
Software Construction Toolkit component setup.

This must be the first tool included in your root environment (that is, the
environment from which all other environments are cloned).
```
root_env = Environment('component_setup')
```
The component\_setup tool also adds the following other tools to the environment
automatically:
  * command\_output
  * component\_bits
  * component\_builders
  * component\_targets
  * concat\_source
  * defer
  * environment\_tools
  * publish
  * replicate

## component\_targets ##
Target management methods.

Adds the following global methods:
  * AddTargetGroup()
  * AddTargetHelp()
  * GetTargetGroups()
  * GetTargetModes()
  * GetTargets()

Adds the following environment methods:
  * env.SetTargetDescription()
  * env.SetTargetProperty()

This tool is automatically included by component\_setup.

## component\_targets\_msvs ##
Builders to create Visual Studio solutions and projects.

Adds the following environment methods:
  * env.ComponentVSDirProject()
  * env.ComponentVSProject()
  * env.ComponentVSSolution()
  * env.ComponentVSSourceProject()

## component\_targets\_xml ##
Tool to create XML representation of the targets which can be generated by
hammer.

If this tool is included in any environment, running
```
hammer --mode=all targets_xml
```
will generate an XML file '$DESTINATION\_ROOT/targets.xml' describing all the
targets which can be built.

## concat\_source ##
Source concatenation builder.

Adds the following environment methods:
  * env.ConcatSource()

## defer ##
Support for deferring evaluation of parts of a SConscript or tool.

Adds the following environment methods:
  * env.Defer()
  * env.ExecuteDefer()
  * env.GetDeferRoot()
  * env.PrintDefer()
  * env.SetDeferRoot()

This tool is automatically included by component\_setup.

## directx\_9\_0\_c ##
Sets up environment to use Windows !DirectX 9.0c.

## directx\_9\_18\_944\_0\_partial ##
Sets up environment to use Windows !DirectX 9.18.944.0.

This will enable distcc support for the compilers listed in $DISTCC\_COMPILERS.

You must set up the variables $DISTCC\_HOME and $HOME in your shell environment
(that is, your bash shell environment, not the SCons environment variable).

## distcc ##
Adds distcc support.

## environment\_tools ##
Adds utility methods to SCons environments.

Adds the following environment methods:
  * env.ApplySConscript()
  * env.BuildSConscript()
  * env.FilterOut()
  * env.Overlap()
  * env.RelativePath()
  * env.SubstList2()

This tool is automatically included by component\_setup.

## gather\_inputs ##
Adds methods to walk the dependency graph to find input files for a target.

Adds the following environment methods:
  * env.GatherInputs()

## publish ##
Support for publishing and consuming lists of nodes.

Adds the following environment methods:
  * env.GetPublished()
  * env.GetPublishedWithSubdirs()
  * env.Publish()
  * env.ReplicatePublished()

This tool is automatically included by component\_setup.

## replace\_strings ##
Search and replace builder.

Adds the following environment methods:
  * env.ReplaceStrings()

## replicate ##
Replicate tool.

Adds the following environment methods:
  * env.Replicate()

This tool is automatically included by component\_setup.

## sdl ##
Sets up environment to use the Simple DirectMedia Layer library.

## seven\_zip ##
Adds 7zip builders.

Adds the following environment methods:
  * env.Archive7zip()
  * env.Compress7zip()
  * env.Extract7zip()

## target\_debug ##
Modifies an environment to build debug mode.

This is usually applied on an environment cloned from one which applied one
of the target platforms.

Example:
```
root_env = Environment(tools=['component_setup'])

windows_env = root_env.Clone(tools=['target_platform_windows'])

windows_debug_env = windows_env.Clone(
    BUILD_TYPE='dbg',
    BUILD_TYPE_DESCRIPTION='Windows debug',
    tools=['target_debug'],
)
```

Inside a SConscript, you can use the 'debug' bit to determine if the current
build environment is debug:
```
if env.Bit('debug'):
  inputs += 'test_debug_only.cpp'
```

## target\_optimized ##
Modifies an environment to build optimized/release mode.

This is usually applied on an environment cloned from one which applied one
of the target platforms.

Example:
```
root_env = Environment(tools=['component_setup'])

windows_env = root_env.Clone(tools=['target_platform_windows'])

windows_opt_env = windows_env.Clone(
    BUILD_TYPE='opt',
    BUILD_TYPE_DESCRIPTION='Windows optimized',
    tools=['target_optimized'],
)
```

Inside a SConscript, you can use the 'debug' bit to determine if the current
build environment is optimized:
```
if not env.Bit('debug'):
  inputs += 'test_opt_only.cpp'
```

## target\_platform\_linux ##
Sets up an environment to build for Linux.

## target\_platform\_mac ##
Sets up an environment to build for MacOS.

Adds the env.Bundle() environment method.
However, since this method is out of date, we don't recommend using it.

## target\_platform\_windows ##
Sets up an environment to build for Windows.

## visual\_studio\_solution ##
Old Visual Studio solution file builder for hammer, based on the msvs tool.
New projects should use component\_targets\_msvs instead.

Adds the env.Solution() environment method.

## windows\_hard\_link ##
Adds hard link support for env.Install() and env.Replicate() on Windows.

This tool is automatically included by component\_setup.


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Command Line Options #

These options can be passed to Hammer when run via hammer.sh or hammer.bat,
or to SCons if the Software Construction Toolkit site-scons directory is
copied directly into your project or specified via --site-dir.

## --brief ##
Reduces the amount of output from hammer.  Does not echo the full command lines
which hammer executes.

This and --verbose are opposites.  --brief is the default.

Use --silent to turn off even more output.

## --host-platform ##
Forces hammer to use the specified platform as the host platform, instead of
autodetecting the platform on which hammer is being run.

Usually, hammer autodetects the host platform, so this option is not necessary.

Useful for examining the dependency tree which would be created on another
platform, but not useful for running the build because it'll attempt to use the
wrong tools for your platform.

## --mode ##
Sets the build mode used by BuildEnvironments() to control which build
environment(s) are executed.  A mode is a build type set via $BUILD\_TYPE, a
build mode set via $BUILD\_MODE, a group set via $BUILD\_GROUPS, or a
comma-delimited list of types and/or modes.

We called this option --mode rather than having --type and --group options,
since --mode can specify a single build type, a build group, or a combination
thereof.

## --retest ##
Normally, hammer only re-runs tests when the test is recompiled or its inputs
change.  This is handy when you want to 'hammer run\_all\_tests' and have it
compile and re-run only the tests which could have been affected by whatever
changes you've made to the source.

Use the --retest option to force hammer to delete its cached test output and
re-run the specified test(s).

Used by the component\_builders tool.

## --site-path ##
Specifies a comma-delimited list of site directories which SCons should
include, as if individually passed to the SCons --site-dir option.  See the
SCons --site-dir option for more details.

## --verbose ##
Increases the amount of output from hammer.  Echoes the full command lines
which hammer executes.

This and --brief are opposites.

**Note:** Use --verbose in continuous builds; it makes it easier to check the
command lines when there is a build error.


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

SCons Variables Used by Software Contruction Toolkit

Since Software Contruction Toolkit extends SCons, you can use any of the SCons
builders and variables.  This section only lists the most common SCons
variables, and/or SCons variables which are used in extended ways.

## CC ##
Name of the C compiler.

## CCFLAGS ##
List of flags to pass to the C and C++ compilers.

## CPPDEFINES ##
List of definitions to pass the C and C++ compilers.  These will be given the
compiler-appropriate prefix (/D or -D).

## CPPSUFFIXES ##
List of file suffixes which are considered C source code by SCons.  Also used
by the Visual Studio project builders in component\_targets\_msvs.

## CXX ##
Name of the C++ compiler.

## CXXFILESUFFIX ##
Suffix for a C++ file.  Used by env.ConcatSource() to determine its output
filename if no file suffix is specified.

## ENV ##
Dictionary containing the shell environment passed to subprocesses started by
SCons.

Note that SCons variables are not expanded in the shell environment.  See the
examples below for how to work around this.

Examples showing how to force SCons variable expansion:
```
# Add a directory to the path
env.AppendENVPath('PATH', env.Dir('$MAIN_DIR/project_utils_bin').abspath)

# Set the FOO_OPTIONS shell environment variable
env['ENV']['FOO_OPTIONS'] = env.subst('--build-type=$BUILD_TYPE')
```

## LIBPATH ##
List of directories to search for libraries when linking.

## LIBS ##
List of libraries to link against.

Since all libraries should be in $LIBPATH by default, you should only need to
specify the base name of the library, not its path or extension.  That is,
use 'foo', not '$LIB\_DIR/libfoo.a'.

The env.ComponentProgram() builder will transitively look through the list of
libraries used to link the program when determining which libraries' resources
need to be copied to the output directory.

## LINKFLAGS ##
List of flags to pass to the linker.

## MSVSREBUILDCOM ##
Rebuild command line used by the msvs tool.

However, in Software Construction Toolkit, it's better to use the
component\_targets\_msvs tool than to use SCons's built-in msvs tool.

## OBJSUFFIX ##
Suffix for static object files.

## PDB ##
For Windows, name of the PDB (debug information) file created when linking
with debug enabled.

## RPATH ##
List of directories to search for shared libraries when running programs.

## SHLIBSUFFIX ##
Suffix for a shared library.  On Windows, this is the suffix of the runnable
portion (i.e., '.dll'), not the import library ('.lib').

## SHOBJSUFFIX ##
Suffix for shared object files.


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Other Terms #

## build environment ##

An environment which defines $BUILD\_TYPE and $BUILD\_TYPE\_DESCRIPTION, and is
passed to the BuildEnvironments() call.

Must be cloned from the root environment or one of its descendants.

## component builder ##

One of the builders from the component\_builders tool, or a builder from another
tool which acts like one of those builders (uses the same environment
variables, creates target aliases in a similar way, etc.)

## root environment ##

The environment which includes the component\_setup tool.  All other
environments are cloned from the root environment (either directly, or as a
clone-of-clone).

## target\_platform tool ##

One of the tools starting with 'target\_platform' (for example,
'target\_platform\_windows') which sets up default environment tools and
variables appropriate for that target platform.


---

<a href='Hidden comment: 
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
'></a>

# Deprecated Terms #
The following terms should be replaced by the current names when a SConscript
file is updated.

## Deprecated Global Methods ##

### BuildComponents ###
The old name for BuildEnvironments().

### SConscript ###
In hammer, use env.BuildSConscript() or env.ApplySConscript() instead.

## Deprecated Environment Methods ##

### Copy ###
Use env.Replicate() instead.

Exception: If you are using env.Copy() as part of a multi-step builder, and
modifying the destination file in-place, keep using env.Copy(), since
env.Replicate() may use hard-linking.

### Install ###
Use env.Replicate() instead.

### InstallAs ###
Use env.Replicate() instead.

### Library ###
This SCons environment method still works, but doesn't hook in properly with
the new hammer builders.
Use env.ComponentLibrary() instead.

### Object ###
This SCons environment method still works, but doesn't hook in properly with
the new hammer builders.
Use env.ComponentObject() instead.

### Program ###
This SCons environment method still works, but doesn't hook in properly with
the new component builders.
Use env.ComponentProgram() or env.ComponentTestProgram() instead.

### SharedLibrary ###
This SCons environment method still works, but doesn't hook in properly with
the new component builders.
Use env.ComponentLibrary(COMPONENT\_STATIC=False) instead.

### SharedObject ###
This SCons environment method still works, but doesn't hook in properly with
the new component builders.
Use env.ComponentObject(COMPONENT\_STATIC=False) instead.

### StaticLibrary ###
This SCons environment method still works, but doesn't hook in properly with
the new component builders.
Use env.ComponentLibrary(COMPONENT\_STATIC=True) instead.

### StaticObject ###
This SCons environment method still works, but doesn't hook in properly with
the new component builders.
Use env.ComponentObject(COMPONENT\_STATIC=True) instead.

## Deprecated Environment Variables ##

### # (hash) ###
SCons uses this to mean the directory containing the SConstruct file.  Since
this is evaluated in a different way than other environment variables, using
`#`
instead of $MAIN\_DIR can cause unexpected behavior.
Replaced in Software Construction Toolkit by $MAIN\_DIR.

### BUILD\_COMPONENTS ###
The old name for $BUILD\_SCONSCRIPTS.

### COMPONENT\_LIBRARY\_DIR ###
The old name for $LIB\_DIR.

### MODE ###
The old name for the --mode command line option.

### PLATFORM ###
SCons uses this to refer to the python platform which is running SCons.  It
can be mapped to the host platform, but not tidily.  Use the 'host\_windows',
'host\_mac', and 'host\_linux' component bits instead.

### SCONSTRUCT\_DIR ###
The old name for MAIN\_DIR.