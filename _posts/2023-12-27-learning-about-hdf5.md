---
title: "Learning About HDF5 💽"
date: 2023-12-27
---

## What is HDF5 and why is it Helpful

To provide a definition from Wikipedia, the Hierarchical Data Format (HDF) comprises a set of file formats (HDF4, HDF5) designed for storing and organizing large amounts of data. HDF is a file format used when there is a need to store massive volumes of data. The primary advantage of using HDF5 is its ability to leverage parallel file systems commonly employed in High-Performance Computing (HPC) systems.

In traditional file systems, such as ext4, accessing files requires the requesting process to obtain a lock on the file, preventing other processes from concurrently accessing it. However, when employing HDF5, multiple processes can read and write to a file in parallel. HDF5 also features collective I/O, a method in which multiple processes collaborate to efficiently read and write data to a shared file system.

While the HDF5 library is written in C function wrappers have been written to support other languages such as FORTAN 90, C++, and Java.

## Today's Reading

So today I plan on reading through the, ["Learning the Basics"](https://docs.hdfgroup.org/hdf5/develop/_learn_basics.html) as I have never worked with the file format myself. Since my future research involves HPC I/O, I have a feeling I should understand HDF5 very well. 

## HDF5 File

HDF uses the concept of groups and datasets in place of files. Both groups and datasets may have associated attributed lists which describe the content. Using goups and datasets results in the similar, "file path" concept that appears in traditional file systems. For instance `/foo/var` could refer to a var dataset that is in the foo group which itself is in the root group.

## Using HDF5

In traditional file systems, the programmer must read and write through abstract APIs, such as POSIX or STDIO, to interact with the underlying file systems, such as ext4. HDF5 differs in that it exposes its own user-level API, in which the implementation will drop down to a lower-level API, such as POSIX, STDIO, or MPI-IO, which the programmer can configure. The user-level API uses an  organized naming system where function signatures begin with *H5\** where *\** is one or two uppercase letters indicating the type of object on which the function operates.

## Compiling HDF5

So the HDF5 comes with both serial and parallel version one for serial file systems and one for parallel file systems respectively. If your on Debian/Ubuntu you should be able to get an HDF5 program to compile using gcc with:

```
sudo apt install libhdf5-dev
gcc -o your_program example.c -lhdf5 -L/usr/lib/x86_64-linux-gnu/hdf5/serial -I/usr/include/hdf5/serial
```

One really helpful command I found from [here](https://stackoverflow.com/questions/43151312/how-to-compile-c-program-with-hdf5-source-code) `h5cc` shows the required parameters to compile your code. Here's the output I get:

```
gcc -I/usr/include/hdf5/serial -L/usr/lib/x86_64-linux-gnu/hdf5/serial /usr/lib/x86_64-linux-gnu/hdf5/serial/libhdf5_hl.a 
/usr/lib/x86_64-linux-gnu/hdf5/serial/libhdf5.a -lcrypto -lcurl -lpthread -lsz -lz -ldl -lm
```

As you can see we are using the serial variant of the library because the underlying filesystem is ext4.

## TO BE CONTINUED!