+++
title = 'Frequently used jq commands'
date = 2023-08-27T00:00:00-04:00
comments = true
math = false
+++

As we deal with JSON data a lot. `jq` is a must-have tool that allows you to work with them more efficiently. It is useful when you want to extract particular information of interest. Here I am documenting some example commands that our team use quite frequently for our daily tasks.

Of course, `jq` is capable of more than these. For a full list of what it can do, refer to its official documentation: https://jqlang.github.io/jq/manual/

#### Extract a single value from an object:
```
$ cat | jq .receipt.id << ---
{
  "receipt": {
    "id": 1
  }
}
---
1
```

####  Extract a single value from an array:
```
$ cat | jq ".receipt | .[1]" << ---
{
  "receipt": ["a", "b"]
}
---
"b"
```

#### Extract values from an array and print them out:
```
$ cat | jq .items[].name << ---
{
  "items": [
    {
      "name": "hello",
      "createdBy": "tom"
    },
    {
      "name": "hi",
      "createdBy": "tom"
    },
    {
      "name": "bye",
      "createdBy": "jerry"
    }
  ]
}
---
"hello"
"hi"
"bye"
```

#### Extract unique values from an array and print them out:
```
$ cat | jq "[.items[].createdBy] | unique | .[]" << ---
{
  "items": [
    {
      "name": "hello",
      "createdBy": "tom"
    },
    {
      "name": "hi",
      "createdBy": "tom"
    },
    {
      "name": "bye",
      "createdBy": "jerry"
    }
  ]
}
---
"jerry"
"tom"
```

#### Filter out items in an array based on a condition and construct a new array:
```
$ cat | jq '[.items[] | select(.createdBy == "tom")]' << ---
{
  "items": [
    {
      "name": "hello",
      "createdBy": "tom"
    },
    {
      "name": "hi",
      "createdBy": "tom"
    },
    {
      "name": "bye",
      "createdBy": "jerry"
    }
  ]
}
---
[
  {
    "name": "hello",
    "createdBy": "tom"
  },
  {
    "name": "hi",
    "createdBy": "tom"
  }
]
```

#### Filter out items in an array based on a condition and print items out:
```
$ cat | jq '.items[] | select(.tags | contains(["red"]))' << ---
{
  "items": [
    {
      "name": "hello",
      "createdBy": "tom",
      "tags": [
        "red",
        "large"
      ]
    },
    {
      "name": "hi",
      "createdBy": "tom",
      "tags": [
        "yellow",
        "small"
      ]
    },
    {
      "name": "bye",
      "createdBy": "jerry",
      "tags": [
        "blue",
        "medium"
      ]
    }
  ]
}
---
{
  "name": "hello",
  "createdBy": "tom",
  "tags": [
    "red",
    "large"
  ]
}
```

#### Sort an array of objects:
```
$ cat | jq ".items | sort_by(.name)" << ---
{
  "items": [
    {
      "name": "hello",
      "createdBy": "tom"
    },
    {
      "name": "hi",
      "createdBy": "tom"
    },
    {
      "name": "bye",
      "createdBy": "jerry"
    }
  ]
}
---
[
  {
    "name": "bye",
    "createdBy": "jerry"
  },
  {
    "name": "hello",
    "createdBy": "tom"
  },
  {
    "name": "hi",
    "createdBy": "tom"
  }
]
```