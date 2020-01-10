---
layout: post
title:  "Training a Super-Resolution Network with Random Shapes"
date:   2020-01-10 19:10:23 +0300
categories: research
tags: [super-resolution, deep-learning, python]
---

![Random Shapes](/assets/images/random_shapes.png)

*Not Complete Yet*

To see if a super-resolution can generalize from randomly generated
shapes to real world, we trained EDSR [1] with images that are generated
randomly. One of them can be seen above. Images from 800 to 810 from
DIV2K [2] are used for evalution between epochs during training.

To create images similar to the above, we used gaussian distribution
with different means and standart deviations for different
variables. The settings to create above image are given below. The
same settings are used to create training images.

- **color_mean** = (<span style="color:red">R:</span>
<span style="color:orange">114.35629928</span>,
<span style="color:green">G: </span><span style="color:orange">111.561547</span>,
<span style="color:blue">B: </span><span style="color:orange">103.1545782</span>)
- **color_std** = (<span style="color:red">R:</span>
<span style="color:orange">72.45974394708261</span>,
<span style="color:green">G: </span><span style="color:orange">68.87470687936465</span>,
<span style="color:blue">B: </span><span style="color:orange">74.47221978843599</span>)
- **background_color** = color_mean 
- **width** = <span style="color:orange">2048</span>: Image Width
- **height** = <span style="color:orange">1080</span>: Image Height
- **max_size** = <span style="color:orange">96</span>: Max Shape Size
- **position_x** = <span style="color:green">**Uniform**</span>(low=**0**,
high=**width**)
- **position_y** = <span style="color:green">**Uniform**</span>(low=**0**,
high=**height**)
- Shape is chosen randomly between circle and rectangle;
  * Circle
    - **radius** = <span style="color:green">**Gaussian**</span>(mean=**max_size/4**,
std=**max_size/10**)
  * Rectangle
    - **width** = <span style="color:green">**Gaussian**</span>(mean=**max_size/2**,
std=**max_size/5**)
    - **height** = <span style="color:green">**Gaussian**</span>(mean=**max_size/2**,
std=**max_size/5**)
    - **yaw** = <span style="color:green">**Uniform**</span>(low=**0**,
high=**2π**)
- **color** = <span style="color:green">**Gaussian**</span>(mean=**color_mean**,
std=**color_std**): Calculated for each channel separately.
- **number_of_shapes** = 10000

Color mean and standart deviation are calculated from DIV2K [2], They are
the mean and the standart devation of images between 001-800.

### Training

We trained the network the same as the paper [1] except the patch
size. We used patch size of 24 instead of 192 to train it faster. The
network trained both with DIV2K [2] and the custom dataset with 24
patch size for comparison.

### Benchmarks
Set5[3], Set14[6], BSDS100[5], Urban100[4]

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;border-color:#aaa;}
.tg td{padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#aaa;color:#333;background-color:#fff;}
.tg th{padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#aaa;color:#fff;background-color:#f38630;}
.tg .tg-0lax{text-align:center;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-0lax">PSNR</th>
    <th class="tg-0lax">Set5</th>
    <th class="tg-0lax">Set14</th>
    <th class="tg-0lax">BSDS100</th>
    <th class="tg-0lax">Urban100</th>
  </tr>
  <tr>
    <td class="tg-0lax">DIV2K</td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-0lax">Shapes</td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
</table>

### Conclusion

If everything was colorful rectangles and circles, this technique
might have generalized to real world. 


### Code

[https://github.com/goktug97/randomshapes](https://github.com/goktug97/randomshapes)

### References
1. Bee Lim and Sanghyun Son and Heewon Kim and Seungjun Nah and Kyoung
Mu Lee (2017). Enhanced Deep Residual Networks for Single Image
Super-ResolutionCoRR, abs/1707.02921.
2. Agustsson, E., & Timofte, R. (2017). NTIRE 2017 Challenge on Single
Image Super-Resolution: Dataset and Study. In The IEEE Conference
on Computer Vision and Pattern Recognition (CVPR) Workshops.
3. M. Bevilacqua, A. Roumy, C. Guillemot, and M. L. Alberi- Morel
(2012). Low-complexity single-image super-resolution based on
nonnegative neighbor embedding. In BMVC.
4. J.-B. Huang, A. Singh, and N. Ahuja (2015). Single image super-
resolution from transformed self-exemplars.  In The IEEE Conference on
Computer Vision and Pattern Recognition (CVPR) Workshops.
5. D. Martin, C. Fowlkes, D. Tal, and J. Malik (2001). A database of
human segmented natural images and its application to evaluating
segmentation algorithms and measuring ecological statistics. In
ICCV.
6. R. Zeyde, M. Elad, and M. Protter (2010). On single image scale-up
using sparse-representations. In Proceedings of the International
Conference on Curves and Surfaces.
