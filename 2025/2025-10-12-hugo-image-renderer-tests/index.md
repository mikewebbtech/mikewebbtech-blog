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

# Test Image Link

![Home Page](homepage.png)

sLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.omething
{class="p-border"}

# Image resized 90%

![svg logo](hugo-logo-wide.svg)
{width="90%" class="center"}

mLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.ore

# Image resized and centered {class="center"}

![Small Square Image](network.jpg "test")
{width="300" class="center"}
aLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.gain here as well

# Image Left {class="left"}

![User Avatar](avatar.png "left")
{class="left"}

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit

# A very long Title for Image Right Example {class="right"}

![User Avatar](avatar.png "right")
{width="150" class="right" border="4px"}
TLorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.his is the end here

**Using Shortcode here**

{{< figure src="hugo-logo-wide.svg" alt="hugo logo" title="Hugo Logo" width="200" align="right" wrap="true" >}}

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.

{{< figure src="hugo-logo-wide.svg" alt="hugo logo" title="Hugo Logo" width="200" align="left" wrap="true" >}}

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum. Integer facilisis mauris non nunc posuere, at pulvinar arcu ultricies. Vivamus maximus, metus non gravida aliquet, nunc libero viverra nisi, a commodo urna nibh at risus. Curabitur faucibus, enim a iaculis condimentum, tellus quam luctus risus, sed feugiat risus arcu vitae magna. Suspendisse potenti. Aenean ac sem at ipsum iaculis pretium. Praesent sit amet dapibus velit, vitae fermentum elit. Nullam condimentum, eros sit amet gravida sodales, nunc ante lacinia nibh, in convallis metus augue a augue. Cras et mi sapien. Phasellus placerat dictum augue nec vehicula. In hac habitasse platea dictumst. Morbi a sapien sed tellus porta rhoncus. Sed porttitor, nibh id tempor tristique, justo sem ultrices odio, non elementum lectus turpis vitae elit.
{class="p-border"}
