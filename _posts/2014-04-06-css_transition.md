---
layout: post
title:  "Different Transitions for different state!"
categories: css
---

we all know, css transitions are the part of the CSS3, that provide a way to control animation when css properties changed. 

for example, now I have a box, I want to add the animation when the mouse hover on the button. The code can like below:

```html
<!DOCTYPE html>
<html>
<head>
	<style> 
		div {
		width:100px;
		height:100px;
		background:red;
		-webkit-transition:width 2s; /* For Safari 3.1 to 6.0 */
		transition:width 2s;
		}

		div:hover {
		width:300px;
		height: 400px;
		-webkit-transition:width 2s; /* For Safari 3.1 to 6.0 */
		transition:height 2s;
		}
	</style>
</head>
<body>
	<p><b>Note:</b> This example does not work in Internet Explorer 9 and earlier versions.</p>
	<div></div>
	<p>Hover over the div element above, to see the transition effect.</p>
</body>
</html>
```

When I hover on the box, the height of box will changed slowly. But the two state both have transition property. How they merge to generate animation?

If you hover off the box, you will find the width changed slowly and the height shrink instantly, is that tricky!

The answer is that the two state's transition property will `override` each other. When the mouse hover on the box, the hover transition override the transiton set in the regular state, so the height transition will take action. But when the mouse hover off the box, the regular transition take over the regular animation will happen. 
