---
layout: post
title: Unified Cardinality Constraints
date: 2025-09-10
description: About interpreting cardinality constraints in UML and ER designs
categories: notes
thumbnail: assets/img/posts/2025-03-30-intro.png
giscus_comments: true
related_posts: false
pretty_table: false
citation: false
toc:
  sidebar: left
_styles: |
  p.aside {
    font-size: 0.875em;
    margin-top: 2rem;
  }
  table {
    margin-bottom: 2rem;
  }
---

## Introduction

In the [previous post]({% post_url 2025-03-30-Look-Here-Look-Across %}), I wrote about the two
readings of a cardinality constraint in an ER diagram: **look here**, as in UML and Chen's ER,
and **look across**, as in the textbook's ER. The two readings are exact opposites of each
other, and which one you end up with depends on the convention that the person who drew the
diagram had in mind. That post promised to explain how a set of business requirements would be
written down on a diagram. I will get to that, but there is a complication to deal with first.

That post was about a relationship between exactly two entity sets. For a relationship among
three or more entity sets, the picture is in worse shape than ambiguous: it has nowhere to put
most of the constraints we care about. This post is about a notation that takes care of both
problems at once.

## How Many Cardinality Constraints Are There?

Consider a ternary relationship set `guide` among `Instructor`, `Project`, and `Student`:

{%
    include figure.liquid
    loading="eager"
    path="assets/img/posts/2025-09-10-guide-ternary.png"
    caption="A ternary relationship set"
    class="img-fluid rounded z-depth-1"
%}

A cardinality constraint says something of the form "for each X, the number of Y is between
_lower_ and _upper_", where X and Y are disjoint groups of the entity sets participating in the
relationship. With three entity sets there is more than one way to choose those groups, so let
us enumerate them. Since every constraint below is on `guide`, I write `p → q` as a shorthand
for `Card(guide; p; q)`. The twelve constraints come in _six pairs_: the right column of each
row is the left column with `p` and `q` exchanged.

| Constraint                          | Constraint (reversed)               |
| :---------------------------------- | :---------------------------------- |
| `{Instructor} → {Project}`          | `{Project} → {Instructor}`          |
| `{Instructor} → {Student}`          | `{Student} → {Instructor}`          |
| `{Project} → {Student}`             | `{Student} → {Project}`             |
| `{Instructor, Project} → {Student}` | `{Student} → {Instructor, Project}` |
| `{Instructor, Student} → {Project}` | `{Project} → {Instructor, Student}` |
| `{Project, Student} → {Instructor}` | `{Instructor} → {Project, Student}` |

_Where does the number twelve come from? A constraint has to name a non-empty group `p` to
iterate over and a non-empty group `q`, disjoint from `p`, to count. Choosing `p` to be one
entity set leaves `2^2 - 1 = 3` choices for `q` — the subsets of the two remaining entity sets,
minus the empty one — and there are three such choices of `p`, for nine constraints. Choosing
`p` to be two entity sets leaves `2^1 - 1 = 1` choice for `q` — the subsets of the single
remaining entity set, minus the empty one — and there are three such choices of `p`, for three
more. `p` cannot be all three entity sets, because then nothing is left to count. That gives
`9 + 3 = 12`._
{: .aside }

Now, the diagram has one edge per entity set, and each edge holds exactly one label. So the
picture can carry at most three of these twelve constraints, and for each edge, _which_ member
of the pair you get is decided by the convention. Write `M` on the `Student` edge, and UML
reads it as `Card(guide; {Instructor, Project}; {Student}) = (1, M)`, while the textbook reads
the same `M` as `Card(guide; {Student}; {Instructor, Project}) = (1, M)`. You cannot express
both at once, and a reader who assumes the other convention takes away the opposite of what you
meant.

## Three Ways to Deal with Cardinality Constraints in an N-ary Relationship

As far as I know there are three ways out of this:

1. Split the n-ary relationship into multiple binary relationships ❌
2. Add an artificial entity set ❌
3. Represent the cardinality constraints non-graphically ✅

The first two are the usual workarounds, and both of them cost more than they look like they
do. Splitting a ternary relationship into binary ones changes what the design means: three
binary relationship sets can record three pairwise facts without ever recording that the three
belong together. An artificial entity set has to be given an identity that the domain does not
provide — its key is something we invent rather than something we discover — and it only pushes
the problem one level down.

The third one is what the rest of this post is about.

## Generalized Cardinality Constraints

Instead of a mark on an edge, write the constraint as a function:

`Card(R; p; q) = (lower, upper)`

- For each `p`, the number of tuples from `q` is between `lower` and `upper`.
- `R` is the relationship set.
- `p` and `q` are disjoint sets of entity sets.

For the ternary example above, the two conventions become two lines:

| Convention                       | Constraint                                   |
| :------------------------------- | :------------------------------------------- |
| UML or Chen's ER (**look here**) | `{Instructor, Project} → {Student} = (1, M)` |
| Textbook (**look across**)       | `{Student} → {Instructor, Project} = (1, M)` |

_The two readings differ **only** in which side is named as `p`. Same picture, same numbers,
exchanged arguments. This is the whole disagreement from the previous post: there, the
many-to-one reading was `Card(advise; {Student}; {Instructor}) = (1, M)` and the one-to-many
reading was `Card(advise; {Instructor}; {Student}) = (1, M)` — again the same two numbers, with
`p` and `q` the other way around._
{: .aside }

That is why I call this notation _unified_. It does not force you to pick a side of the
look-here/look-across debate, and it does not care whether the relationship set is binary or
n-ary. Since every constraint names its own `p` and `q`, there is nothing left to guess: you
write the reading you mean, and a reader — or a program — gets exactly that one.

## Encoding Generalized Cardinality Constraints

Since `Card(R; p; q)` is now a mathematical object rather than a mark on a picture, it can be
written down directly. Because the bounds are numbers, the unbounded `M` has to be replaced by
a concrete value; assume `M = 10` throughout, and note that an upper bound which is genuinely
unbounded is written `*`. The two conventions of the previous section then become two lines,
which can sit next to each other without contradiction because each one names its own `p` and
`q`:

- `Card(guide; {Instructor, Project}; {Student}) = (1, 10)`
- `Card(guide; {Student}; {Instructor, Project}) = (1, 10)`

The real payoff is that we are no longer limited to one constraint per edge. The same
relationship set can carry four constraints at once:

- `Card(guide; {Student}; {Instructor, Project}) = (1, 10)`
- `Card(guide; {Instructor}; {Project}) = (0, 2)`
- `Card(guide; {Instructor}; {Student}) = (1, 5)`
- `Card(guide; {Student}; {Project}) = (1, *)`

Read out, this says that each student is guided on 1 to 10 (instructor, project) combinations,
that each instructor works on at most two projects, that each instructor is associated with 1
to 5 students, and that each student works on at least one project. Three of those four
constraints have no place on the diagram at all, and the fourth is the one whose reading you
would otherwise have had to guess.

Note also that these constraints do not all follow the same convention, and that this no longer
matters. Each one names its own `p` and `q`, so it says what it means on its own.

## What's Next

Writing the constraints down as data is also what makes them something a program can read,
which is where I want to go from here: implicit cardinality constraints, the constraints that
follow for free from the ones you wrote; inference rules such as decomposition and augmentation;
and, at last, the three business requirements left over from the introduction of the previous
post.
