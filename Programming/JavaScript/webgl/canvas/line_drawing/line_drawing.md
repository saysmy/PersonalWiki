## Animated Glowing Line Drawing [Back](./../canvas.md)

Before what we talk, we can see a screen shot of it:

![](./canvas_glow.png)

A few random parameters are first set up to determine the color of the line, as well as it's start and end points, and the control points, then a window interval just calls a function which draws a series of lines in decreasing thicknesses in the highlight color, followed by one thin white line on top.

```html
<canvas id="glow" width="1200" height="700">
    Your browser does not support HTML5 Canvas.
</canvas>
```

```js
function draw_line() {
	/** paint over the existing canvas */
	context.fillStyle = "#000";
	context.fillRect(0, 0, canv.width, canv.height);

	// get the new coords based on each ones current trajectory
	get_bounce_coord(x1);
	get_bounce_coord(y1);
	get_bounce_coord(x2);
	get_bounce_coord(y2);

	get_bounce_coord(cx1);
	get_bounce_coord(cy1);
	get_bounce_coord(cx2);
	get_bounce_coord(cy2);

	for(i=5; i>=0; i--)
	{
		context.beginPath();

		// draw each line, the last line in each is always white
		context.lineWidth = (i+1)*4-2;
		if(i==0)
			context.strokeStyle = '#fff';
		else
		{
			context.strokeStyle = 'rgba('+highlight[0]+','+highlight[1]+','+highlight[2]+',0.2)';
		}
		context.moveTo(x1[0], y1[0]);
		context.bezierCurveTo(cx1[0],cy1[0],cx2[0],cy2[0],x2[0],y2[0]);
		context.stroke();
		context.closePath();
	}
}
```