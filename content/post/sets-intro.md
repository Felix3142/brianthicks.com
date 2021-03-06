---
date: "2016-11-13T22:30:00-06:00"
title: "Functional Sets, Part 1: Construction"
tags: ["elm", "sets"]
featureimage: "/images/windows-by-samuel-zeller.jpeg"
thumbnail: "/images/windows-by-samuel-zeller-with-title.png"
section: "Technology"

---

Today we're starting a series on building functional data structures using Elm.
The goal: build a native set implementation from first principles.
We're going to reimplement the [standard library's Set API](http://package.elm-lang.org/packages/elm-lang/core/4.0.5/Set).
This is going to be a toy implementation, so we won't optimize as much as we could, but we'll be able to learn some new things!

<!--more-->

## What's In A Set? A Tree!

A set is like a list, but without an order or duplicate elements.
If you put a list `[1, 2, 2, 3]` into a set, you'll get `{1, 2, 3}`.
Or you might get `{2, 1, 3}`, since there is no order.
(Putting the elements of the set in `{braces}` is something I'm borrowing from Python, it just indicates that it's a set instead of a list.)

We're going to implement Sets with [binary search trees](https://en.wikipedia.org/wiki/Binary_search_tree).
A binary search tree containing the numbers 1, 2, and 3 might look like this:

{{< figure src="/images/sets/example.png"
           caption="A binary search tree containing 1, 2, and 3. Empty circles represent empty trees." >}}

An element in this particular data structure has a *head* and two sub-elements, or *sub-trees*.
The *left* tree contains items less than the head, and the *right* tree contains items greater than the head.
If there are no items less than or greater than the head, the trees below it are empty.
In the example above, we have a set containing 2 as the head, and the left and right subtrees are less than and greater than 2, respectively.
But the trees containing 1 and 3 have no sub values, so the trees them are empty.

This all sounds *kiiiiind* of complicated.
Why are we using this particular data structure?
Well, think about this: if a branch contains itself, and items less and greater than itself, then there's only one of every item in the set.
We can figure out if an item is in the set pretty quickly too (in `O(log n)` time), and we'll be doing a lot of that.

## The Data Model

Enough theory!
How does this look in actual code?

```elm
type Set comparable
    = Tree comparable (Set comparable) (Set comparable)
    | Empty
```

This is a type union with two tags: `Tree` and `Empty`.
`Tree` will represent when we have a value in our set, and `Empty` will represent when the set is empty.
We're going to treate these two tags as our base-level data structures.
`Empty` is a valid set, and so is `Tree 1 Empty Empty`.

Our `Set` type takes a `comparable` as part of the type.
This is because we want to be able to store anything in the set.
`comparable` is a typeclass that refers to any type in which we can compare the values.
This includes `Int`, `Float`, `Time`, `Char`, `String`, and tuples or lists of those types.

Let's create some basic constructor functions:

```elm
empty : Set comparable
empty =
    Empty


singleton : comparable -> Set comparable
singleton item =
    Tree item empty empty
```

`empty` gives us, as you might expect, `Empty`.
`singleton` gives us a set containing a single item.
So what do we get when we call `singleton 1`?
`Tree 1 Empty Empty`!
This tells us that the *head* of the set is `1`.
It has no items either greater or less than itself, so both *left* and *right* are empty.
Remember: both `Empty` and `Tree` are valid sets, so our constructor functions have covered all our bases.

{{< figure src="/images/sets/singleton.png"
           caption="A singleton set containing 1." >}}

## Inserting

So we've got sets with no values, singleton sets, but how do we get sets with more than one item?
We'll want a function that will insert a new value into our set, let's call it `insert`.
Let's dive into the code, and I'll explain as we go:

```elm
insert : comparable -> Set comparable -> Set comparable
insert item set =
    case set of
        Empty ->
            singleton item

        Tree head left right ->
            if item < head then
                Tree head (insert item left) right
            else if item > head then
                Tree head left (insert item right)
            else
                set
```

First, we need to determine if the set is empty.
If it is, we can replace it with a singleton!
Otherwise, we need to compare the head with the new item:

- If the item we're inserting is less than the head, we insert into the left set.
- If it's greater, we insert into the right set.
- If the two values are equal, we do nothing.
  The item is already present, so return what we have already!

Whew!
That's a whirlwind of an algorithm!
Recursive algorithms can be hard to understand, so let's visualize what's happening.
Let's say we want to insert `0` into a singleton set containing 1.
We'll call `insert 0 (singleton 1)`:

{{< figure src="/images/sets/insert1.png"
           caption="The singleton from before, and a new value: 0." >}}

We compare to the singleton set in the pattern matching.
Now we're comparing `0` (the item we're inserting) to `1` (the head of the current tree):

{{< figure src="/images/sets/insert2.png"
           caption="Comparing the inserted value to the head." >}}

We find that `0` is less than `1`, so we reconstruct the current tree and insert the new item to the left.
This is the same as calling `insert 0 Empty`, since the left node is empty.

{{< figure src="/images/sets/insert3.png"
           caption="Comparing the inserted value to the left branch." >}}

This time around, the `set` in question is `Empty`, so we'll return a new singleton.
Our new set then bubbles up through the recursive calls.
It ends up replacing the left branch of the original set.
The original call returns with the modified set, and we're finished.

{{< figure src="/images/sets/insert4.png"
           caption="The final tree, with the new value inserted." >}}

## List to Set

Now that we have `insert` taken care of, we can convert lists to sets.
All it takes is `List.foldl`!

```elm
fromList : List comparable -> Set comparable
fromList items =
    List.foldl insert empty items
```

We're using foldl to insert the items from the list one at a time and starting with the empty set.
Conversion function: complete!

## So Far

To sum up: we've got:

- a recursively defined Set data structure
- constructors for both kinds of base values (empty sets and singleton sets)
- two ways to insert values into sets (`insert` and `fromList`, which folds a list using `insert`)

We've got a good foundation, but we have quite a way to go.
Next week we're going to look at how to tell if a set contains a value.
We'll also cover how to measure how many values are in our set.
Stay tuned!

{{< setsSeriesSignup >}}
