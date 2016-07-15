---
layout: post
title: Introducing JSON Guard
---

## Introduction

[JSON Schema](http://json-schema.org/) is a way to describe what JSON should look like, in JSON, and it's written in JSON.

<iframe src="//giphy.com/embed/mNdOc0Aziv88E?html5=true" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

So if you are writing an e-commerce API that accepts product data like this:

```json
{
    "name":       "Nintendo 64",
    "category":   "Game Consoles"
    "price":      "199.99",
    "updated_at": "1996-09-26T08:30:06.283185Z"
}
```

...you can write a schema like this:

```json
{
  "type": "object",
  "properties": {
    "name":       { "type": "string" },
    "category":   { "type": "string" },
    "price":      { "type": "string", "pattern": "^[0-9]*\\.[0-9]{2}$"}
    "updated_at": {"format": "date-time"}
  }
}
```

...and use that schema to validate that user input meets your expectations.  And since it's JSON, it's language agnostic and reusable.

It isn't only useful for validation though.  [Heroku uses JSON Schema to generate API clients in different languages](https://blog.heroku.com/archives/2014/1/8/json_schema_for_heroku_platform_api), [generate API documentation](https://github.com/interagent/prmd), [validate requests](https://github.com/interagent/committee), and TEST API endpoints.  It's really powerful stuff.

## Validating

Since I do PHP development, I've been using a package called [justinrainbow/json-schema](https://github.com/justinrainbow/json-schema) to validate data against my schemas.  It was the only JSON Schema validation package available for PHP that implemented most of Draft4.  Which is crazy, since there are 17+ validators for Javascript.

## Issues


## Finding a Solution

At the end of last year I was writing an API with the [Lumen](https://lumen.laravel.com/) framework.  We used JSON Schema for test assertions and documentation.  I got really excited about the idea of using JSON Schema for validation too, since I was writing a schema and than writing PHP code to perform all of the validations.  I didn't think the validator was reliable enough to use in production, so I started looking in to fixing it.
