---
layout: post
title: "Replace Postman Collection Runner with `curl` and `jq`"
---

If you work with API, chances are that you use [Postman](https://www.postman.com). I have been using it for quite some time. But recently while I was using its Collection Runner to reproduce a bug, I noticed that, with a free plan, it only gives you a limited number of runs each month. Postman is a great product and they have every right to make money on it. Personally I am not willing to upgrade to a paid plan anytime soon. That means I have to find something else to meet my needs.

Honestly, implementing something similar to a collection runner feature on your own is not that difficult at all. You can literally achieve the goal in any programming language. I realized that mostly because I have been so used to those button clicks on Postman GUI, I get lazy on writing codes for simple tasks like this, since we are all busy creating API applications. This limit forced me to quickly put together something I know but not an expert of.

To chain several API requests together and run them sequentially, bash script is a perfect way to achieve this. I just need a simple, flexible and temporary script to allow me to reproduce a bug. I know the server I am using has `curl` and `jq` installed already. Even if they are missing, the installation process is trivial.

I use `jq` primarily to extract the values from a JSON response. For example:
```
$ echo '{"id":1,"name":"hello"}' | jq .id
1
$ echo '{"items":[{"id":1,"name":"hello"},{"id":2,"name":"bye"}]}' | jq .items[].id
1
2
```

Sometimes, it's also handy to count the number of items in a collection. For example:
```
$ echo '{"items":[{"id":1,"name":"hello"},{"id":2,"name":"bye"}]}' | jq '.items | length'
2
```

Another nice thing to programmatically achieve this is to add any delays you want between requests.

In the example below, I extract a field from the first request, sleep a random amount of time between 0s-1s, then make a second request with a field value from the first request.
```
resp=$(curl --location -s "https://$host:443/jobs" \
--data '{
    "name": "test1",
}')

id=$( jq -r '.id' <<< "${resp}" )
echo $id

number=$(( ((RANDOM<<15)|RANDOM) % 999 ))
time=`awk "BEGIN{ printf \"%.3f\n\", $number/1000 }"`
sleep $time

curl --location --request DELETE "https://$host:443/jobs/$id"
```
