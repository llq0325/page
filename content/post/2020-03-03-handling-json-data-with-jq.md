---
title: Handling JSON data with jq
author: CE West
date: '2020-03-03'
slug: handling-json-data-with-jq
categories:
  - commandline
tags: 
  - json
  - jq
meta_img: images/image.png
description: Introduction for navigating JSON data in the command-line with jq


---

For a new project I've unwillingly transitioned from the safety and comfort of CSV files to a scary new world of JSON files.
<!--more-->

[jq](https://stedolan.github.io/jq/) is a command-line JSON processor that was recommended to me by a colleague. Being written in C with no dependencies, it's very [easy to install](https://stedolan.github.io/jq/download/). 

Here's what won me over:

> jq is like `sed` for JSON data - you can use it to slice and filter and map and transform structured data with the same ease that `sed`, `awk`, `grep` and friends let you play with text.

*Swoon!*

A jq program/command filters a stream of JSON data in a way that should be familiar to users of `sed`, `awk` and `grep`. By default, the output to standard out is also nicely formatted JSON. 

As an example, let's get some information from the PDB about my favourite entry, 1CEW:

```console 
clare$ curl --request GET https://www.rcsb.org/pdb/json/describeMol?structureId=1cew 
{"id":"1CEW","polymerDescriptions":[{"entityNr":"1","length":"108","chain":{"id":"I"},"polymerDescription":{"description":"CYSTATIN"}}]}
```

Nobody has time to read unformatted JSON, so let's pipe that through the simplest jq programme -- `jq '.'` --  which parses the JSON and makes no change other than formatting it nicely:

```console
clare$ curl --request GET https://www.rcsb.org/pdb/json/describeMol?structureId=1cew | jq '.'
{
  "id": "1CEW",
  "polymerDescriptions": [
    {
      "entityNr": "1",
      "length": "108",
      "chain": {
        "id": "I"
      },
      "polymerDescription": {
        "description": "CYSTATIN"
      }
    }
  ]
}
```

We can retrive the values of keys that we care about using '.field', which produces the value at the key 'field', or null if there is no value present:

```console
clare$ curl --request GET https://www.rcsb.org/pdb/json/describeMol?structureId=1cew > test.json
clare$ cat test.json | jq '.id'
"1CEW"
clare$ cat test.json | jq '.name'
null

```
We can also filter for particular fields that we care about, outputting in JSON format:

```console
clare$ cat test.json | jq '{id}'
{
  "id": "1CEW"
}

jq commands can be piped together using `|`

`.a | .b | .c` or, equivalently, `a.b.c`

For example, to get the length of the first entity, we can filter for `polymerDescription` and then filter for the field `length`

```console
clare$ cat test.json | jq '.polymerDescriptions[] | .length'
"108"
clare$ cat test.json | jq '.polymerDescriptions[].length'
"108"
```

You can index or slice arrays, for example:  
- `.[0]` -- first element in the array
- `.[2]` -- third element in the array
- `.[-1]` -- last element
- `.[1-5]` -- second to fifth elements

For example, the result of our API request is only one JSON object, if it was many we could use `jq '.[0]'` to only get the first.  

To iterate over the elements and perform the commands on each individually, use:
`.[]`  

Using jq, you can therefore easily manipulate JSON data including filtering, renaming, creating and subsetting fields:

```console
clare$ cat test.json | jq '{id, protein: "Chicken egg white cystatin", chain: .polymerDescriptions[0].chain.id, length: .polymerDescriptions[0].length}'
{
  "id": "1CEW",
  "protein": "Chicken egg white cystatin",
  "chain": "I",
  "length": "108"
}
```

Maybe you want to extract all the objects from a large JSON file with a particular field value:

```console
clare$ jq 'select(.thing=="thingofinterest")' large.json
```

If you're saving to a file, you may prefer to use the option `--compact-output`/`-c` to generate a more compact output in which each JSON object is on a single line. 

See the [jq manual](https://stedolan.github.io/jq/manual/) for more information and lots of other cool things you can do!


