---
title: A glance at GIT internals
date: 2016-09-02 18:37:25
categories:
 - IT Technology
tags:
 - GIT
---

### Git - a stupid content tracker
fast, scalable, distributed revision control system with unusually rich command set that provides both high-level operations and full access to internals.
Total nearly 150+ commands.
> porcelain command: user friendly command
> plumbing command: low-level command

### Revision control? How hard?
simple method: just copy all the old files and paste it into a new folder and rename it with date.
it must supply main function: 
> tracking changes and versions

traditional revision control shortcomings (central repository):
> need to re-sync everytime your commit.
> single point failure: if server down, you can not recover the code history from client.(you can just recover snapshot)

why git is different? 
> git is distributed.

what does distributed mean? 
> you do not have one central place which keeps track of your data.
> client have all the data(includes the history).

<!-- more -->

### Git's version database
Basicly, it is a content addressable filesystem. a simple key-value data store.
Key is SHA-1 hash (20 bytes = 40 hex = 160 bits)
value is binary files, which contains:
> commits: snapshot
> trees: directories
> blobs: content of files / data. (Aha, Blob is short for binary large object)

### Next, let's see how git works:

#### git init

```cmd bash
git init demo1
```

git init creates an empty git repository: basicly a .git derectory.

fake git init:
```cmd bash
mkdir demo2
cd demo2
mkdir .git
mkdir .git\refs\heads
mkdir .git\refs\tags
mkdir .git\objects
echo ref: refs/heads/master > .git\HEAD
git status
```
git folder creation is successfully done.

#### git add

in demo1
```cmd bash
echo hello world > hello.txt
git add hello.txt

git hash-object hello.txt
4a1f4754cfddb3dd0d24067b7a1eed9ceac6a31d
```

and in objects directory, there would be a new folder called 4a and a file named `1f4754cfddb3dd0d24067b7a1eed9ceac6a31d` in it. 

in demo1, use git ls-files -- stage to see what files is in staging area.
and use git cat-file to see the content of the specific SHA-1 code. (given the prefix of the SHA-1)
```cmd bash
git ls-files --stage
100644 4a1f4754cfddb3dd0d24067b7a1eed9ceac6a31d 0       hello.txt

git cat-file -p 4a1f475
hello world

git cat-file -t 4a1f475
blob
```

git add will generate the blob in the objects folder, and update it to stage area(staging area is a index file under .git directory, if you remove index file, then use `git status` you will see the files it back to working area).

at this stage, if you create a file named hello2.txt and share the same content with hello.txt, in objects, it won't generate a new file. since it only contains file content in the SHA-1 named blob file.

in demo2, how to use `git add` manually?
step1: generate SHA-1 hash file
step2: generate index file

```cmd hash
echo hello world > hello.txt
git hash-object hello.txt -w
git update-index --add hello.txt
git status
```

#### git commit

in demo1
```cmd bash
git commit -m "test"
git log
commit 1f72ef4ecd9f5e7764ae724ae3f5d4ae66a89b00
Author: xxx
Date:   Fri Sep 2 17:38:52 2016 +0800

git cat-file -t 1f72
commit

git cat-file -p 1f72
tree e06d595ebb52f88bd09eca6580cc8d59c77e16ec
author xxx 1472809132 +0800
committer xxx 1472809132 +0800

test

git cat-file -t e06d
tree

git cat-file -p e06d
100644 blob 4a1f4754cfddb3dd0d24067b7a1eed9ceac6a31d    hello.txt
```

there are 2 more files under objects, first is `1f72ef4ecd9f5e7764ae724ae3f5d4ae66a89b00`, it is a commit, and contains tree hash, author, date. the second is `e06d595ebb52f88bd09eca6580cc8d59c77e16ec`, it is a tree(directory).

how to find every commits?
in refs\heads\master, commits' head hash is stored in this file.

then I will add another commit
```cmd bash
git cat-file -p 44298
tree 60ca37b3eacbbafe7fe2d4937435fc066ea66f4d
parent 1f72ef4ecd9f5e7764ae724ae3f5d4ae66a89b00
author xxx 1472810103 +0800
committer xxx 1472810103 +0800

test submit 2

git cat-file -p  60ca37
100644 blob 4a1f4754cfddb3dd0d24067b7a1eed9ceac6a31d    hello.txt
100644 blob a11c93477878ecbf51db644f8f3d216401cdecee    hello2.txt

git ls-files --stage
100644 4a1f4754cfddb3dd0d24067b7a1eed9ceac6a31d 0       hello.txt
100644 a11c93477878ecbf51db644f8f3d216401cdecee 0       hello2.txt
```

all files are in root directory. if there is a sub directory, it might be like this

```cmd bash
git cat-file -p e893
040000 tree d9eed2a757811df3416c243044f61f62e6d03813    folder
100644 blob 4a1f4754cfddb3dd0d24067b7a1eed9ceac6a31d    hello.txt
100644 blob a11c93477878ecbf51db644f8f3d216401cdecee    hello2.txt

git cat-file -p d9ee
100644 blob a11c93477878ecbf51db644f8f3d216401cdecee    hello2.txt
```

#### git branch, git checkout
```cmd bash
git checkout -b branch
Switched to a new branch 'branch'
```
there is a new file under refs\head\branch, and its content is its head SHA-1 code.
and if you use git checkout, it will modify HEAD file's content to `ref: refs/heads/branch
`.

#### git merge
step1, find the common ancestor of two linked list.
step2, copy the diff objects. deal with conficts of file. and generate a new commit.

### summary
Git as a filesystem, it is content addressable.