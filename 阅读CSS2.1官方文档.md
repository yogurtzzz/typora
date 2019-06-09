# CSS Processing Model

User agents(通常是，浏览器)渲染分以下步骤



1. Parse HTML document and create *document tree*
2. Identify media type
3. Retrieve all CSS associated and specified for the target media type
4. Annotate every element of the *document tree* by assigning value to every property
5. From the annotated *document tree* ,generate a *formatting structure* which closely resembles the *document tree* (this phase does not alter the *document tree*)
6. Transfer the *formatting structure* to the target medium (e.g. print the results; display them on screen; render them as speech..etc)



# Visual Formatting Model

i.e.  How user agents process the *document tree* for *visual media*



## Viewport

## Containing blocks

In css2.1, many box positions and sizes are calculated with respect to the edges of a rectangular box called a *containing block* 

We say that a box "establishes" the *containing block* for its descendants.

The phrase "a box's containing block" means "the containing block in which the box lives" , not the one it generated.



e.g.

```html
<div class="dad">
	<div class="son">
        Son
    </div>
</div>
<!-- we say that son's containing block is dad -->
```



## Controlling box generation

This section describes the **types of box** , which may affects its behavior in the *visual formatting model*, in part. The *display* property , specifies a box's type.

### Block-level elements

The following values of *display* property make an element **block-level**: `block`,`list-item`,`table`



*Block-level boxes* are boxes that participate in a [*block formatting context*][#BFC]  [^2]



A *block-level box* is also a *block container box*. But a *block container box* could be a *non-block-level box*[^1].

A *block container box* either contains **only** *block-level boxes*  **or** establishes an *inline formatting context* and thus contains **only** *inline-level boxes*.



[^1]:   Non-replaced inline blocks and non-replaced table cells are block containers but not block-level boxes. 
[^2]: 其实这句话不一定对，设想一个父div，其中包含几个子div，此时父div内部的环境应该不是BFC，所以此时子div所处的环境并不是BFC，而（可能）是IFC

*Anonymous block boxes*

e.g.

```html
<div>
	Hi,this is
    <p>
        Some test text
    </p>
</div>
```

See the example above, the `DIV` seems to have both *inline content*  and *block content*.To make it easier to define the formatting, we assume that there is an *anonymous block boxes* 

![1550918936540](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550918936540.png)

That is to say, if a *block container box* (`DIV` element above) , has a *block-level box* inside it (`P` element above),then we force it to have **only** *block-level boxes* inside it.



When an inline box contains an in-flow block-level box,the inline box will be broken around the block-level box, one box on each side,which are enclosed in *anonymous block box* 

```html
<style>
	p.te{
	display: inline;
	background:lightblue;
}
span.te{
	display: block;
	background: lightgreen;
}
</style>

<p class="te">
	This is a span below
		<span class="te">
			I am the span
		</span>
	This is a span above
</p>
<span>Another span</span>
```



![1550919640040](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550919640040.png)



### Inline-level elements

They are those elements that do not form new blocks of content. The content is distributed in lines.

The following values of *display* property make an element *inline-level*: `inline`,`inline-block`,`inline-table`.

Inline-level elements generate *inline-level boxes*, which participate in an *inline formatting context*  (IFC)





### Run-in boxes(CSS3)

`display:run-in;` is now defined in CSS3



### Display property(CSS2)

The values of this property have the following meanings:

* **block**: This value causes an element to generate a block box.
* **inline**:  This value causes an element to generate one or more inline boxes.
* **inline-block**:This value causes an element to generate an inline-level block container. The inside of an inline-block is formatted as a block box, and the element itself is formatted as an *atomic inline-level box*.
* **list-item**:  This value causes an element (e.g., LI in HTML) to generate a principal block box and a marker box.
* **none**:  This value causes an element to not appear in the [formatting structure] (i.e., in visual media the element generates no boxes and has no effect on layout). Descendant elements do not generate any boxes either; the element and its content are removed from the formatting structure entirely. This behavior **cannot** be overridden by setting the 'display' property on the descendants.

* ........



## Position Schemes

In CSS 2.1, a box may be laid out according to three positioning schemes:

1. **Normal flow**. In CSS 2.1, normal flow includes *block formatting* of block-level boxes, *inline formatting* of inline-level boxes, and *relative positioning* of block-level and inline-level boxes.
2. **Float.** In the float model, a box is first laid out according to the normal flow, then taken out of the flow and shifted to the left or right as far as possible. Content may flow along the side of a float.
3. **Absolute positioning**. In the absolute positioning model, a box is removed from the normal flow entirely (it has no impact on later siblings) and assigned a position with respect to a containing block.

An element is called *out of flow* if it is floated, absolutely positioned, or is the root element. An element is called *in-flow* if it is not *out-of-flow*. The *flow* of an element A is the set consisting of A and all in-flow elements whose nearest *out-of-flow* ancestor is A.



### Normal flow

Boxes in the normal flow belong to a formatting context, which may be block or inline, but not both simultaneously. Block level boxes participate in a ***block fomatting context***. Inline-level boxes  participate in an ***inline formatting context***.



#### Block Formatting Context<span id="BFC"></span>

Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.

In a block formatting context, boxes are laid out one after the other, vertically, beginning at the top of a containing block. The vertical distance between two sibling boxes is determined by the 'margin' properties. Vertical margins between adjacent block-level boxes in a block formatting context collapse.

In a block formatting context, each box's left outer edge touches the left edge of the containing block (for right-to-left formatting, right edges touch). This is true even in the presence of floats (although a box's line boxes may shrink due to the floats), unless the box establishes a new block formatting context (in which case the box itself may become narrower due to the floats).



#### Inline Formatting Context<span id="IFC"></span>



## 