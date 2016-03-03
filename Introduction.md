

# Execution Requirements #

Software Construction Toolkit requires:

  * Python 2.4 or later.
    * For example, from http://www.python.org
    * If on Windows, also need the pywin32 extensions (http://sourceforge.net/projects/pywin32/)
    * Software Construction Toolkit is not yet compatible with Python 3.

  * SCons 1.2 or later.
    * For example, from http://www.scons.org/download.php
    * A scons or scons-local install is required for building projects.
    * A scons-src install is required for developing Software Construction Toolkit.

There should be no other direct dependencies or requirements to run the
Software Construction Toolkit.


# Installation #

Installation of this package is as simple as copying its directory tree to any
directory.

If you are using the hammer.bat or hammer.sh entry point, you also need to
set up a SCONS\_DIR environment variable to point to the directory containing
SCons.
  * For a normal SCons install, this is the `engine` subdir.
  * For a scons-local install, this is the `.` subdir.
  * For a scons-src install, this is the `src/engine` subdir.
  * For a Windows install of SCons which used a .exe or .msi installer, SCons may be installed in a subdirectory of the Python directory.  For example, `C:\Python24\Lib\site-packages\scons-1.2.0`.

If you are modifying Software Construction Toolkit and need to run the unit
tests, set up a SCONS\_DEV\_DIR environment variable to point to the directory
containing the scons-src install.

Once you have installed this package, you should write a main.scons file at the
top level of your source tree to build your software.

Then modify the build/install instructions for your package to instruct
your users to execute Software Construction Toolkit by running hammer.bat or
hammer.sh.


# Contents of This Package #

This package consists of the following:

| **File** | **Description** |
|:---------|:----------------|
| COPYING  | A copy of the copyright and terms under which Software Construction Toolkit is distributed (the BSD license). |
| README.software\_construction\_toolkit | Readme for the Software Construction Toolkit |
| hammer.bat | The entry point for Software Construction Toolkit on Windows. |
| hammer.sh | The entry point for Software Construction Toolkit on posix and unix-like systems, including Cygwin. |
| history.txt | Change history for releases. |
| wrapper.py | A script called by hammer which does some additional setup and then calls SCons. |
| site\_scons/ | Tools and modules for Software Construction Toolkit. |
| samples/ | Sample projects which demonstrate usage of the Software Construction Toolkit. |
| bin/     | Additional executables and utilities for developing the Software Construction Toolkit, including the test-runner. |
| lib/     | Additional libraries for developing the Software Construction Toolkit, including the test framework module. |
| test/    | Tests for Software Construction Toolkit. |


# Licensing #

Software Construction Toolkit is distributed under the BSD license, a full copy
of which is available in the COPYING file in this package.

# Reporting Bugs #

You can report SCons by following the Issues link above:

> http://code.google.com/p/swtoolkit/issues/list


# Mailing Lists #

A mailing list for users of Software Construction Toolkit is available.  You
may send questions or comments to the list at:

> swtoolkit@googlegroups.com

You may subscribe to the mailing list at:

> http://groups.google.com/group/swtoolkit


# Author Info #

Randall Spangler (randall dot spangler at gmail dot com)

With plenty of help from the Software Construction Toolkit Development team:
  * Brad Nelson
  * Steven Knight
  * Stephen Ng
  * Greg Spencer