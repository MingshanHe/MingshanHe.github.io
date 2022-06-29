---
title: A CMake-Based Project
date: 2022-06-17 22:35:00
tags: [Tool]
categories: Tool
---

# Introduction

CMake is a versatile tool that helps you build C/C++ projects on just about any platform. It's used by many popular open source projects including LLVM, QT, KDE and Blender. All CMake-based projects contain a script named `CMakeList.txt`, and this post is meant as a guide for configuring and building such projects. This post won't show you *how to write a CMake script* - that's getting ahead of thinks. As an example, I've prepared a CMake-based project named `SOFA` which you can search it in website. It can be build on Windows, Mac or Linux.

# Source and Binary Folders

CMake generates **build pipelines**. A build pipeline might be a Visual Studio `.sln` file, an Xcode `.xcodeproj` or a Unix-style `Makefile`. It can also take several other forms.

To generate a build pipeline, CMake needs to know the **source** and **binary** folders. The source folder is the one containing `CMakeLists.txt`. The binary folder is where CMake  generates the build pipeline. You can create the binary folder anywhere you want. A common practice is to create a subdirectory `build` beneath `CMakeLists.txt`. By keeping the binary folder separate from the source, you can delete the binary folder at any time to get back to a clean slate. You can even create several binary folders, side-by-side, that use different build systems or configuration options.

The **cache** is an important concept. It's a single text file in the binary folder named `CMakeCache.txt` where **cache variables** are stored. Cache variables include user-configurable options defined by the project. You aren't meant to submit the generated build pipeline to source control, as it usually contains paths that are hard coded to the local file system. Instead, simply re-run CMake each time you clone the project to a new folder.

# Configure and Generate Steps

As you'll see in the following sections, there are several ways to run CMake. No matter how you run it, it performs two steps: the **configure** step and the **generate** step. The `CMakeLists.txt` script is executed during the configure step. This script is responsible for defining targets. Each target represents an executable, library, or some other output of the build pipeline. If the configure step succeeds - meaning `CMakeLists.txt` completed without errors - CMake will generate a build pipeline using the targets defined by the script. The type of build pipeline generated depends on the type of **generator** used, as explained in the following sections.

Additional things my happen during the configure step, depending on the contents of `CMakeLists.txt`. For example:

* Finds the header files and libraries.
* Generates a header file in the binary folder, which will be included from C++ code.

In a more sophisticated project, the configure step might also test the availability of system functions (as a traditional Unix **configure** script would), or define a special "install" target (to help create a distributable package). If you re-run CMake on the same binary folder, many of the slow steps are skipped during subsequent runs, thanks to the cache.
