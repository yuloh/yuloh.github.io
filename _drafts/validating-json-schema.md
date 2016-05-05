---
layout: post
title: Validating JSON Schema
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
    "price":      { "type": "string", "pattern": "^[0-9]*\\\\.[0-9]{2}$"}
    "updated_at": {"format": "date-time"}
  }
}
```

...and use that schema to validate that user input meets your expectations.  And since it's JSON, it's language agnostic and reusable.

It isn't only useful for validation though.  [Heroku uses JSON Schema to generate API clients in different languages](https://blog.heroku.com/archives/2014/1/8/json_schema_for_heroku_platform_api), [generate API documentation](https://github.com/interagent/prmd), [validate requests](https://github.com/interagent/committee), and TEST API endpoints.  It's really powerful stuff.

## Validating

Since I do PHP development, I've been using a package called [justinrainbow/json-schema](https://github.com/justinrainbow/json-schema) to validate data against my schemas.  It was the only JSON Schema validation package available for PHP that implemented most of Draft4.  Which is crazy, since there are 17+ validators for Javascript.

## References

After using it for awhile, I ran into a lot of problems.  We used references, which are a way to reference another part of the document, or another schema.  A simple example would look like this:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "definitions": {
    "pet": {
      "type": "object",
      "properties": {
        "name":  { "type": "string" },
        "breed": { "type": "string" },
        "age":  { "type": "string" }
      },
      "required": ["name", "breed", "age"]
    }
  },
  "type": "object",
  "properties": {
    "cat": { "$ref": "#/definitions/pet" },
    "dog": { "$ref": "#/definitions/pet" }
  }
}
```

The reference is a [JSON pointer](https://tools.ietf.org/html/rfc6901) to another part of the document.  To validate the schema the reference needs to be resolved.  The naive way to do this is to inline the reference, so that the document looks like this:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "definitions": {
    // The definition is probably still here, but not actually used anymore.
  },
  "type": "object",
  "properties": {
    "cat": {
      "type": "object",
      "properties": {
        "name":  { "type": "string" },
        "breed": { "type": "string" },
        "age":  { "type": "string" }
      },
      "required": ["name", "breed", "age"]
    },
    "dog": {
      "type": "object",
      "properties": {
        "name":  { "type": "string" },
        "breed": { "type": "string" },
        "age":  { "type": "string" }
      },
      "required": ["name", "breed", "age"]
    }
  }
}
```

That works, but it breaks down because JSON Schema allows circular references.  A circular reference looks like this:

```json
{
  "person": {
    "properties": {
        "name": {
          "type": "string"
        },
        "spouse": {
          "type": {
            "$ref": "#/person"        // circular reference
          }
        }
    }
  }
}
```

If you try to inline that reference, you are going to end up with infinite recursion.

![]({{ site.url }}/assets/article_images/Droste.jpg)

The library I was using solved this by keeping track of how many levels of recursion took place, and throwing an exception if you exceeded them (they have since fixed this in v2.0.0).  In practice this meant that you had to re-adjust the depth depending on how nested your schema was.  It would also silently pass validation for any schemas with root pointer references, which is a pointer that starts with `#`.  This meant that you couldn't validate using the JSON Meta Schema, so it was impossible to verify your schema was correct [^n].

## Finding a Solution

At the end of last year I was writing an API with the [Lumen](https://lumen.laravel.com/) framework.  We used JSON Schema for test assertions and documentation.  I got really excited about the idea of using JSON Schema for validation too, since I was writing a schema and than writing PHP code to perform all of the validations.  I didn't think the validator was reliable enough to use in production, so I started looking in to fixing it.

##

[^n]: The solution is to not inline the schema, but to replace circular references with a proxy object that is only resolved when accessed.  [Here is a link to the gist I wrote when I was figuring it out](https://3v4l.org/MA7ND).  The proxy object is only resolved as long as there is data to validate.  You can still use a max depth to prevent deeply nested payloads from killing your server, just like the max depth for `json_decode`.  If you do that, your data won't re-encode as JSON since it has circular references.  Instead you can put the `$ref` object back and everything looks like it did before you dereferenced it.

