# CMake-codecov

[![Travis](https://img.shields.io/travis/RWTH-ELP/CMake-codecov/master.svg?style=flat-square)](https://travis-ci.org/RWTH-ELP/CMake-codecov) [![Codecov](https://img.shields.io/codecov/c/github/RWTH-ELP/CMake-codecov.svg?style=flat-square)](https://codecov.io/github/RWTH-ELP/CMake-codecov?branch=master)  [![](https://img.shields.io/github/issues-raw/RWTH-ELP/CMake-codecov.svg?style=flat-square)](https://github.com/RWTH-ELP/CMake-codecov/issues)
[![BSD (3-clause)](http://img.shields.io/badge/license-3--clause_BSD-blue.svg?style=flat-square)](LICENSE)

CMake module to enable code coverage easily and generate coverage reports with CMake targets.



## Include into your project

To use [Findcodecov.cmake](cmake/Findcodecov.cmake), simply add this repository as git submodule into your own repository
```Shell
mkdir externals
git submodule add git://github.com/RWTH-ELP/CMake-codecov.git externals/CMake-codecov
```
and adding ```externals/cmake-codecov/cmake``` to your ```CMAKE_MODULE_PATH```
```CMake
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/externals/cmake-codecov/cmake" ${CMAKE_MODULE_PATH})
```

If you don't use git or dislike submodules you can copy the [Findcodecov.cmake](cmake/Findcodecov.cmake), [FindGcov.cmake](cmake/FindGcov.cmake) and [FindLcov.cmake](cmake/FindLcov.cmake) files into your repository. *Be careful when there are version updates of this repository!*

Next you have to include the codecov package. This can be done in your root CMake file, or in the file were your targets will be defined:
```CMake
# enable code coverage
find_package(codecov)
```

For coverage evaluation you have to add ```coverage_evaluate()``` after all other targets have been defined. A good place for this is your root CMakeLists.txt in the last lines after you included your sub-directories and added targets.


## Usage

To enable coverage support in general, you have to enable ```ENABLE_COVERAGE``` option in your CMake configuration. You can do this by passing ```-DENABLE_COVERAGE=On``` on your command line or with your graphical interface.

If coverage is supported by your compiler, the specified targets will be build with coverage support. If your compiler has no coverage capabilities (I asume intel compiler doesn't) you'll get a warning but CMake will continue processing and coverage will simply just be ignored.

#### Compiler issues

Different compilers may be using different implementations for code coverage. If you'll try to cover targets with C and Fortran code but don't use gcc & gfortran but clang & gfortran, this will cause linking problems. To avoid this, such problems will be detected and coverage will be disabled for such targets.

Even C only targets may cause problems, if e.g. clang compiles the coverage for an older gcov version than the one is shipped with your distribution. [FindGcov.cmake](cmake/FindGcov.cmake) tries to find a compatible coverage evaluation tool to avoid this issue, but may fail. In this case you should check coverage with a different compiler or install a compatible coverage tool.

### Build targets with coverage support

To enable coverage support you have two options: You can mark targets explictly for coverage by adding your target with ```add_coverage()```. This call must be done in the same directory as your ```add_executable()```or ```add_library()``` call:
```CMake
add_executable(some_exe foo.c bar.c)
add_coverage(some_exe)

add_library(some_lib foo.c bar.c)
add_coverage(some_lib)
```

The second option is to enable ```ENABLE_COVERAGE_ALL``` option, which will enable coverage for **all** targets. You can do this by passing ```-ENABLE_COVERAGE_ALL=On``` on your command line or with your graphical interface.


### Executing your program

To be able to evaluate your coverage data, you have to run your application first. Some projects include CMake tests - it might me a good idea to execute them now by ```make test```, but you can run your application however you want (e.g. by running ```./a.out```).


### Evaluating coverage data

#### Gcov

Gcov is a console program to evaluate the generated coverage data. You can evaluate the data by calling the following targets:

| target  | description |
|---------|-------------|
|```<TARGET>-gcov```|Evaluate coverage data for target ```<TARGET>```.|
|```gcov```|Evaluate the coverage data of all your targets. **Warning:** You have to run programs generated by every target before you can call this target without any error. Otherwise you might get errors, if ```*.gcda``` files will not be found.|

The files generated by Gcov reside in the binary directory of the target ```<TARGET>``` you're evaluating the coverage data for (e.g. for target ```bar``` in this repository it'll be ```${CMAKE_BINARY_DIR}/src/bar/CMakeFiles/bar.dir/```).


#### Lcov

Lcov is a console program to evaluate the generate coverage data, but instead of writing the results into a copy of the original source file (like Gcov does) a HTML report will be generated, which is much easier to read than several gcov files.

| target  | description |
|---------|-------------|
|```<TARGET>-geninfo```|Evaluate coverage data for target ```<TARGET>```.|
|```<TARGET>-genhtml```|Generate a report for a specific target *(and only this one, even if it has dependencies!)*. This target will call ```<TARGET>-geninfo``` before. Reports will be generated in ```${CMAKE_BINARY_DIR}/lcov/html/<TARGET>```.|
|```lcov-geninfo```|Evaluate the coverage data of all your targets.|
|```lcov-genhtml```|Generate a *single* report for all evaluated data that is available now. **Note:** You have to call ```<TARGET>-geninfo``` for all targets you want to have in this report before calling this target or ```lcov-geninfo```. You can use this option, if you like to have a single report for the targets ```foo``` and ```bar``` together, but without all the other targets. Reports will be generated in ```${CMAKE_BINARY_DIR}/lcov/html/selected_targets```.|
|```lcov```|Generate a *single* report for all targets. This target will call ```lcov-geninfo``` before. Reports will be generated in ```${CMAKE_BINARY_DIR}/lcov/html/all_targets```.|

##### Excluding files from coverage reports
If you want to exclude some files from your coverage reports by ```lcov --remove``` subcommand, you can append their path patterns to ```LCOV_REMOVE_PATTERNS``` in your CMakeLists.txt like a following example.
```CMake
list(APPEND LCOV_REMOVE_PATTERNS "'${PROJECT_SOURCE_DIR}/extlib/*'")
```
Note that asterisks in patterns should not be expanded by the shell interpreter.


## Contribute

Anyone is welcome to contribute. Simply fork this repository, make your changes **in an own branch** and create a pull-request for your change. Please do only one change per pull-request.

You found a bug? Please fill out an issue and include any data to reproduce the bug.

#### Contributors

[Alexander Haase](https://github.com/alehaa)


## License

CMake-codecov is released under the 3-clause BSD license. See the [LICENSE](LICENSE) file for more information.

Copyright &copy; 2015-2016 RWTH Aachen University, Federal Republic of Germany.
