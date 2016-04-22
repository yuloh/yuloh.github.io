---
layout: post
title: Understanding ORM Relationships
---

# Introduction

# The Database Schema

```
+-------------------+
| manufacturers     |
|-------------------|
| id                |
| name              |
+-------------------+

+-------------------+
| accounts          |
|-------------------|
| id                |
| account_number    |
| manufacturer_id   |
+-------------------+

+-------------------+
| bikes             |
|-------------------|
| id                |
| model             |
| manufacturer_id   |
+-------------------+

+-------------------+
| parts             |
|-------------------|
| id                |
| part_number       |
+-------------------+

+-------------------+
| bike_part         |
|-------------------|
| bike_id           |
| part_id           |
+-------------------+

+-------------------+
| serial_numbers    |
|-------------------|
| id                |
| serial_number     |
| serializable_id   |
| serializable_type |
+-------------------+
```

# Relationship Types

## One To One

### Has One

### The Inverse: Belongs To One

## One To Many

### Has Many

### The Inverse: Belongs To One

## Many To Many

## The Inverse: Belongs To Many

## Polymorphic Relationships

