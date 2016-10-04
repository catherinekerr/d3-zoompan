Using viewBox, behavior.zoom() AND .on("click")
===============================================

* Use the mouse wheel to zoom in/out
* Use mouse drag to pan canvas
* Click on circle to enlarge and center in box

With this example, I wanted to solve a three-fold problem:  
1. Scale SVG to fit into smaller window size.  
2. Use mouse drag and thumbwheel to pan and zoom the SVG.  
3. Use mouse-click to zoom into specific path and center it.  
  
The first and second problems are easily solved using the viewBox attribute and d3.behavior.zoom(). The difficulty is calculating the transform.translate co-ordinates when solving problem 3. When you use the viewBox attribute to transform the SVG, the scale is automatically reset to 1. Therefore, subsequent transformations need to take that into consideration.  

In this example, there are three circles as described in file circles.tsv. The SVG viewbox (outlined in red) has width 1000 and height 500, so we need to fit the 3 circles into this container. The bounding box that surrounds the 3 circles has top-left co-ordinates (200,100), width 2000 and height 2000.  
So we are trying to fit an SVG of width 2000 and height 2000 into a viewbox of width 1000 and height 500. The SVG is twice the width and 4 times the height of the viewbox. Therefore, if we want to scale uniformly, we must scale the whole SVG to one quarter (0.25). However, with the viewBox attribute, we don't need to worry about this. Simply set the viewBox attribute to the top-left co-ordinates, width and height of the circles' bounding box as follows:  

<pre style="background: #E3E3E3;">
<code>
  bbox = container.node().getBBox();  
  vx = bbox.x;  
  vy = bbox.y;  
  vw = bbox.width;  
  vh = bbox.height;  

  defaultView = "" + vx + " " + vy + " " + vw + " " + vh;  

  svg  
	.attr("viewBox", defaultView)  
	.attr("preserveAspectRatio", "xMidYMid meet")  
        .call(zoom);  
</code>
</pre>

`container` is the SVG group containing the 3 circles  
`vx` and `vy` are the top-left co-ordinates of the `container` (200,100)  
`vw` is the width from the left of the blue circle to the right of the red circle  
`vh` is the height from the top of the blue circle to the bottom of the yellow circle  
`preserveAspectRatio` scales the SVG uniformly  
`xMidYMid meet` centers the SVG horizontally and vertically in the viewBox  
`.call(zoom)` transforms the co-ordinates and scale upon upon using the mousewheel or upon dragging with the mouse  

So far so good, but I also wanted to zoom into a circle when the circle is clicked. To do this, we need to increase the scale and center the bounding box of the circle in the viewbox. Remember that the scale has been reset to 1 using the viewBox attribute, so if we want to double the circle's width and height, we use `scale = 2`. Now the tricky bit is working out the translate values for the x and y co-ordinates. We do this using the following function:  

<pre style="background: #E3E3E3;">
<code>
  function getTransform(node, scale) {
    bbox = node.node().getBBox();
    var bx = bbox.x;
    var by = bbox.y;
    var bw = bbox.width;
    var bh = bbox.height;
    var tx = -bx*scale + vx + vw/2 - bw*scale/2;
    var ty = -by*scale + vy + vh/2 - bh*scale/2;
    return {translate: [tx, ty], scale: scale}
  }
</code>
</pre>

`bx` and `by` are the top-left co-ordinates of the `node` (in this case, the circle that was clicked)  
`bw` (and `bh`) is the diameter of the circle  
`vx, vy, vw, vh` are as above
`tx` is the translation of the x co-ordinate  
`ty` is the translation of the y co-ordinate  

Note the formulas for `tx` and `ty`. You can try different scenarios by changing the `width`, `height` and `scale` values or by changing the circle positions and sizes in circles.tsv.

