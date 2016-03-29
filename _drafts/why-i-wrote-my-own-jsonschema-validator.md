---
layout: post
title: Why I Wrote My Own Json Schema Validator
---

# Introduction

[JSON Schema](http://json-schema.org/) is a way to describe what JSON should look like, in JSON, and it's written in JSON.

<iframe src="//giphy.com/embed/mNdOc0Aziv88E?html5=true" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

So if you are writing an API that accepts data like this:

```json

```

...you can write a schema like this:

```json

```

...and use that schema to validate the JSON the user sends meets your expectations.  And since it's JSON, it's language agnostic and reusable.

# Validating

Since I do PHP development, I've been using a package called [justinrainbow/json-schema](https://github.com/justinrainbow/json-schema) to validate data against my schemas.