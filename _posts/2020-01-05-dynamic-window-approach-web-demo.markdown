---
layout: post
permalink: /dwa/
date:   2020-01-05 19:31:54 +0300
categories: demo
tags: [C, OpenGL, WebGL]
---

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
<script type="text/javascript" src="/assets/js/dat.gui.min.js"></script>
<script type="text/javascript">
    update_config = Module.cwrap('update_config', ['number', 'number',
            'number', 'number', 'number']);

    var obj = {
            dt: 0.1,
            PredictTime: 1.0,
            Heading: 0.15,
            Clearance: 1.0,
            Velocity: 1.0,
        };

    var gui = new dat.gui.GUI();

    gui.remember(obj);

    gui.add(obj, 'PredictTime').min(1.0).max(10.0).step(0.1).onChange(function () {
            update_config(obj.dt, obj.PredictTime, obj.Heading,
                    obj.Clearance, obj.Velocity);
        });;
    gui.add(obj, 'dt').min(0.1).max(1.0).step(0.01).onChange(function () {
            update_config(obj.dt, obj.PredictTime, obj.Heading,
                    obj.Clearance, obj.Velocity);
        });
    gui.add(obj, 'Heading').min(0.1).max(1.0).step(0.01).onChange(function () {
            update_config(obj.dt, obj.PredictTime, obj.Heading,
                    obj.Clearance, obj.Velocity);
        });;
    gui.add(obj, 'Clearance').min(0.1).max(1.0).step(0.01).onChange(function () {
            update_config(obj.dt, obj.PredictTime, obj.Heading,
                    obj.Clearance, obj.Velocity);
        });;
    gui.add(obj, 'Velocity').min(0.1).max(1.0).step(0.01).onChange(function () {
            update_config(obj.dt, obj.PredictTime, obj.Heading,
                    obj.Clearance, obj.Velocity);
        });;
</script>

<!-- -->

- Web Demo Source Code: [DWAGL](https://github.com/goktug97/DWAGL)
- Dynamic Window Approach Source Code: [Dynamic Window Approach](https://github.com/goktug97/DynamicWindowApproach)


