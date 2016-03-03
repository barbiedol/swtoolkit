Examples of how to implement common tasks in .scons files.

**Contents**



---

<a href='Hidden comment:  ------------------------------------------------ '></a>
<a href='Hidden comment:  ------------------------------------------------ '></a>
<a href='Hidden comment:  ------------------------------------------------ '></a>
# Environment setup #

Most of this takes place in main.scons, unless otherwise noted.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## The Bare Minimum Main ##

The following is the smallest reasonable main.scons.
```
# Set up root environment with required component_setup tool and settings
# common to all platforms.
root_env = Environment(
    tools = ['component_setup'],
    BUILD_SCONSCRIPTS = ['hello.scons'],
)

# Derive a platform-specific environment and make this the default.
windows_opt_env = root_env.Clone(
    BUILD_TYPE = 'opt',
    BUILD_TYPE_DESCRIPTION = 'Windows optimized build',
    BUILD_GROUPS = ['default'],
    tools = ['target_platform_windows', 'target_optimized'],
)

# Process the build environments
BuildEnvironments([windows_opt_env])
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Single platform, debug and release builds ##

This extends the previous example to include debug and release builds.
```
# Set up root environment with required component_setup tool and settings
# common to all platforms.
root_env = Environment(
    tools = ['component_setup'],
    BUILD_SCONSCRIPTS = ['hello.scons'],
)

# Set up things common to all Windows builds
windows_env = root_env.Clone(tools = ['target_platform_windows'])

# Windows debug build
windows_dbg_env = windows_env.Clone(
    BUILD_TYPE = 'dbg',
    BUILD_TYPE_DESCRIPTION = 'Windows debug build',
    BUILD_GROUPS = ['default'],
    tools = ['target_debug'],
)

# Windows optimized build
windows_opt_env = windows_env.Clone(
    BUILD_TYPE = 'opt',
    BUILD_TYPE_DESCRIPTION = 'Windows optimized build',
    tools = ['target_optimized'],
])

# Process the build environments
BuildEnvironments([windows_dbg_env, windows_opt_env])
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Multiple platforms, debug and release builds ##

This extends the previous example to include multiple platforms.

Since the number of build environments is starting to grow, it is more
convenient to make a list of build environments and append to it rather than
constructing the list explicitly in the call to BuildEnvironments().
```
# main.scons for hello, world
# Copyright 2008 Google Inc. All Rights Reserved.
# Author: rspangler@google.com (Randall Spangler)

build_environments = []

# Set up root environment with required component_setup tool and settings
# common to all platforms.
root_env = Environment(
    tools = ['component_setup'],
    BUILD_SCONSCRIPTS = ['hello.scons'],
)

#------------------------------------------------------------------------------
# Windows builds
windows_env = root_env.Clone(tools = ['target_platform_windows'])

# Windows debug build
windows_dbg_env = windows_env.Clone(
    BUILD_TYPE = 'dbg',
    BUILD_TYPE_DESCRIPTION = 'Windows debug build',
    BUILD_GROUPS = ['default'],
    tools = ['target_debug'],
)
build_environments.append(windows_dbg_env)

# Windows optimized build
windows_opt_env = windows_env.Clone(
    BUILD_TYPE = 'opt',
    BUILD_TYPE_DESCRIPTION = 'Windows optimized build',
    tools = ['target_optimized'],
)
build_environments.append(windows_opt_env)

#------------------------------------------------------------------------------
# Linux builds
linux_env = root_env.Clone(tools = ['target_platform_linux'])

# Linux debug build
linux_dbg_env = linux_env.Clone(
    BUILD_TYPE = 'dbg',
    BUILD_TYPE_DESCRIPTION = 'Linux debug build',
    BUILD_GROUPS = ['default'],
    tools = ['target_debug'],
)
build_environments.append(linux_dbg_env)

# Linux optimized build
linux_opt_env = linux_env.Clone(
    BUILD_TYPE = 'opt',
    BUILD_TYPE_DESCRIPTION = 'Linux optimized build',
    tools = ['target_optimized'],
)
build_environments.append(linux_opt_env)

#------------------------------------------------------------------------------
# Process the build environments
BuildEnvironments(build_environments)
```
Note that both windows\_dbg\_env and linux\_dbg\_env have $BUILD\_TYPE='dbg' and
are in the default $BUILD\_GROUPS.  This works because BuildEnvironments()
only looks at environments which match the current host platform.  So on
Windows, typing 'hammer' will build windows\_dbg\_env, and on Linux, typing
'hammer' will build linux\_dbg\_env.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Add code coverage build ##

