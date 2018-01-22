---
layout: post
title: Protobuf PHP Services
---

# Introduction

Lately I've been investigating [Protobuf](https://developers.google.com/protocol-buffers/) as a replacement for JSON.  If you aren't familiar with Protobuf, it's a language neutral serialization format from Google.  It's most commonly associated with Google's RPC framework [GRPC](https://grpc.io/) but it can be used standalone too.  In this guide we are going to build a simple calculator RPC service using nothing but the Protobuf compiler and PHP.

## Example Code

The example code for this article is available at [https://github.com/yuloh/yuloh.github.io/tree/master/examples/protobuf-php-services](https://github.com/yuloh/yuloh.github.io/tree/master/examples/protobuf-php-services).

# Writing Protos

Protobuf definitions are written in a text file with the `.proto` extension.  We are going to define our calculator service and all of it's messages in a single proto file.  Let's start by defining our service.

I like to put my protos in a separate `proto` directory, so I am going to make a new file at `proto/math.proto`.

```bash
mkdir proto
touch proto/math.proto
```

## Services

```protobuf
syntax = "proto3";

package yuloh.math;

option php_generic_services = true;

service Calculator {
    rpc add (AddRequest) returns (AddReply) {}
    rpc subtract (SubtractRequest) returns (SubtractReply) {}
}
```

The first line tells the protobuf compiler which version we want to use.  PHP only supports proto3 syntax.  The `package` specifier is optional but a good idea to avoid name clashes.  In the generated PHP code it will be converted to the namespace. the `php_generic_services` option tells the compiler we want to generate an interface for the service.  It's optional because you might be using a plugin like grpc that generates it's own service classes.


## Messages

Now we need to define the request and reply messages.  This should be pretty straightforward but you may be wondering about the value assignment after each field name.  The number is the **tag**, and it's used to uniquely identify the field when the message is serialized to the binary format.

```protobuf
message AddRequest {
    int32 x = 1;
    int32 y = 2;
}

message AddReply {
    int32 sum = 1;
}

message SubtractRequest {
    int32 x = 1;
    int32 y = 2;
}

message SubtractReply {
    int32 diff = 1;
}
```

# Compilation

For the next step you will need to install the protobuf compiler.  Head over to the [installation instructions](https://github.com/google/protobuf#protocol-compiler-installation) and install it.

The protobuf compiler will read our proto definitions and generate a few classes for us.  We are going to put the generated code in a separate directory from our application code.  Make the directory (otherwise the compiler will complain) and run protoc.

```bash
mkdir gen
protoc  --php_out=./gen ./proto/math.proto
```

You should see a generated class for every message we defined as well as an interface for the service.


# Composer Setup

## Autoloading

The generated code is organized using PSR-4 standards and can be autoloaded with composer.  There are currently 2 namespaces that need to be autoloaded - the `Yuloh\Math` namespace for our service and `GPBMetadata\Proto`, which contains internal files required by the Protobuf runtime.  If you don't want to manually map namespaces each time you can just use the empty namespace.

```json
{
    "autoload": {
      "psr-4": {
        "": "gen"
      }
    }
}
```

## Protobuf Runtime

You will also need the protobuf runtime for PHP.  You can either install the runtime as a C extension or use the PHP package.  The PHP can be installed with composer.  Follow the [instructions](https://github.com/google/protobuf/tree/master/php) if you want to use the extension.

```bash
composer require google/protobuf
```

# Writing the Service

The interface has been generated but we still need to implement it.  Create a `src` directory and inside of it create the calculator service.

```php
<?php
// src/Calculator.php

namespace Yuloh\Math;

class Calculator implements CalculatorInterface
{
    public function add(AddRequest $request): AddReply
    {
      $sum = $request->getX() + $request->getY();

      return (new AddReply())->setSum($sum);
    }

    public function subtract(\Yuloh\Math\SubtractRequest $request): SubtractReply
    {
      $diff = $request->getX() - $request->getY();

      return (new SubtractReply())->setDiff($diff);
    }
}
```

The service implements a method for each `rpc` we defined in the proto.  The contract defined in the proto file is enforced by the generated interface.

# Serializing Messages

A protocol buffer can be serialized to text, JSON, or an efficient binary format.  You should always prefer the binary format for production.  Not just for efficiency but also because it allows you to benefit from protobuf's backwards and forwards compatibility.

The PHP classes can be (de)serialized to JSON or binary.  For binary use `serializeToString` and `mergeFromString`.  JSON uses `serializetoJsonString` and `mergeFromJsonString`.

```php
use Yuloh\Math\AddRequest;

$request = new AddRequest();
$request->setX(2)->setY(5);
$encoded = $request->serializeToString();

$decodedRequest = new AddRequest();
$decodedRequest->mergeFromString($encoded);

var_dump($decodedRequest->getX(), $decodedRequest->getY());
```

# HTTP Server

Let's test our service by making it available over HTTP.  Our HTTP script will check if the path is `/add` or `/subtract`.  if so it will deserialize the request, call the service, and return the serialized reply.  Save this to `public/index.php` and boot the built in webserver.

```php
<?php
// public/index.php

use Yuloh\Math\AddRequest;
use Yuloh\Math\Calculator;
use Yuloh\Math\SubtractRequest;

require '../vendor/autoload.php';

$method = ltrim(rawurldecode($_SERVER['REQUEST_URI']), '/');

switch ($method) {
  case 'add':
    $request = new AddRequest();
    $request->mergeFromString(file_get_contents('php://input'));
    $reply = (new Calculator())->add($request);
    echo $reply->serializeToString();
    break;
  case 'subtract':
    $request = new SubtractRequest();
    $request->mergeFromString(file_get_contents('php://input'));
    $reply = (new Calculator())->subtract($request);
    echo $reply->serializeToString();
    break;
  default:
    http_response_code(404);
}
```

```bash
php -S localhost:8000 -t ./public
```

# Making Requests

Since our service currently only supports the binary format we need a way to encode our request.  We can use protoc to encode a text version of the request and pipe that into curl.  We will use protoc again to decode the response.

```bash
echo 'x:9,y:2' | protoc --encode=yuloh.math.AddRequest ./proto/math.proto | curl -sS -X POST --data-binary @- 'localhost:8000/add' | protoc --decode=yuloh.math.AddReply ./proto/math.proto
```

```
sum: 11
```

```bash
echo 'x:9,y:2' | protoc --encode=yuloh.math.SubtractRequest ./proto/math.proto | curl -sS -X POST --data-binary @- 'localhost:8000/subtract' | protoc --decode=yuloh.math.SubtractReply ./proto/math.proto
```

```
diff: 7
```

# Conclusion