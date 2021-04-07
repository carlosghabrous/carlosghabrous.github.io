---
layout: post
title: Is this directory empty? A python and golang comparison
tags: python golang
author: carlos
---
I recently started learning Golang, because well, it's func! (pun intended). 
Over time, I found that working on a project plus reading are the most effective ways for me to learn something new. In this little project I am working on, I came up with the need of checking whether a directory is empty or not. I also realized that right now, when I think about doing something programmatically, my head starts the process in Python mode. So, I thought it could be interesting (interesting for me, at least), to compare how I would tackle such a simple task in both languages. 

In Golang, I first jumped to the _os_ package documentation, to find that there is nothing really implementing this feature (don't reinvent the wheel kind of thing).

So, I came up with something like this: 
```go
func isDirEmpty(dir string) (bool, error) {
    d, err := os.Open(dir)
    if err != nil {
        return false, fmt.Errorf("Error while reading current directory: %v\n", err)
    }
    defer d.Close()

    _, err = d.Readdirnames(1)
    if err == io.EOF {
        return true, nil
    }

    return false, err
}

```
Pretty obvious, just open the directory and check for errors. Make sure the directory handler is closed once we are done (_defer_). 
In choosing what to do next, I found two functions that could help: _Readdir_ and _Readdirnames_, whose signature are the following: 

```go
func (f *File) Readdir(n int) ([]FileInfo, error)
func (f *File) Readdirnames(n int) (names []string, err error)
```

They are pretty similar, and the _n_ argument serves the same purpose: it just limits the number of items returned by the function to _n_ tops. The key difference is that _Readdirnames_ returns a slice of strings, whereas _Readdir_ returns a slice of _FileInfo_ structs. If you follow the documentation, the _FileInfo_ struct looks like this:
```go
type FileInfo interface {
    Name() string       // base name of the file
    Size() int64        // length in bytes for regular files; system-dependent for others
    Mode() FileMode     // file mode bits
    ModTime() time.Time // modification time
    IsDir() bool        // abbreviation for Mode().IsDir()
    Sys() interface{}   // underlying data source (can return nil)
}
```

Taking this into account, it is pretty obvious that using the former function is more effective, since a _FileInfo_ slice is potentially heavier than a slice of strings.
Finally, we just need to make the chosen function read just one item and check whether the error returned is _io.EOF_, which according to the documentation indicates an empty directory. We just need to account for the case where no error is returned, which would mean the directory has some content. 

What about the Python implementation? In my head, such simple test should require less lines of code. Let's see: 
```python
import os
def is_dir_empty(dir_name):
    if not os.listdir(dir_name):
        return True

    return False
```

The _listdir_ function in the _os_ package returns an empty list if the directory is empty, and a list containing the names of the entries in the given directory. 
The code above would certainly work with directories that exist, but what would happen if the directory doesn't? 
```python
In [7]: is_dir_empty(‘/some/dir‘)
---------------------------------------------------------------------------
FileNotFoundError                         Traceback (most recent call last)
<ipython-input-7-523e8da95b46> in <module>
----> 1 is_dir_empty(‘/some/dir')

<ipython-input-5-4c93824b1a99> in is_dir_empty(dir_name)
      1 def is_dir_empty(dir_name):
----> 2     if not os.listdir(dir_name):
      3         return True
      4     return False
      5

FileNotFoundError: [Errno 2] No such file or directory: '/some/non/existing/dir'
```
Ups! In production, such code would make our application crash. So, like in Golang, we need to take care of protecting it against wrong input values. There are several alternatives for this situation: 

```python
def is_dir_empty(dir_name):
    if os.path.exists(dir_name) and os.path.isdir(dir_name):
        if not os.listdir(dir_name):
            return True
        else:    
            return False
    else:
        raise Exception("some message here")
```
The problem with this approach is that we are checking two conditions at the beginning (directory exists and item is a directory), and throwing the same error regardless of which one happens. So, we could re-write it like this: 

```python
def is_dir_empty(dir_name):
    if not os.path.exists(dir_name):
        raise FileNotFoundError(f'Directory {dir_name} does not exist!')
        
    if not os.path.isdir(dir_name):
        raise NotADirectoryError(f'Item {dir_name} is not a directory!')

    if not os.listdir(dir_name):
        return True
    else:    
        return False
```
To me, there is less code nested, and this is more readable than the first implementation. 
We could also do this by just trying to read first without checking anything at all: 
```python
def is_dir_empty(dir_name):
    try:
        return len(os.listdir(dir_name)) == 0

    except FileNotFoundError as e:
        raise FileNotFoundError(f'Custom message {e}')

    except NotADirectoryError as e:
        raise NotADirectoryError(f'Custom message {e}')

```
In these previous implementations, re-raising the exception doesn't really add value, but it just shows that it can be done in case sending a custom message is needed. 

As a conclusion, the naive first one-liner python implementation might suggest that writing in other languages, such Golang in this case, might be cumbersome. But if we want to cover all possible misuses of our software, extra care is needed, and the code in different languages might end up looking just alike. 

Code safe!
Carlos. 
