---
id: 7112
title: '#CASBAN6: the data model explained'
date: '2022-11-11T16:00:00+01:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am going to show and explain the data model of my serverless blog engine.'
layout: post
permalink: /casban6-the-data-model-explained/
image: /assets/img/2022/11/CASBAN6_data_model_title.jpg
categories:
    - 'Dev Stories'
    - Azure
    - Database
tags:
    - .NET
    - .NET6
    - '#CASBAN6'
    - 'data model'
    - database
    - EF
    - 'Entity Framework'
    - 'Entity Framework Core'
    - model
    - SQL
---

### Preface

Initially, this post should have been about the direct implementation and design of the data model for my serverless blog engine. As the data model became a bit more complex, I decided to split the data model post into two posts. The first aims to explain the data model, while the second post is for the implementation with Entity Framework Core.

### The data model

A picture is worth a thousand words, they say. So here is a complete picture of the data model:

<div class="wp-block-image is-style-default"><figure class="aligncenter size-large is-resized">![CASBAN6 data model](https://msicc.net/assets/img/2022/11/ServerlessBlogDBSchema_final-1024x775.png "ServerlessBlogDBSchema.png")</figure></div>I will go through the model table by table and tell you a sentence or two on each of it for the rest of this post.

### The tables

#### \_\_EFMigrationsHistory

This table just stores all the `MigrationId`s and is handled by the Entity Framework. I recommend you to not touch this table.

#### Blogs

In theory, the database could hold more than one blog. By adding a new row to this table, you are creating a new blog in the database. The `BlogId` is essential for a range of other tables.

#### Authors

To be able to issue blog posts, our blog needs at least one author. As you may have more than one person to fill your blog, a collection of authors can be saved within this table. One author can only be assigned to one blog (at least for now, this is intentional).

#### Posts

The content fuelling our blog is in the posts table. One post can only be on one blog. The published date will be set on insert, all subsequent changes will modify the `LastModified` column. Also, one post can only have one author. The author can be replaced on updating the row in the table.

The slug can be used to create a human-readable URL for the post (instead of the `PostId`, which is a `GUID`). The slug must be unique across all blogs.

#### Tags

In order to group and categorize posts, I decided to go with a tags-only approach (unlike other platforms, which allow both categories and tags). Tags are unique to a blog, but can be used in multiple posts.

#### Media

Of course, our blog should support also media content like images, videos and other types. I opted in for a URL-based approach, which is making it easier to add content from other platforms (like videos hosted on dedicated platforms). A medium can be used in several posts.

#### MediaTypes

To make it a bit easier to determine a medium’s type, I added the `MediaTypes` table. It holds information about the MIME-Type and possibly also the encoding of a file. The uniqueness is based off the MIME-Type.

#### Mapping tables

As we learned already above, both tags and media can be used in multiple posts. At the same time, posts can have multiple media and also multiple tags. To cover this many-to-many relationships, I use two mapping-tables with a composite primary key to ensure their uniqueness across the blog.

### Conclusion

In this post, I outlined the data model of my serverless blog engine. In the next post, I will show you the implementation of this model with Entity Framework Core.

#### Until the next post, happy coding, everyone!