Code coverage is added as a cloned environment for each target platform where
coverage should be run.
```
# Windows coverage variant
environment_list.append(windows_opt_env.Clone(
    BUILD_TYPE = 'opt-coverage',
    BUILD_TYPE_DESCRIPTION='Windows coverage variant',
    tools = ['code_coverage']
))
```
This adds a 'coverage' target which will run all the unit tests and produce a
coverage file $COVERAGE\_OUTPUT\_FILE.  To run coverage in this above example:
```
hammer --mode=opt-coverage coverage
```


---

<a href='Hidden comment:  ------------------------------------------------ '></a>
<a href='Hidden comment:  ------------------------------------------------ '></a>
<a href='Hidden comment:  ------------------------------------------------ '></a>
# Tasks in SConscripts #

<a href='Hidden comment:  ------------------------------------------------ '></a>
## The Bare Minimum Build ##

The following is the smallest reasonable SConscript:
```
# Import calling environment
Import('env')

# Build a program
env.ComponentProgram('hello', 'hello.cpp')
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build a library ##

The following code builds a library which is static or shared, depending on
the default for the target platform.
```
inputs = [
    'a.cc',
    'b.cc',
    'c.cc',
]
env.ComponentLibrary('my_lib', inputs)
```
You can build this library at the command line using 'hammer my\_lib'.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build a static library ##

The following code builds a static library (.a or .lib)
```
inputs = [
    'a.cc',
    'b.cc',
    'c.cc',
]
env.ComponentLibrary('my_static_lib', inputs, COMPONENT_STATIC=True)
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build a shared library ##

The following code builds a shared library (.dylib, .dll, .so)
```
inputs = [
    'a.cc',
    'b.cc',
    'c.cc',
]
env.ComponentLibrary('my_shared_lib', inputs, COMPONENT_STATIC=False)
```
Programs which link against 'my\_shared\_lib' will automatically copy the
resulting shared library to their output directory.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build an output DLL ##

If you are building a library which is not linked by any program (for example,
a browser plugin), use $COMPONENT\_LIBRARY\_PUBLISH to let hammer it should be
treated like a program (that is, it should replicate itself and any resources
and/or libraries it uses to $STAGING\_DIR).  You can also clear
$COMPONENT\_LIBRARY\_LINK\_SUFFIXES so that hammer doesn't attempt to publish the
library to $LIB\_DIR for other programs to link against.
```
env.ComponentLibrary('some_plugin', 'foo.cc', COMPONENT_STATIC=False,
                     COMPONENT_LIBRARY_PUBLISH=True)
```
or
```
env.ComponentLibrary('some_plugin', 'foo.cc', COMPONENT_STATIC=False,
                     COMPONENT_LIBRARY_PUBLISH=True,
                     COMPONENT_LIBRARY_LINK_SUFFIXES=[])
```


<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build a program ##

The following code builds a program (application).
```
inputs = [
    'a.cc',
    'b.cc',
    'c.cc',
]
env.ComponentProgram('my_program', inputs)
```
You can build this library at the command line using 'hammer my\_program'.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Specify different input files based on platform ##

The following example shows building a program where the list of input files
differs based on the platform.
```
# Inputs for all platforms
inputs = [
    'a.cc',
    'b.cc',
    'c.cc',
]

if env.Bit('windows'):
  inputs += ['win_only.cc']

if env.Bit('linux'):
  inputs += ['linux_only.cc']

env.ComponentProgram('my_program', inputs)
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Add windows resources to a target ##

This example shows compiling windows resources into a program.
```
# Inputs for all platforms
inputs = [
    'a.cc',
    'b.cc',
    'c.cc',
]

