---
layout: post
title: "Combine multiple Git repositories and keep their histories"
---

You may run into situations where multiple Git repositories need to be merged. For us, it is driven by a conclusion that several separate but related services can be combined into one merged service that handles multiple context paths. 

Obviously, we would like to keep the commit histories when combining our services. This can be essentially achieved by creating a merge commit that connects the repos together. 

To use an oversimplified example, let's say we have two Git repositories, "repo1" and "repo2", and we want to move all the content in "repo2" into "repo1".

Here is what looks like before merge:
```
repo1
├── file1.txt
└── file2.txt

repo2
├── file3.txt
└── file4.txt
```

Optionally, to avoid any merge conflict hassle, you can reorganize the repo structure so that each project is placed in a subproject directory. 
```
$ cd repo1
$ mkdir -p services/repo1
$ git mv !(services) services/repo1
$ git commit -m "restructure repo"
```

Repeat that for repo2 as well.

The file structure may look like this:
```
repo1
└── services
    ├── file1.txt
    └── file2.txt

repo2
└── services
    ├── file3.txt
    └── file4.txt
```

To start the actual merge, first go to repo1 and then set up repo2 as a remote repo.
```
$ cd repo1
$ git remote add --fetch repo2 ../repo2
```

After that, merge with remote repo repo2's main branch, without auto-creating a merge commit.
```
$ git merge repo2/main --no-commit --allow-unrelated-histories
Automatic merge went well; stopped before committing as requested
```

At this point, the content in repo2 should have been moved into repo1, along with commit histories. Inspect, make changes as necessary. When it's ready, do the merge commit.
```
$ git add .
$ git commit -m "merge repo2 into repo1"
```

To make sure histories are kept, do a `git log` to confirm.

Finally, the repo2 remote can be deleted.
```
$ git remote rm repo2
```
