---
date: '2025-05-13'
lastmod: '2025-05-13'
draft: false
title: "Hidden Performance Trap in Rails: Avoid nil in .where(id: [...])"
slug: 'rails-where-id-nil-performance-trap'
authors: ['Alex']
enableReadingTime: true
description: "Accidentally including nil in a .where(id: [...]) clause in Rails can drastically slow down PostgreSQL queries — or even trigger ActiveRecord::QueryCanceled. Learn why, and how to prevent it."
keywords: ["rails performance", "postgresql slow query", "activerecord querycanceled", "nil in where clause", "where id in nil", "rails best practices"]
tags: ['rails', 'postgresql', 'performance', 'activerecord']
categories: ['Post']
hiddenFromHomePage: false
featuredImage: 'rails-nil-performance-trap.png'
---

# ⚠️ Why Including `nil` in `.where(id: [...])` Can Severely Impact PostgreSQL Performance

When writing queries like:

``` ruby
User.where(id: [1, 2, nil, 3, 4])
```

it’s easy to accidentally include a `nil` in the array. Although it may seem harmless, this can have **serious performance implications**, especially on large datasets — and can even cause **query timeouts**.

---

## 🔍 What Happens Behind the Scenes?

ActiveRecord transforms the query to something like this:

``` sql
SELECT * FROM users WHERE id IN (1, 2, 3, 4) OR id IS NULL;
```

PostgreSQL requires the `OR id IS NULL` part because `NULL` cannot be directly compared using `IN (...)`.

---

## 📉 Performance Impact

This small change completely alters the query plan:

- Instead of a fast `Index Scan`, PostgreSQL may use `BitmapOr` and `Bitmap Heap Scan`
- It must separately evaluate the `id IS NULL` condition
- On large tables, this often leads to **sequential scans**

Even if **no NULLs exist in the table**, the planner still has to check.

---

## 🧨 Real-World Example: Query Timeout

This issue occurred on database table with **over 100 million rows**.  
A query that should have completed instantly began taking several seconds — and eventually **failed** with:

```
ActiveRecord::QueryCanceled
```

This happened because the database exceeded the `statement_timeout` due to the inefficient plan triggered by the presence of `nil` in the `id` list.

---

## ✅ Recommended Practice

Always sanitize your arrays before using them in `.where(id: ...)`:

``` ruby
User.where(id: ids.compact)
```

Or validate explicitly:

``` ruby
raise ArgumentError, "ID list contains nil!" if ids.any?(&:nil?)
```
