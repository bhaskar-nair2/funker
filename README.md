# Funker

Functions as Docker containers.

Funker allows you to package up pieces of application as Docker containers and have them run on-demand on a swarm.

## Getting started

### Creating a function

First, you need to package up a piece of your application as a function. Let's start with a trivial example: a function that adds two numbers together.

Save this code as `handler.js`:

```javascript
var funker = require('funker');

funker.handler(function(args, callback) {
  return args.x + args.y;
});
```

We also need to define the Node package in `package.json`:

```
{
  "name": "app",
  "version": "0.0.1",
  "scripts": {
    "start": "node handler.js"
  },
  "dependencies": {
    "funker": "^0.0.1"
  }
}
```

Then, we package it up inside a Docker container by creating `Dockerfile`:

```
FROM node:7-onbuild
```

And building it:

```
$ docker build -t add .
```

To run the function, you create a service and put it inside a network:

```
$ docker network create --attachable -d overlay funker
$ docker service create --name add --network funker add
```

The function is now available at the name `add` to other things running inside the network.

### Calling a function

Let's try calling the function from a Node shell:

```
$ docker run -it --net funker funker/python
```

(The `funker/python` image is just a Python image with the `funker` package installed.)

You should now see a Python prompt. Try importing the package and running the function we just created:

```python
>>> import funker
>>> funker.call("add", x=1, y=2)
3
```

Cool! So, to recap: we've put a function written in Node inside a container, then called it from Python. That function is run on-demand, and this is all being done with plain Docker services and no additional infrastructure.

## Architecture

## Deploying with Compose