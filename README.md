# idiomorph

idiomorph is a javascript library for morphing one DOM tree to another.  It is inspired by other libraries that 
pioneered this functionality:

* [morphdom](https://github.com/patrick-steele-idem/morphdom) - the original DOM morphing library
* [nanomorph](https://github.com/choojs/nanomorph) - an updated take on morphdom

Both morphdom and nanomorph use the `id` property of a node to match up elements within a given set of sibling nodes.  When
an id match is found, the existing element is often not removed from the DOM, but is morphed in place to the new content.
This preserves the node in the DOM, and allows state (such as focus) to be retained. However, in both these algorithms,
the structure of the _children_ of sibling nodes is not considered when morphing two nodes: only the ids of the nodes are
considered.  This is due to performance considerations: it is not feasible to recurse through all the children of 
siblings when matching things up. 

idiomorph takes a slightly different approach: before node-matching occurs, both the new content and the old content
are processed to create _id sets_, a mapping of elements to _a set of all ids found within that element_.  That is, the
set of all ids in all children of the element, plus the elements' id, if any.

id sets can be computed relatively efficiently via a query selector + a bottom up algorithm.

Given an id set, you can now adopt a broader sense of "matching" than simply using id matching: if the intersection between
the id sets of element 1 and element 2 is non-empty, they match.  This allows idiomorph to relatively quickly match elements
based on structural information from children, who contribute to a parents id set, which allows for better overall matching
when compared with id-based matching.

## Usage

idiomorph has a very simple useage:

```js
  Idiomorph.morph(existingNode, newNode);
```

This will morph the existingNode to have the same structure as the newNode.  Note that this is a destructive operation
with respect to both the existingNode and the newNode

## Performance

Idiomorph is not intended to be as fast as morphdom or nanomorph.  Rather, its goals are:

* Better DOM tree matching
* Relatively simple code

Performance is a consideration, but better matching is the reason idiomorph was created

## Example Morph

Here is a simple example of some HTML in which idiomorph does a better job of matching up than morphdom:

*Initial HTML*
```html
<div>
    <div>
        <p id="p1">A</p>
    </div>
    <div>
        <p id="p2">B</p>
    </div>
</div>
```

*Final HTML*

```html
<div>
    <div>
        <p id="p2">B</p>
    </div>
    <div>
        <p id="p1">A</p>
    </div>
</div>
```

Here we have a common situation: a parent div, with children divs and grand-children divs that have ids on them.  This
is a common situation when laying out code in HTML: parent divs often do not have ids on them (rather they have classes,
for layout reasons) and the "leaf" nodes have ids associated with them.

Given this example, morphdom will detach both #p1 and #p2 from the DOM because, when it is considering the order of the
children, it does not see that the #p2 grandchild is now within the first child.

idiomorph, on the other hand, has an _id set_ for the (id-less) children, which includes the ids of the grandchildren.
Therefore, it is able to detect the fact that the #p2 grandchild is now a child of the first id-less child.  Because of
this information it is able to only move/detach _one_ grandchild node, #p1.  (This is unavoidable, since they changed order)

So, you can see, by computing id sets for nodes, idiomoroph is able to achieve better DOM matching, with fewer node 
detachments.