---
layout: post
title:  "Sed"
date:   2011-03-18 20:09:30 +0100
---

# sed

Sed command is very powerfull. It may take some time to master it, here are some basics.

With sed command you can:
- select or replace text
- add or remove lines
- modify (or not) original file

## Substitution

```bash
echo Hello World | sed 's/World/Jon/'
```

This will search for and replace word `World` with `Jon`. In case that text contains more then one occurrance of the word, you can replace them all with following experssion.

```bash
echo Hello there. I said hello there. | sed 's/there/Dave/g'
```

To ignore case, use `i` in the expression.

```bash
echo Hello there. I said hello there. | sed 's/hello/hi/ig'
```

To append line after line with matching text use `a` in the expression.

```bash
echo Hi Fred. | sed '/Hi/a Hi Jon'
```

Or in case you need to insert line before one with matching text use `i`.

```bash
echo Hi Fred. | sed '/Hi/i Hi Jon'
```

Notice the difference with expression that uses ignore case.

If you need to refer to matched text in case you want to add text to that line use `&`

```bash
echo Hi Jon | sed 's/Jon/Mr &/'
echo Hi Jon | sed 's/.*/Dave: &/'
```

## Delete

For deleting lines containing search pattern, use `d` in the expression.

```bash
echo Hi Jon | sed '/Jon/d'
```

## Print

```bash
echo Hello World | sed -n '/World/p'
```

## Saving output

In case you're working with a file and need to save changes (overwrite), use `-i`.

```bash
sed -i 's/Jon/Tom/' file.txt
```

And in case you wish to save a backup of original file.

```bash
sed -i'.bak' 's/Jon/Tom/' file.txt
```
