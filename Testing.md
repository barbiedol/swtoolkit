**Contents**



---

<a href='Hidden comment:  ------------------------------------------------ '></a>
# Introduction #

The testing infrastructure for the Software Construction Toolkit is built on
the testing infrastructure for SCons.  Since build tools are typically
intertwined with their environment and external programs, the SCons
infrastructure centers on end-to-end functional tests.  We have extended
the SCons infrastructure to enable both small tests and test coverage.

Tests are located in the tests/ subdirectory.

<a href='Hidden comment:  ------------------------------------------------ '></a>
# Requirements #

Writing and running Software Construction tests requires the following:
  * Python 2.4 or later. It is not yet compatible with Python 3.
  * A scons-src install of SCons 1.2 or later.

Set the following variables:
  * SCONS\_DEV\_DIR should point to the directory containing the scons-src install.

<a href='Hidden comment:  ------------------------------------------------ '></a>
# How to run tests #

To run one test:
```
bin/runtest.py test/<testname>.py
```

To run all tests:
```
bin/runtest.py -a
```

By default, tests don't print much information.  Run with `--verbose=3` to
dump more information:
```
bin/runtest.py --verbose=3 test/<testname>.py
```

Since tests clean up after themselves, you can't normally see what files were
being generated.  To stop runtest from doing this, add `PRESERVE=1` to your
shell environment before running the tests:
```
export PRESERVE=1
bin/runtest.py test/<testname>.py
```
or on Windows:
```
set PRESERVE=1
bin\runtest.bat test/<testname>.py
```
The test framework will then leave its temporary files, and will tell you
where it put them.

<a href='Hidden comment:  ------------------------------------------------ '></a>
# How to add a test #

Here is the smallest functional test, broken down into sections:

Standard python header, including docstring.
```
#!/usr/bin/python2.4

"""your_test_description"""
```

All tests must import TestFramework.
```
import TestFramework
```

The SConscript file for your test is implemented in the test as a function.  SCons
globals that you need should be copied from the scons\_globals dict.
```
def TestSConstruct(scons_globals):
  """Test SConstruct file.

  Args:
    scons_globals: Global variables dict from the SConscript file.
  """
  # Get globals from SCons
  Environment = scons_globals['Environment']

  # Construct the root environment
  root_env = Environment(
      tools=['component_setup'],
  )

  # The rest of your test would go here
```

The main routine does the following:
  * Construct a test object.
  * Set up the test directory.
  * Create the SConstruct file (SCons's name for main.scons).
  * Optional: Create input files and/or directories necessary to run the test, for example, source files.  Use the following functions:
    * test.write(filename, contents\_string).
    * test.subdir(dirname)
  * Run the test.  This invokes SCons on the SConstruct file, which then calls back into your test's SConstruct function.
    * Optional: test.run() may also check the contents of stderr, stdout, or the exit code from running the test.
  * Optional: Check output files using the following functions:
    * test.must\_exist(filename)
    * test.must\_match(filename, expected\_contents\_string).
  * Explicitly pass the test.  If this is not done, SCons's test framework records the test as NO RESULT.
```
def main():
  test = TestFramework.TestFramework()
  test.subdir('your_test_name')
  base = 'your_test_name/'
  test.WriteSConscript(base + 'SConstruct', TestSConstruct)
  test.run(chdir=base)
  test.pass_test()
```

Python boilerplate to run main().
```
if __name__ == '__main__':
  main()
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
# Test size definitions #

Tests are classified by size:
  * Small test: Unit tests of modules / methods / classes (see below).
  * Medium test: Tests which are end-to-end within SCons, or build a simple "hello, world" application.
  * Large test: Tests which build an entire application.  Also called functional or end-to-end tests.
  * Smoke test: Tests run before each check-in.  Currently, this is all the small tests (`bin/runtest -a`).

<a href='Hidden comment:  ------------------------------------------------ '></a>
# Example test #
This is the previous example, combined for easier copy-pasting to get you
started.
```
#!/usr/bin/python2.4

"""your_test_description"""

import TestFramework


def TestSConstruct(scons_globals):
  """Test SConstruct file.

  Args:
    scons_globals: Global variables dict from the SConscript file.
  """
  # Get globals from SCons
  Environment = scons_globals['Environment']

  # Construct the root environment
  root_env = Environment(
      tools=['component_setup'],
      TOOL_ROOT=TestFramework.ToolRoot(),
  )

  # The rest of your test would go here


def main():
  test = TestFramework.TestFramework()
  test.subdir('your_test_name')
  base = 'your_test_name/'
  test.WriteSConscript(base + 'SConstruct', TestSConstruct)
  test.run(chdir=base)
  test.pass_test()
```

<a href='Hidden comment:  ------------------------------------------------ '></a>
# Using unittest to write small tests #
You can also import the unittest module and write small tests inside your
TestSConstruct function.

```
#!/usr/bin/python2.4

"""Test for defer."""

import unittest
import SCons.Errors
import TestFramework


class DeferTests(unittest.TestCase):
  """Tests for defer module."""

  def setUp(self):
    """Per-test setup."""
    self.call_list = []
    self.env = self.root_env.Clone()

  def testDeferReentrancy(self):
    """Test re-entrant calls to ExecuteDefer()."""

    def Sub1(env):
      env.ExecuteDefer()

    env = self.env
    env.Defer(Sub1)
    self.assertRaises(SCons.Errors.UserError, env.ExecuteDefer)


def TestSConstruct(scons_globals):
  """Test SConstruct file.

  Args:
    scons_globals: Global variables dict from the SConscript file.
  """

  # Get globals from SCons
  Environment = scons_globals['Environment']
  env = Environment(tools=['environment_tools', 'defer'])

  # Run unit tests
  TestFramework.RunUnitTests(DeferTests, root_env=env)


def main():
  test = TestFramework.TestFramework()
  test.subdir('defer')
  base = 'defer/'
  test.WriteSConscript(base + 'SConstruct', TestSConstruct)
  test.run(chdir=base, stderr=None)
  test.pass_test()


if __name__ == '__main__':
  main()
```