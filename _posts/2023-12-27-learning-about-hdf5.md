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

In traditional file systems, the programmer must read and write through abstract APIs, such as POSIX or STDIO, to interact with the underlying file systems, such as ext4. HDF5 differs in that it exposes its own user-level API, in which the implementation will drop down to a lower-level API, such as POSIX, STDIO, or MPI-IO, which the programmer can configure. The user-level API uses an  organized naming system where function signatures begin with **H5\*** where **\*** is one or two uppercase letters indicating the type of object on which the function operates.

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

One really handy package to install is `hdf5-tools`. This gives additional tools to inspect HDF5 files making it easier to write and debug your programs.

## Open/Close Files

### New Functions Used

- [H5Fcreate](https://docs.hdfgroup.org/hdf5/develop/group___h5_f.html#gae64b51ee9ac0781bc4ccc599d98387f4)
- [H5Fclose](https://docs.hdfgroup.org/hdf5/develop/group___j_h5_f.html#gae2af8e1b3fcf6a3a98ab345ed3ff710f)

```
#define FILE_NAME "file.h5"
hid_t file_id = H5Fcreate(FILE_NAME, H5F_ACC_TRUNC, H5P_DEFAULT, H5P_DEFAULT);
herr_t status = H5Fclose(file_id);
```

You could then check the status for errors.

After running the program you should get a file named `file.h5` in the directory the program was run. Running `h5dump file.h5` outputs:

```
HDF5 "file.h5" {
GROUP "/" {
}
}
```

This shows that we have an HDF5 file named **file.5** which contains the root `/` group.

## Dataset Creation

### New Functions Used

- [H5Fopen](https://docs.hdfgroup.org/hdf5/develop/group___h5_f.html#gaa3f4f877b9bb591f3880423ed2bf44bc)
- [H5Screate_simple](https://docs.hdfgroup.org/hdf5/develop/group___h5_s.html#ga8e35eea5738b4805856eac7d595254ae)
- [H5Dcreate2](https://docs.hdfgroup.org/hdf5/develop/group___h5_d.html#gabf62045119f4e9c512d87d77f2f992df)
- [H5Dclose](https://docs.hdfgroup.org/hdf5/develop/group___h5_d.html#gae47c3f38db49db127faf221624c30609)
- [H5Sclose](https://docs.hdfgroup.org/hdf5/develop/group___j_h5_s.html#ga44b145f5c6977c082ac2c2dc121641fa)
- [H5Fclose](https://docs.hdfgroup.org/hdf5/develop/group___j_h5_f.html#gae2af8e1b3fcf6a3a98ab345ed3ff710f)


So in this section we need to write code to open the existing file and then create a dataset and write the dataset to the root group in the file.

```
#define FILE_NAME "file.h5"

// NOTE: opening file
hid_t file_id = H5Fopen(FILE_NAME, H5F_ACC_RDWR, H5P_DEFAULT);

// NOTE: setting dimensions of data
hsize_t dims[2] = {4, 6};

// NOTE: creating dataspace and dataset
hid_t dataspace_id = H5Screate_simple(2, dims, NULL);

// NOTE: writes default values to file
hid_t dataset_id = H5Dcreate2(file_id, "/dset", H5T_STD_I32BE, dataspace_id, H5P_DEFAULT, H5P_DEFAULT, H5P_DEFAULT);

// NOTE: closing all resources, should check status after closing
H5Dclose(dataset_id);
H5Sclose(dataspace_id);
H5Fclose(file_id);
```

`h5dump file.h5` outputs:

```
HDF5 "file.h5" {
GROUP "/" {
   DATASET "dset" {
      DATATYPE  H5T_STD_I32BE
      DATASPACE  SIMPLE { ( 4, 6 ) / ( 4, 6 ) }
      DATA {
      (0,0): 0, 0, 0, 0, 0, 0,
      (1,0): 0, 0, 0, 0, 0, 0,
      (2,0): 0, 0, 0, 0, 0, 0,
      (3,0): 0, 0, 0, 0, 0, 0
      }
   }
}
}
```

## Conclusion

Hopefully you enjoyed my rantings about the HDF5 file format. I just wanted to give a basic overview of what HDF was and how to do some basic operatiosn with the file format. See you next time ✌️!