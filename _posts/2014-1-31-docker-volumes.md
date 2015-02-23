---
layout: post
title: "Using docker volumes for rapid development of containers"
description: ""
category: 
tags: [docker, development environments, vagrant]
---

Its fairly obvious how to use docker for shipping an immutable image
that is great for deployment.. It was less obvious (to me) how to use
docker to iterate on the image, run tests in it, etc...

Lets say you have a node project and your writing some web service
thing:

```js
// server.js
var http = require('http');
...
```

```js
// server_test.js
suite('my tests', function() {
});
```

```
# Dockerfile
FROM lightsofapollo/node:0.10.24
ADD . /service
WORKDIR /service
CMD node server.js
```

## Before Volumes

Without using volumes your workflow is like this:

```
docker build -t image_name
docker run image_name ./node_modules/.bin/mocha server_test.js
# .. make some changes and repeat...
```

While this is certainly not awful its a lot of extra steps you probably
don't want to do...

## After Volumes

While iterating ideally we could just "shell in" to the container and
make changes on the fly then run some tests (like lets say vagrant).

You can do this with volumes:

```
# Its important that you only use the -v command during development it
# will override the contents of whatever you specify and you should also
# keep in mind you want to run the final tests on the image without this
# volume at the end of your development to make sure you didn't forget to
# build or somthing.

# Mount the current directory in your service folder (override the add
# above) then open an interactive shell
docker run -v $PWD:/service -i -t /bin/bash
```

From here you can hack like normal making changes and running tests on
the fly like you would with vagrant or on your host.


## When your done!

I usually have a makefile... I would setup the "make test" target
something like this to ensure your tests are running on the contents of
your image rather then using the volume

```make
.PHONY: test
test:
  docker build -t my_image
  docker run my_image npm test

.PHONY: push
push: test
  docker push my_image
```
