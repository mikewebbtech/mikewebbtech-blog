---
date: 2025-10-12T00:00:00+08:00
draft: false
title: Hugo image-renderer Tests
summary: Testing out some hugo internal image render hooks and to what extent custom styling can be added to images
tags:
  - blog
  - testing
categories:
  - hugo
series:
---

# Testing Hugo Image Render Tricks

<div class="left padded">
  <img src="homepage.png" alt="Home Page" width="420" loading="lazy">
  <img src="hugo-logo-wide.svg" alt="hugo" width="420" loading="lazy">
</div>

<div class="right padded">
  <img src="hugo-logo-wide.svg" alt="hugo" width="420" loading="lazy">
  <img src="homepage.png" alt="Home Page" width="420" loading="lazy">
</div>

sLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.omething
{class="p-border"}

## Image resized 70%

![svg logo](hugo-logo-wide.svg)
{ style="width:70%" .center }

mLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.ore

## Image resized and centered {class="center"}

<br>

![Small Square Image](network.jpg "test")
{width="600" hight="400" .center}
aLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus.

Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.gain here as well

> ## Image Left { .two }

Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula.

![User Avatar](avatar.png "left")
{  .left }

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. <br><br> Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium.<br><br>
Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit

> ## A very long Title for Image Right Example { .right .three }

![User Avatar](avatar.png "right")
{ .right width="150" style="border: 2px"}
TLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit.

Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.his is the end here

### Using Shortcode here

{{< figure src="hugo-logo-wide.svg" alt="hugo logo" width="150" align="right" wrap="true" caption="[Hugo the fast SSG](https://gohugo.io)">}}

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna.

{{< figure src="vimage.png" alt="hugo logo" imgStyle="height:275px; width:auto;" align="left" wrap="true" >}}

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus.<br><br>
Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.

<!-- {class="p-border"} -->
