---
date: "2016-11-27T20:25:00-06:00"
title: "Functional Sets, Part 3: Balancing"
tags: ["elm", "sets"]
featureimage: "/images/balanced-stones-by-austin-neill.jpg"
thumbnail: "/images/balanced-stones-by-austin-neill-with-title.png"
section: "Technology"

---

In this series, we've already [implemented sets, convenient ways to create them]({{< ref "sets-intro.md" >}}) and [rotation]({{< ref "sets-balancing-rotation.md" >}}).
Next, we'll make our trees use those rotation functions!

<!--more-->

## Rotation Recap

When we rotate a tree, we move the subtrees and heads around so that the tree has a different root.
We do this in such a way that we preserve the order in the tree.
It looks like this:

{{< figure src="/images/sets/Tree_rotation_animation_250x250.gif"
           caption="A Tree Rotating Left and Right"
           attr="Wikipedia"
           attrlink="https://en.wikipedia.org/wiki/File:Tree_rotation_animation_250x250.gif" >}}

[Last time we created functions]({{< ref "sets-balancing-rotation.md" >}}) (`rotl` and `rotr`) to do this rotation for us.
Now, let's use them!

## Our Trigger: Height Difference

Let's talk about what we need to do to automatically balance our tree.
We measure the balance of a tree as the height of the right subtree minus the height of the left subtree.
A tree is a valid AVL tree if the height is -1, 0, or 1.
Since any value outside these three is an invalid AVL tree, we'll use it as our trigger to balance.

We can get the height difference for a set with a new function:

```elm
diff : Set comparable -> Int
diff set =
    case set of
        Empty ->
            0

        Tree _ _ left right ->
            height right - height left
```

If the tree is empty, the difference is zero.
Otherwise, we subtract the height of the left subtree from the height of the right subtree.
This means that if the tree leans to the right, the value will be positive.
If the value is negative, the tree is leaning to the left.

Remember that we're talking about *height* here, not the number of nodes in the tree.
This is OK, though, because we still get that `O(log n)` operation guarantee!

## The Simplest Case: 2 or More

To balance our tree, we'll first need to handle if the height difference is greater than one.
(More precisely, we catch the difference at exactly two; we'll never let it get above that.)

Say we insert the letters A through C, in order.

{{< figure src="/images/sets/a-through-c-unbalanced.png"
           caption="A through C, leaning right" >}}

The difference here is 2 since the height of the right subtree is two and the height of the left subtree is 0.
Since the tree is leaning right we'll rotate left, so that B is at the root.

{{< figure src="/images/sets/a-through-c-balanced.png"
           caption="A through C, balanced" >}}

Now the height of the left and the right subtrees is the same, so the difference is 0.

We'll implement our initial balancing function like this:

```elm
balance : Set comparable -> Set comparable
balance set =
    case set of
        Empty ->
            set

        Tree _ head left right ->
            if diff set < -1 then
                rotr set
            else if diff set > 1 then
                rotl set
            else
                set
```

An empty set is already balanced, so we return it immediately.
Remember that a non-empty set needs balancing if the difference in height between the left and right subtrees is greater than 1.
If we're leaning right (a positive number) we'll rotate left, and if we're leaning left (a negative number) we'll rotate right.

We'll also change `insert` to call `balance` after inserting:

```elm
insert : comparable -> Set comparable -> Set comparable
insert item set =
    case set of
        Empty ->
            singleton item

        Tree _ head left right ->
            if item < head then
                tree head (insert item left) right |> balance
            else if item > head then
                tree head left (insert item right) |> balance
            else
                set
```

This will balance the tree depth-first, since we call `insert` recursively.
Notice that we're only balancing the tree after insertion.
Since we're doing that, we don't need to balance when an item is already in the set.
We also don't need to balance the singleton set, which is already balanced.

You can try this out in the embedded example below.

## Left- and Right-Leaning Sets

There's one more shape we need to consider.
What if we inserted A, C, then B?

{{< figure src="/images/sets/a-c-b-unbalanced.png"
           caption="A through C, unbalanced but with a \"between\" value (B)" >}}

The difference would be 2, but if we rotated right it would swap to -2!
In this case, we need to rotate the subtree right, then the main tree left.
The right rotation will create our unbalanced set from above, then the left rotation will move B to the root.
If the tree were leaning the other way, we'd rotate the subtree left, then the main tree right; it's always the opposite of the "main" rotation.

The code looks like this:

```elm
balance : Set comparable -> Set comparable
balance set =
    case set of
        Empty ->
            set

        Tree _ head left right ->
            if diff set == -2 && diff left == 1 then
                -- left leaning tree with right-leaning left subtree. Rotate left, then right.
                tree head (rotl left) right |> rotr
            else if diff set < -1 then
                -- left leaning tree, generally. Rotate right.
                rotr set
            else if diff set == 2 && diff right == -1 then
                -- right leaning tree with left-leaning right subtree. Rotate right, then left.
                tree head left (rotr right) |> rotl
            else if diff set > 1 then
                -- right leaning tree, generally. Rotate left.
                rotl set
            else
                -- diff is -1, 0, or 1. Already balanced, no operation required.
                set
```

Whew!
That's quite a few conditionals!
The comments describe what's happening in each.
Generally, we're examining the difference in each tree, and the difference in the relevant subtrees.
If we find that the difference is 2 (or -2) and the difference in the subtree is -1 (or 1) we've hit our problem.
We send the subtree to `rotl` or `rotr` before rotating the whole tree in question.

It looks sort of messy, especially with the comments, but this is the least code that satisfies every condition to keep an AVL tree balanced.
We're done!

## Make Your Own Trees, Again

Now you can try this out for yourself.
This is the same visualization from last time, but with balancing turned on.
Insert values to see how the balancing code reacts and keeps everything tuned up.

Things to play around with:

- Insert three values in order.
  They get reblanced!
- Insert three values so that you would get a leaning tree.
  Check out how the code fixes it.
- Insert a bunch of values.
  AVL trees are height-balanced, not weight balanced.
  This means that depending on the order you insert you can get trees with vastly different weights, but always close heights.

{{< elmEmbed src="/scripts/sets/insertBalanced.js" name="InsertBalanced" >}}

{{< setsSeriesSignup >}}
