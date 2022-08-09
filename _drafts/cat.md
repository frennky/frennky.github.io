---
layout: post
title:  "Cat"
date:   2011-03-18 20:09:30 +0100
---

# cat

Cat - Concatenate files to standard output.

Concatenate one file to standard output:

```
cat README.md
```

Concatenate multiple files, one after another, to standard output:

```
cat file1.txt file2.txt
```

Concatenate multiple files, one after another, to new file:

```
cat file1.txt file2.txt > file3.txt
```

Concatenate multiple files, one after another, and append to another file:

```
cat file1.txt file2.txt >> file3.txt
```

Concatenate file to standard output with line numbers:

```
cat -n README.md
```

Concatenate one file to standard output with all characters displayed:

```
cat -A README.md
```