if env.Bit('windows'):
  inputs += env.RES('my_resources.rc')

env.ComponentProgram('my_program', inputs)
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build a test program ##

This example shows a test for the foo library.
```
# Inputs for all platforms
inputs = [
    'foo_test.cc',
]

env.Append(LIBS=['foo'])
env.ComponentTestProgram('foo_test', inputs)
```
Note that typing 'hammer run\_foo\_test' will build _and_ run the test.

Here's another example which will copy a.jpg to the test directory before
running the test:
```
# Inputs for all platforms
inputs = [
    'foo_test.cc',
]

env.Append(LIBS=['foo'])
env.ComponentTestProgram('foo_test', inputs)
env.Publish('foo_test', 'test_input', 'a.jpg')
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Set the size for a test program ##

Test programs have a size, which affects the timeout for the test as well as
which run-alias it goes into.  Use the COMPONENT\_TEST\_SIZE variable to change
the test size:
```
inputs = [
    'foo_test.cc',
]

env.ComponentTestProgram('foo_test', inputs, COMPONENT_TEST_SIZE='small')
```
foo\_test is now a small test.  It has a test timeout set via
`env['COMPONENT_TEST_TIMEOUT']['small']`, and will be run as part of
`hammer run_small_tests`.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Build a test program with a helper ##

Some tests need another program to assist in the test.  For example, to test
a network library, you may need a helper program which listens on a socket so
the network library has something to connect to.

The helper program should also use env.ComponentTestProgram(), but should set
COMPONENT\_TEST\_RUNNABLE=False so that Hammer won't attempt to run it separately
from the test which requires it.

Use env.Depends() to make SCons compile the helper when the test itself is
compiled.  Note that the test name is actually an alias, so we need to
pass it to env.Alias() before passing it to env.Depends().  (Hammer
automatically creates aliases for components, since it's easier to type
`hammer my_net_test` than `hammer scons-out/dbg/tests/my_net_test.exe`.)

```
# Inputs for all platforms
inputs = [
    'my_net_test.cc',
]

env.Append(LIBS=['my_net'])
env.ComponentTestProgram('my_net_test', inputs)

# Helper program
p = env.ComponentTestProgram('my_net_helper', ['net_helper.cc'],
                             COMPONENT_TEST_RUNNABLE=False)

# Tell SCons that my_net_test needs my_net_helper
env.Depends(env.Alias('my_net_test'), p)
```
Now, 'hammer run\_my\_net\_test' will build both my\_net\_test and my\_net\_helper
before running my\_net\_test.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Run an external unit test ##

This example shows how to run a unit test program which was not built by
hammer - for example, a shell script or external python script.
```
# Running 'precompiled_tests/foo.exe -o foo.dat', creates foo.dat.
test_out = env.Command(['foo.dat'], ['precompiled_tests/foo.exe'],
                       '$SOURCE -o $TARGET')

# Add 'run_foo' as an alias for running that test, which is a medium test.
env.ComponentTestOutput('run_foo', test_out, COMPONENT_TEST_SIZE='medium')
```
Here's another example which runs a python unit test.
```
# Run 'bar.py' and save output to 'bar.out'.
test_out = env.Command(['bar.out'], ['bar.py'], '$PYTHON $SOURCE > $TARGET')

# Add 'run_bar' as an alias for running that test.
env.ComponentTestOutput('run_bar', test_out)
```
Note the use of the $PYTHON variable in this example; Hammer translates that
into the python executable used to run Hammer itself, which is handy if you
have multiple versions of python on your system, or if python isn't in your
shell path.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Disable a test program ##

Sometimes you need to temporarily disable a unit test so it's not run when you
`hammer run_all_tests`, but you still want to make sure it compiles.

This example shows disabling running a test on Windows.  Running
`hammer all_test_programs` will still compile the test, but
on Windows, `hammer run_all_tests` will not run the test.  Running
`hammer run_disabled_tests` will run the test.
```
# Inputs for all platforms
inputs = [
    'foo_test.cc',
]

