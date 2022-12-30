---
title: Bash script cheatsheet
date: 2022-12-10 15:33:25
categories:
 - IT Technology
tags:
 - Bash
 - Cheatsheet
---

# And or list

```shell
echo "I am going to do sth" && doSomething 2>/dev/null || error cmd (echo $?)`
```

print the exit code: `echo $?`
blackhole for information: `/dev/null`

<!-- more -->

# File redirection

```
redirect to file: `> file`
append to file: `>> file`
read from file: `< file`
```

## File descriptor
file descriptors:

```
0 - input data
1 - standard out
2 - standard error
```

- throw stdout(file descriptor 1) to file: `command > file`
- throw stderr(file descriptor 2) to file: `command 2 > file`
- throw both to file: `command > file 2>&1`

# Utility classes

```
sort
uniq
grep
fmt
tr
head/tail
sed
awk
```


pipe: 

![pipe](pipe.png)

# Arrays

```
numbers=(1 2 3 four "5", 6)
$ echo $numbers
1 2 3 four 5, 6
$ echo ${numbers[3]}
3
$ echo ${numbers[@]}
1 2 3 four 5, 6
$ echo ${#numbers[@]}
6
$ numbers+=(seven)
$ echo ${#numbers[@]}
7
$ echo ${numbers[@]}
1 2 3 four 5, 6 seven
$ echo ${numbers[@]:2:4}
3 four 5, 6
$ echo ${numbers[@]:2:2}
3 four
$ echo ${numbers[@]:2}
3 four 5, 6 seven
```

# Substitution

```
$ cat <(echo "Hello world")
Hello world
$ echo <(echo "Hello world")
/dev/fd/11
$ diff <(ll /folder1) <(ll /folder2)
# output diff files
```

# Loop

```
for i in $(seq 1 10); do
echo "count to $i";
done;

count to 1
...
count to 10

// test command
$ test 10 -lt 0 && echo "false" || echo "true"
true
$ [ 10 -lt 0 ] && echo "false" || echo "true"
true

while [ $Counter -gt 0 ]; do
echo "count to $Counter"
let Counter-=1
done;
count to 10
...
count to 1

until [ $Counter -lt 1 ]; do
echo "count to $Counter"
let Counter-=1
done;
count to 10
...
count to 1
```

# Signal/traps

```
cleanup()
{
    echo "cleaning up ..."
}

trap cleanup SIGINT

while true
do
    echo Sleeping
    sleep 10
done
```

### Heredoc/herestring

```
$ cat << DELIMITER
heredoc>        hello
heredoc> ${USER}
heredoc> DELIMITER
	hello
ttingtao

$ cat << "DELIMITER"
heredoc>        hello
heredoc> ${USER}
heredoc> DELIMITER
	hello
${USER}

$ cat <<- "DELIMITER"
heredocd>       hello
heredocd> ${USER}
heredocd> DELIMITER
hello
${USER}

$ VAR="this is a simple test"
$ cat <(echo $VAR)
this is a simple test
$ cat <<< $VAR
this is a simple test
```

# Debug scripts

```
// dryrun to check syntax error
set -n
// treat unset parameters/variables as error
set -u
// display the expanded command that run
set -x

// add on shebang
#!/bin/bash -x
```

# Regex

```
$ grep -oE "\<of\>" <(echo "of off offff")
of

cat testSedInput
hello word:
------
name: sed
city: vancouver
weather: snowing
------
name: awk
city: singapore
weather: raining
------

$ sed -r '1s/(.+):/Report: \1\n/' testSedInput
Report: hello word

------
name: sed
city: vancouver
weather: snowing
------
name: awk
city: singapore
weather: raining
------

$ sed -r '/-{3,}/d' testSedInput
hello word:
name: sed
city: vancouver
weather: snowing
name: awk
city: singapore
weather: raining

$ sed -r 's/([a-z]+): ([a-z]+)/\1->\2/' testSedInput
hello word:
------
name->sed
city->vancouver
weather->snowing
------
name->awk
city->singapore
weather->raining
------

// or put all sed command into a script

$ cat sedpro.sh
#! /usr/bin/sed

1s/(.+):/Report: \1\n/
/-{3,}/d
s/([a-z]+): ([a-z]+)/\1->\2/

$ sed -E -f sedpro.sh testSedInput
Report->hello word

name->sed
city->vancouver
weather->snowing
name->awk
city->singapore
weather->raining
```