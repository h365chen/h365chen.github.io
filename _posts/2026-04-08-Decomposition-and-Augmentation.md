---
layout: post
title: Decomposition and Augmentation
date: 2026-04-08
description: About deriving implied cardinality constraints from the one that is written
categories: notes
thumbnail: assets/img/posts/2025-03-30-intro.png
giscus_comments: true
related_posts: false
pretty_table: false
citation: false
toc:
  sidebar: left
_styles: |
  table {
    margin-bottom: 2rem;
  }
---

## Introduction

The [previous post]({% post_url 2026-02-10-Implicit-Cardinality-Constraints %}) started from one
written cardinality constraint on the ternary relationship set `guide`, and ended with four that
follow from it. This post is about where those four come from: the two rules that produce them,
what it means for one bound to follow from another, and what the notation has to do with
functional dependencies.

The constraint numbers below are that post's. Constraint 10, `{Student} → {Instructor, Project}`
with bound `(2, 2)`, is the written one; 5 and 6 are `{Instructor, Student} → {Project}` and
`{Project, Student} → {Instructor}`; 8 and 9 are `{Student} → {Instructor}` and
`{Student} → {Project}`.

## Decomposition and Augmentation

We didn't find the four implied constraints by staring. They came from two rules, and it's worth
naming them, because a program needs them by name.

**Decomposition** relates a group to a larger group that contains it. Each `q`-value a tuple has
comes with at least one `s`, so the number of `q`s can never exceed the number of `(q, s)` pairs:

$$
\begin{aligned}
C_{\min}(R; p; q \cup s) &\geq C_{\min}(R; p; q) \\
C_{\max}(R; p; q \cup s) &\geq C_{\max}(R; p; q)
\end{aligned}
$$

Knowing a bound on `p` over the pair-group $q \cup s$, we can therefore bound `p` over the
smaller group `q`. Constraint 10 is a bound on `{Student}` over `{Instructor, Project}`, so
decomposing gives bounds on `{Student}` over `{Instructor}` and over `{Project}`: constraints 8
and 9, `(1, 2)` each. The maximum carries over unchanged. The minimum can drop, and does: the
two pairs may name the same instructor. It can't drop below 1, because a student who has a pair
at all has an instructor.

**Augmentation** relates a group to a larger group that contains it on the other side. An
`(p, s)`-pair determines a `p`, and the `q`s available to the pair are among those available to
that `p`. So anything true of every `p` is true of every `(p, s)` as well, and the bound tightens
with the requirement:

$$
\begin{aligned}
C_{\min}(R; p; q) &\leq C_{\min}(R; p \cup s; q) \\
C_{\max}(R; p; q) &\geq C_{\max}(R; p \cup s; q)
\end{aligned}
$$

Constraint 10 is a bound on `{Student}`, so it's also a bound on `{Instructor, Student}` and on
`{Project, Student}`: constraints 5 and 6, each `(0, 2)`. The upper bound of 2 is inherited from
constraint 10. The lower bound stays at the default of 0, because nothing in the design requires
a given instructor and student to be working on anything together.

The two rules lean in opposite directions and both lose information. Decomposition shrinks the
group being counted; augmentation shrinks the group being iterated over. Neither ever produces a
bound tighter than the default on its own authority: the minimum of a derived constraint is 1
only when the constraint it came from forces it to be.

## Implied, Redundant, or Contradictory

Let's be precise about what "follows from" means here, because the same comparison decides three
different questions.

Bounds are intervals, so a constraint `Card(R; p; q) = (l, u)` is contained in another,
`(l', u')`-shaped one, exactly when `l ≥ l'` and `u ≤ u'` (_i.e._, when its interval sits inside
the other). The contained constraint is the stronger statement, since it rules out more
instances. Two constraints on the same `(R; p; q)` combine to `(max(l, l'), min(u, u'))`, and the
pair is contradictory exactly when that interval is empty.

With that in hand:

- A constraint is **implied** when the rest of the design already forces something at least as
  strong.
- It is **redundant** when it is implied and adds nothing.
- The design is **contradictory** when two written constraints have no interval in common.

Two bounds that merely differ are neither. A design that says `(1, 5)` for constraint 8, when
constraint 10 forces `(1, 2)`, isn't in conflict with itself; it just has a loose bound in it,
which is the sort of thing a marker would want to know and a program can point out for free.

## One Tuple

Let's change the one constraint that is written, and the whole set of bounds changes with it.

Suppose constraint 10 is `(1, 1)` instead of `(2, 2)`: each student is guided on exactly one
(instructor, project) pair. Then constraints 5, 6, 8, and 9 all become `(0, 1)` (at most one,
with no lower bound), and the rest stay at the default.

| #   | Constraint                          | Bound            |
| :-- | :---------------------------------- | :--------------- |
| 5   | `{Instructor, Student} → {Project}` | `(0, 1)` implied |
| 6   | `{Project, Student} → {Instructor}` | `(0, 1)` implied |
| 8   | `{Student} → {Instructor}`          | `(0, 1)` implied |
| 9   | `{Student} → {Project}`             | `(0, 1)` implied |
| 10  | `{Student} → {Instructor, Project}` | `(1, 1)` given   |

Written the other way, this is a derivation everyone has seen before:

```
S = Student, I = Instructor, P = Project

S → IP          (given)
⇒ S → I, S → P  (decomposition)
S → P ⇒ IS → P  (augmentation)
S → I ⇒ PS → I  (augmentation)
```

A constraint of the form `Card(R; p; q) = (0, 1)` says that each `p` determines at most one `q`,
which is a functional dependency `p → q`. Raising the lower bound to 1 adds a requirement that
each `p` has a `q` at all, which is total participation rather than part of the dependency. Drop
the cardinality notation and the derivations above are the standard decomposition and
augmentation rules for functional dependencies, which I find reassuring: the notation of
[Unified Cardinality Constraints]({% post_url 2025-09-10-Unified-Cardinality-Constraints %})
wasn't invented so much as recovered.

Cardinality constraints and functional dependencies remain different classes of business rule.
Cardinality constraints live in the ER model and carry application semantics: that every project
has at least one department behind it is a statement about a relationship set, not about the
tuples of any relation, and no functional dependency expresses it. Functional dependencies live
in the relational model, and normalization is about them. Combining the two is what
[Link and Wei](https://doi.org/10.1145/3448016.3459238) and
[Link et al.](https://doi.org/10.1016/j.is.2023.102208) do, to reach a normal form with update
inefficiency and join efficiency quantified. That's a different post.