# Temporarily disable test on Windows; it's flaky there.
if env.Bit('windows'):
  env['COMPONENT_TEST_ENABLED'] = False

env.Append(LIBS=['foo'])
env.ComponentTestProgram('foo_test', inputs)
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Make a new test group ##

By default tests are categorized by size ($COMPONENT\_TEST\_SIZE) and whether
they are enabled ($COMPONENT\_TEST\_ENABLED).  It can also be handy to create
different subgroups of tests (for example, layout tests or smoke tests).

First, set up your test group.  This is usually done in main.scons, since it
only needs to happen once.  Note that we need to add groups to both build and
run the tests.
```
AddTargetGroup('smoke_tests', 'smoke tests can be built')
AddTargetGroup('run_smoke_tests', 'smoke tests can be run')
```
This also tells hammer's help system about these groups, so that `hammer -h`
will print your test groups if they're not empty.

Then for each test which should belong to the new group, add it to the
appropriate groups:
```
inputs = [
    'foo_test.cc',
]

env.Append(
    COMPONENT_TEST_PROGRAM_GROUPS=['smoke_tests'],
    COMPONENT_TEST_OUTPUT_GROUPS=['run_smoke_tests'],
)

env.ComponentTestProgram('foo_test', inputs)
```
Note that the effects of `env['COMPONENT_TEST_ENABLED'] = False` take effect
before adding the test to its output groups, so if you disable a test, it will
also disappear from your test group.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Include source from somewhere outside your project ##

When you compile a source file inside your project, the object files goes in
a matching location in $OBJ\_ROOT.  For example,  If your build.scons file in
your main project directory does:
```
env.ComponentLibrary('foo', ['foo.cc', 'bar/baz.cc'])
```
then the source files are $MAIN\_DIR/foo.cc and $MAIN\_DIR/bar/baz.cc, and the
object files will be $OBJ\_ROOT/foo.o and $OBJ\_ROOT/bar/baz.o.

But if the source file is outside your project (that is, not undernearth
$MAIN\_DIR), then SCons refers to it with an absolute path, and the object file
will end up in the same directory as the source file.  For example,
```
env.ComponentLibrary('foo', ['../other/a.cc'])
```
will end up creating an object file ../other/a.o.

That's not so good for several reasons:
  * It pollutes your source tree with build output.
  * If you build multiple modes at once ('hammer --mode=all'), then SCons has two ways to build ../other/a.o - one for debug, one for release.  This makes SCons unhappy.

The way around this is to use SCons's addRepository() function to map the other
directory down inside your project.  For example:
```
env.Dir('third_party/other').addRepository(env.Dir('../other'))

env.ComponentLibrary('foo', ['third_party/other/a.cc'])
```
This will make an object file $OBJ\_ROOT/third\_party/other/a.o.

Note that the effect of addRepository() is global in scope, so if you're
referencing code from the repository in multiple build.scons files, it's better
to put the addRepository() call in your main.scons.  It's also often handy to
define a variable for repository destination, particularly if you also need to
add it to the include paths.
```
env['OTHER_DIR'] = 'third_party/other'
env.Dir('$OTHER_DIR').addRepository(env.Dir('../other'))

env.Append(CPPPATH=['$OTHER_DIR']

env.ComponentLibrary('foo', ['$OTHER_DIR/a.cc'])
```

Note also that addRepository() only maps files it can't find in your project
dir.  So if you have:
```
    your_project_dir/
      main.scons
      foo/
        foo.scons
        a.cc
    third_party/
      foo/
        a.cc
        b.cc
        c.cc
```
and you do
```
env.Dir('foo').addRepository(env.Dir('../third__party/foo'))
```
then your\_project\_dir/foo/foo.scons can act like it has a.cc, b.cc, and c.cc
all in its directory.  Since a.cc is present in your\_project\_dir/foo/, this
will be used instead of the a.cc in third\_party/foo/.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Include a single source file from somewhere outside your project ##

