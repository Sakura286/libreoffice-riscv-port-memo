
# Debug UITest

This article focus on debugging UITest, which includes debugging python components and pyuno.

## Debugging python

Add `import pdb;` to the python file you want to debug, and add `pdb.set_trace()` to the line that you want to set breakpoint.

## Run UITest

One way to run a specific UITest is

```shell
export SAL_USE_VCLPLUGIN=gen
# change directory into sw is not essential, but it will apparently shorten your
# time when you are testing with a slow cpu.
cd sw
make -srj1 UITest_<module_name> UITEST_TEST_NAME="<py_file>.<class_name>.<def_name>"
```

The other way is to directly run a specific UITest python file. This is way faster and you can test LibreOffice with your hand-written python file.

```shell
export SRCDIR=<path_to_libreoffice_source_code>
export PYTHONPATH=$SRCDIR/instdir/program
export PYTHONPATH=$PYTHONPATH:$SRCDIR/unotest/source/python
export URE_BOOTSTRAP=file://$SRCDIR/instdir/program/fundamentalrc
export SAL_USE_VCLPLUGIN=gen
# The external file opened by LibreOffice during testing is included in TDOC
export TDOC=<data_path>
export TestUserDir=file:///tmp 
export LC_ALL=C

rm -rf /tmp/libreoffice/4

python3 "$SRCDIR/uitest/test_main.py" --soffice=path:"$SRCDIR/instdir/program/soffice" --userdir=file:///tmp/libreoffice/4 --file=<path_of_python_file>
```

## UITest low-level debugging(caller side)

If we need a further debug, such as inspecting a low-level c/c++ function and reading registers, GDB is nessassary, especially when UNO bridge works unproperly.

We need to grasp the architecture first:

```plain
     UITest                        remote
 (caller side)                   callee side
+--------------+              +----------------+
|    Python    |              |                |
| +----------+ |  UNO bridge  |                |
| |PyUNO(cpp)+-+--------------+->cpp function  |
| +----------+ |              |                |
|              |              |                |
+--------------+              +----------------+
```

On caller side, add `pdb.set_trace()` to the line that you want to break in python file. Then run `gdb python3` to get python3 debugged by GDB. PDB and GDB will work together.

## UITest low-level debugging(callee side)

The easiest way is to run the following command directly before you start debugging UITest side.

```shell
make debugrun
```

This will create a GDB debugging session attached to a new blank LibreOffice instance which is listening to UITest call. When you run UITest, you could see the progress of the test in the window.

But the performance is awful on slow platform. A wise choice is run UITest first. Let UITest raise LibreOffice instance without gdb attached, then run gdb and attach to soffice process.

On caller side, run UITest first.

```shell
...
# Run UITest first
python3 ...
```

On callee side (in a new terminal session), after the libreoffice splash screen appeared.

```shell
# Get the pid of LibreOffice
ps aux | grep soffice.bin
gdb
(gdb) attach <the_pid_of_libreoffice>
```

## References

1. [Debugging Python components in LibreOffice - The Document Foundation Wiki](https://wiki.documentfoundation.org/Development/How_to_debug#Debugging_Python_components_in_LibreOffice)
2. [Development/UITests - The Document Foundation Wiki](https://wiki.documentfoundation.org/Development/UITests)
3. [libreoffice-riscv64-performance-testing - Sakura286 - GitHub](https://github.com/Sakura286/libreoffice-riscv64-performance-testing)