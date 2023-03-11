---
title: The ID chosen
date: 2023-03-11 10:30:00 +0700
categories: [Solutioning]
tags: [database]
---

The identity property of a database record, the ID, is the most important property to consider when designing your database. Without it, you may not be able to find the exact record you want, or may experience poor performance when looking up that record. Although it is critical, many of us are not taught to choose it carefully. I remember being taught in class to let the database generate the ID using some functions, and not to worry about it. And I followed that advice for a long time before realizing how important it is to consider the ID and how it can significantly affect my database design decisions.

Today, I will introduce to you some types of IDs that I used to use and explain my favorite choice.

## Auto-incremented ID

The Auto-incremented ID is a feature offered by most SQL databases, where a value is automatically incremented and used as the primary or ID column. This method is simple, straightforward, and powerful.

### Advantages

- The size of ID column is small as it is just a number ([8 bytes for BIGINT](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html) compared to [255 bytes for VARCHAR(255)](https://dev.mysql.com/doc/refman/8.0/en/char.html))
- The value of ID is reflected to the timestamp ordered. That making it useful for implementing cursor pagination, which requires a unique, orderable column
- The method is simple to implement since the database generates the value and ensures it is unique.

### Disadvantage

- Exposing a public API that returns auto-incremented IDs may make it easy for scanning crawlers to collect your data. For example: [Shopee Shop details API](https://shopee.vn/api/v4/product/get_shop_info?shopid=807592281)
- You cannot perform a bulk insert with related records without retrieving parent records from the database. For example, if you need to extract payment information and insert both the payment and the transaction it belongs to in one request, you cannot do so directly. Instead, you must insert the transaction first, retrieve the returning ID, and then use it to refer to the payment transaction column and insert the payment into the database.
- May not work well with distributed system which contains multiple nodes. Because you may encoutered two conflicted it with auto-incremented value if you generate it in different node at the same time.

## The UUID

The UUID (Universally unique identifier) is another choice for primary column. Is it unqiue? Yes. Is it simple to implement? Yes, you can generate it in your code or delegate it to database. Does it reflect to the timestamp ordered? Unfortunately, no.

The advantages are straightforward: you have a unique ID that allows you to perform bulk modify queries with related records directly, and it is hard to conflict when generated on different nodes at the same time

However, you lose the ability to use the ID column to implement cursor pagination, and the storage size is larger than the auto-incremented ID