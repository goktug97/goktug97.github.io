---
layout: page
permalink: /dwa/
---

# Dynamic Window Approach Web Demo

Draw obstacles and it will follow your mouse while avoiding obstacles using
Dynamic Window Approach. Press R to reset.

<!-- -->
<canvas id=canvas></canvas>
<script type='text/javascript'>
var Module = {
canvas: (function() {
         var canvas = document.getElementById('canvas');
         canvas.addEventListener("webglcontextlost", function(e) {
                 alert('WebGL context lost. You will need to reload the page.');
                 e.preventDefault(); },
                 false);
         return canvas;
         })(),
};
</script>
<script src="/assets/js/dwa.js"></script>
<!-- -->

- Web Demo Source Code: [DWAGL](https://github.com/goktug97/DWAGL)
- Dynamic Window Approach Source Code: [Dynamic Window Approach](https://github.com/goktug97/DynamicWindowApproach)


