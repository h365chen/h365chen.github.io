---
layout: post
title: Look Here or Look Across
date: 2025-03-30
description: About interpreting cardinality constraints in UML and ER designs
categories: notes
thumbnail: assets/img/posts/2025-03-30-intro.png
giscus_comments: true
related_posts: false
pretty_table: false
citation: true
toc:
  sidebar: left
---

## Introduction

Entity-Relationship (ER) Modeling is one of the essential topics covered in
database courses. It aims to equip students with the necessary knowledge to
transform business requirements into database schemas, which is a crucial step
in database design.

One of the key aspects of ER modeling is understanding how to model cardinality
constraints based on business requirements. For example, when designing a
database to store employees' information for a university, the business
requirements might be:

1. A person can be assigned to at most two departments at a time.
2. For each assigned department, a person cannot work on more than one
   project.
3. Every project must be associated with at least one department, with at least
   one person working on it.

It is important to ensure these constraints are enforced in the database, as it
becomes difficult to enforce them once real data has been populated.

Correctly understanding cardinality constraints is critical. However, I have
frequently encountered unclear cardinality constraints in ER diagrams drawn by
students. Nine out of ten students fail to interpret them correctly, and the
actual number may be even worse.

Why is that?

To my surprise, the issue is not that students don't understand the concept.
Rather, it's because there are **_TWO_** interpretations of cardinality
constraints in ER diagrams, and these two interpretations are **_EXACT
OPPOSITES_** of each other! These conflicting interpretations are scattered
randomly across the internet, so it's no wonder students are confused.

## Instructor to Student

Let's assume we have two entity sets, `Instructor` and `Student`, and a
relationship set `advise`. Suppose they form the following design, in which `M`
indicates `1..*` (many & total participation), and `1` indicates `1..1` (one &
total participation).

How should we interpret the design? Is it many-to-one, or one-to-many?

{%
    include figure.liquid
    loading="eager"
    path="assets/img/posts/2025-03-30-intro.png"
    caption="A sample diagram"
    class="img-fluid rounded z-depth-1"
%}

Many-to-One?

This interpretation is very intuitive since from left to right, the letters can
be read as "M-to-1".

But wait, the semantics seem confusing. Why can there be _many_ (_i.e._, an
unlimited number of) instructors who `advise` _one_ student?

This interpretation does not make sense.

Well, if that interpretation is not correct, then should the correct
interpretation be:

One-to-Many?

## Entity-Relationship (ER) & Unified Modeling Language (UML)

**ER**, standing for _Entity-Relationship_, is a modeling technique proposed by
[Chen in 1976](https://dl.acm.org/doi/10.1145/320434.320440). It was first
proposed as an alternative technique for modeling data, intended to be more
intuitive than directly using the relational model. While people today often
think in terms of relational models (such as SQL tables) for data management, ER
modeling remains very useful design and is now the _de facto_ standard for
database schema.

**UML**, the Unified Modeling Language, proposed in 1990s, aims to provide a
standardized way to visualize the design of systems, including structure,
behavior, and interactions. While it is primarily used in object-oriented
software development, UML is also commonly used for ER diagrams due to its
similar way of representing entities, attributes, and relationships.

It's not surprising that students often use UML tools to draw ER diagrams.
However, this can lead to confusion due to differing interpretations of
cardinality constraints. We'll soon see why.

## It is Many-to-One in UML or Chen's ER

UML refers to cardinality constraints as _association rules_. In UML's
interpretation, the diagram above is **many-to-one**. This is intuitive.

Interestingly, this is also the interpretation in Chen's ER model.

## It is One-to-Many in the Textbook's ER

However, in the ER model taught by textbooks like [_Database System
Concepts_](https://www.db-book.com), the interpretation is **one-to-many**.

The rationale is based on how ER diagrams are eventually translated into
relational schemas. To make this translation more explicit, it helps to think
this way:

- Each instructor can appear in `advise` up to `M` times;
- Each student can appear in `advise` only once.

The resulting relation, _e.g._, the `Advise` SQL table, would then look like
this. Note that there must be no duplication, as it is a relational model:

|  Instructor  |  Student  |
| :----------: | :-------: |
| Instructor 1 | Student 1 |
| Instructor 1 | Student 2 |
| Instructor 1 | Student 3 |
|     ...      |    ...    |

This is the stage where we check whether the cardinality constraint is
satisfied. Looking at the relational schema, it clearly reflects a one-to-many
relationship.

## Look Here or Look Across

This distinction remains important even when there is only the `M` shown.

{%
    include figure.liquid
    loading="eager"
    path="assets/img/posts/2025-03-30-only-m.png"
    caption="With only one cardinality constraint"
    class="img-fluid rounded z-depth-1"
%}

The `M` can be interpreted as:

- (UML or Chen's ER): For each _student_, there can be up to `M` _instructors_
- (Textbook's ER): For each _instructor_, there can be up to `M` _students_

This is what people refer to as **look here** and **look across**. It is "look
here" in UML because the `M` is read along with the entity set on the **same**
side, `Instructor` in this case; It is "look across" in the textbook's ER,
because the `M` is interpreted relative to the entity set on the **opposite**
side, `Student`.

## N-ary Relationships

"Look here" and "look across" are two useful techniques for interpreting
cardinality constraints. They can also be applied to N-ary relationships.

Assume we have the following diagram:

{%
    include figure.liquid
    loading="eager"
    path="assets/img/posts/2025-03-30-n-ary.png"
    caption="N-ary relationships"
    class="img-fluid rounded z-depth-1"
%}

Then we interpret it as:

- (UML or Chen's ER): For each (person, department) tuple, there can be many
  projects
- (Textbook's ER): For each project, there can be many (person, department)
  tuples

## Discussion

Referring to the cardinality constraints at the beginning of the post:

1. A person can only be assigned to at most two departments at a time
2. For each assigned department, a person cannot work on more than one project
3. Every project must be associated with at least one department, with at least
   one person working on it

How do we represent these constraints using numbers on the diagram?

I'll explain that in a future post.