Another way of handling source files from outside your project is to
compile them using ComponentObject() and explicitly specify the name of
the output object file.  Make sure that if you specify options to
ComponentLibrary() such as COMPONENT\_STATIC=False, that you specify the same
options to ComponentObject().
```
# Explicitly specify the object file
obj = env.ComponentObject('a_from_other', ['../other/a.cc'])

# Now link the object file into the real target
env.ComponentLibrary('foo', [obj])
```
Because of the needs to explicitly name the object file and keep the options
to ComponentObject() and ComponentLibrary() the same, this isn't as clean a
solution as using addRepository() (see the previous example).

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Use a source file in more than one target ##

Sometimes it's necessary to compile the same source file into multiple targets.
For example, you may have a source that you want to compile into a unit test
for that source, but you can't link against the entire library containing that
source, or it's a shared library which doesn't export the symbols you need to
link against.

If you try doing this the simple way:
```
env.ComponentLibrary('foo', ['a.c', 'foo.c'])
env.ComponentLibrary('bar', ['a.c', 'bar.c'])
```
SCons will complain that there are multiple ways of building the object file
(a.obj, a.o, or a.so, depending on your platform).

There are a few ways of dealing with this.

1. Compile the shared file(s) to object(s) first.  This is easiest if both
consumers of the object file are in the same (or closely affiliated)
SConscripts, and compile the source with the same command line options.
```
# Since the same options need to be passed to both the ComponentObject() call
# and the ComponentLibrary() calls, it's tidier to set them in the environment.
env['COMPONENT_STATIC'] = False

# Compile the file into an object
a_obj = env.ComponentObject('a.c')
env.ComponentLibrary('foo', [a_obj, 'foo.c'])
env.ComponentLibrary('bar', [a_obj, 'bar.c'])
```
This also works properly if ComponentObject() is passed a list of sources - in
that case, a\_obj will be a list of the corresponding objects.

1a. Another way of passing the object files to sub-environments using
environment variables.  This is handy if the shared files are high up in the
tree and used by numerous leaves in the tree.

In some parent SConscript:
```
env['A_SHARED_OBJ'] = env.ComponentObject('a.c')
```

In some other SConscript(s):
```
env.ComponentLibrary('foo', ['foo.c', '$A_SHARED_OBJ'])
env.ComponentLibrary('bar', ['bar.c', '$A_SHARED_OBJ'])
```

2. Change $SHOBJSUFFIX (or $OBJSUFFIX, if compiling static) for one of the
targets.
```
env.ComponentLibrary('foo', ['a.c', 'foo.c'])

env2 = env.Clone()
env2.Append(CPPDEFINES=['SOMEFLAG'])
env2.ComponentLibrary('bar', ['a.c', 'bar.c'],
                      OBJSUFFIX='_bar' + env['OBJSUFFIX'],
                      SHOBJSUFFIX='_bar' + env['SHOBJSUFFIX'])
```
This will cause the source file(s) to be compiled twice, but with different
object file names.  This is necessary if the targets have different compiler
options.

3. Compile the common files as a static library, and link the other libs
against it.
```
env.ComponentLibrary('foobar_shared', 'a.c', COMPONENT_STATIC=True)
env.Append(LIBS=['foobar_shared'])
env.ComponentLibrary('foo', 'foo.c', COMPONENT_STATIC=False)
env.ComponentLibrary('bar', 'bar.c', COMPONENT_STATIC=False)
```
This is convenient if the number of shared sources and/or the number of
components using them are large, and works well if the users are in very
different parts of the source tree.  It's also a great opportunity to write
unit tests for foobar\_shared, since those files apparently represent a distinct
shared interface.  However, if any of the targets which use the shared file are
themselves static libraries, you need to remember to link the users of those
static libraries with foobar\_shared as well.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Make a build step depend on a component ##

Components built by Software Construction toolkit component builders
(env.ComponentLibrary(), env.ComponentProgram(), etc.) automatically set up
aliases to refer to their outputs.

If you need to express a dependency on one of those components, you can do it
one of two ways.

If the thing which depends on the component is in the same build.scons as the
component builder call, you can directly use the list of output nodes returned
by the builder call:
```
p = env.ComponentProgram('foo', foo_inputs)

# Set up to run bar.bat
cmdout = env.Command(['bar.out'], ['bar.sh'], '$SOURCE > $TARGET')

# If bar.sh calls foo, we need to tell SCons that foo should be built first
env.Depends(cmdout, p)
```

