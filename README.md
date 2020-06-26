# README #

A CMakeLists.txt for developing DLLs in C++ that work with Racket.

### What is this repository for? ###

This repository hosts a CMake file to setup new projects with the following directory structure:
```
project
│   CMakeLists.txt 
│
└───build   
└───project-ffi
    │   info.rkt
    │   main.rkt
    └───private (optional)
    └───lib (installation target directory for the generated shared lib.)
└───src
    │   CMakeLists.txt
    │   project.h
    │   project.cpp
```

`project-ffi` is created by running `raco pkg new <project>` as defined inside the `CMakeLists.txt` file

**Once files are created, they are not modified.**

### How do I get set up? ###

* Install [CMAKE](https://cmake.org/).
* Install Racket (https://racket-lang.org/) and configure your path so that raco runs on the command line,
* Download CMakeLists.txt from this repo and place in a folder.
* Edit the project name `project(<name>)` in the file.
* Create a `build` subdirectory and run `cmake-gui ..`

There are some configurable options:

| Option                    | Description                                         |
|--------------------------:|:----------------------------------------------------|
|`RKT_SEPARATE_DOC_PACKAGE` | Add a second package for documentation              |
|`RKT_ADD_PRIVATE_SUBDIR`   | Add a `private` directory inside the Racket project |