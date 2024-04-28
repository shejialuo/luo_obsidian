# Python Binding

[python-llfuse](https://github.com/python-llfuse/python-llfuse) provides bindings for the low level [FUSE](http://github.com/libfuse/libfuse) library.

## Overview

A file system is implemented by subclassing the `llfuse.Operations` class and implement the various request handlers. The handlers respond to requests received from the FUSE kernel module and perform functions like looking up the inode given a file name and looking up attributes of an inode.

An instance of the operations class is passed to `llfuse.init` to mount the file system. To enter the request handling loop, run `llfuse.main`. This function will return when the file system should be unmounted again, which is done by calling `llfuse.close`.

All character data are passed as bytes and must be returned as bytes.

## First Example: Doing Nothing

Below is an example which shows an useless filesystem that does nothing but existing.

```python
# nothingfs.py

import sys
import llfuse

class NothingOps(llfuse.Operations):
    def __init__(self):
        super().__init__()

if __name__ == '__main__':
    print("Mounting to directory", sys.argv[1])
    ops = NothingOps()
    llfuse.init(ops, sys.argv[1], ['fsname=nothingfs'])
    try:
        llfuse.main()
    except:
        llfuse.close()
        raise
    llfuse.close()
```

And we could type the following commands in the shell:

```bash
mkdir Mountpoint
python3 nothingfs.py Mountpoint &
```

And we use `ls` to see what happened:

```bash
ls -all
```

We could see the following results:

```txt
ls: cannot access 'Mountpoint': Function not implemented
d????????? ? ?    ?       ?            ? Mountpoint
```

And to unmount the filesystem, use

```bash
fusermount -u Mountpoint
```

From above, we can see that something is necessary to

1. Access the directory and list its content.
2. Get information about the directory (permissions, size, creation time...)

## Add The Handlers

We should add the handlers to the `NothingOps` class. The handlers are some common file system operations. Below are some interfaces.

+ `opendir`
+ `readdir`
+ `read`
+ `open`
## Reference

1. [python-llfuse documentation](https://llfuse.readthedocs.io/en/latest/)
2. [llfuse tutorial in python3](https://gist.github.com/akiross/3d7952b186d523c61cbc)