However, suppose that foo is built in some other build.scons file in some other
directory.  To refer to foo's outputs, we can use the shortcut alias:
```
env.Depends(cmdout, env.Alias('foo'))
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Compile one source file with different options than the others ##

Sometimes you need to compile one source file for a target with different
options than the other files.  For example, that file may not work well with
optimization enabled.  Use the ComponentObject() builder to compile that file
into an object file, then include the object file with the rest of the source.

```
inputs = ['a.c', 'b.c', 'foo.c']

# bar.c doesn't compile with -O2, so use -O1
envbar = env.Clone()
if '-O2' in envbar.SubstList2('$CCFLAGS'):
  envbar.FilterOut(CCFLAGS=['-O2'])
  envbar.Append(CCFLAGS=['-O1'])
inputs += envbar.ComponentObject('bar.c')

env.ComponentLibrary('foo', inputs)
```
Note that we used SubstList2() to evaluate CCFLAGS, since it properly flattens
sublists of options and evaluates text variables inside options.

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Enable exceptions for some source files ##

Compiling most files with exceptions disabled and some with exceptions enabled
was a common enough pattern in our projects we have a variable which changes
the compiler and linker flags.

To disable exceptions by default, put in your main.scons:
```
# For your windows environment
windows_env.Append(CPPDEFINES = ['_HAS_EXCEPTIONS=0'])

# For your posix environments (assuming mac and linux environments derive from
# this common posix ancestor; if not, put in each environment).
posix_env.Append(CCFLAGS = ['-fno-exceptions'])
```

Then in the SConscript where you need to compile a file with exceptions:
```
inputs = ['a.cc', 'b.cc', 'foo.cc']

# bar.cpp needs to compile with exceptions
inputs += env.ComponentObject('bar.cc', ENABLE_EXCEPTIONS=True)

env.ComponentLibrary('foo', inputs)
```
(Of course, if you needed to enable exceptions for the entire library, you'd
just set `env['ENABLE_EXCEPTIONS'] = True` for the entire file)

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Write a string to a file ##

If you need to write a string to a file as part of your build, you may be
tempted to use python to write it directly, as part of your SConscript.  Doing
this works, but has two nasty side-effects:
  * SCons doesn't know about the file, so may not properly rebuild it.
  * Even if you run hammer with the `-n` option to 'do nothing', your build will still write the file.

One way around this is to use the ReplaceString() builder to modify a template
file.

Another way is to move the code which does the writing into a subroutine, and
call it via env.Command():
```
def WriteLog(target, source, env):
  f = open(str(target[0]), "w")
  f.write(env['write_data'])
  f.close()
  return 0

# Write 'thing to write' to the foo file in the staging directory
env.Command('$STAGING_DIR/foo', [], WriteLog, write_data='thing to write')
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
## Refer to a program in the staging directory ##

Normally when you create a program via ComponentProgram(), it's compiled in a subdir of $OBJ\_ROOT, and then published (linked) into $STAGING\_DIR.

To refer to the program file link in the staging dir, use the following:
```
env.ComponentProgram('foo', foo_inputs)

foo_in_staging = env.File('$STAGING_DIR/${PROGPREFIX}foo${PROGSUFFIX}')
```


---

<a href='Hidden comment:  ------------------------------------------------ '></a>
<a href='Hidden comment:  ------------------------------------------------ '></a>
<a href='Hidden comment:  ------------------------------------------------ '></a>
# Examples Coming Soon #

We know we need examples for the following:
  * Mac app bundle
  * Browser plugin (mac / activeX / npapi / moz)
  * Code coverage
  * Visual Studio solution + projects
  * Make a variant of a project (when to make a new mode vs. env.SetBitFromOption())
  * Chain to a sub-build-script, such as an IDL compile script
  * Chain to a sub-build which is also in hammer
  * Call a custom python function to do something (show how to make a tool in your project's site\_scons)
  * Checked-in third-party library (needs to env.Publish() itself for running, ala third\_party.scons)