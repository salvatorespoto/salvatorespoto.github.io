---
title: "Signal Processing in Computer Graphics #4: Sampling and Filtering" # Title of the blog post.
date: 2024-09-13T20:28:24+02:00 # Date of post creation.
description: "Signal Processing in Computer Graphics: Sampling." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
series: "Signal Processing in Computer Graphics"
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "featured.png" # Sets featured image on blog post.
featureImageAlt: '' # Alternative text for featured image.
featureImageCap: '' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - AntiAliasing
tags:
  - Sampling
  - Filtering
# comment: false # Disable comment if false.
---

<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
            delimiters: [
                {left: "$$", right: "$$", display: true},
                {left: "$", right: "$", display: false}
            ]
        });
    });
</script>

**Sampling means to take samples of the continous signal at **discrete intervals**. Filtering is the opposite operation: reconstructing the continuous time signal from
its discrete samples.**

{{% notice info "Info" %}}
In this pages you'll find some shaders written with [Shadertoy](https://shadertoy.com/ "ShaderToy"), and some [Desmos](https://desmos.com/ "Desmos") graphs. 
[Read how to use the interactive content in this site](/post/howto-interactive-content).
{{% /notice %}}

<br/>

## Sampling a signal at discrete intervals
**Sampling** means to take samples of the continous signal at **discrete intervals**. This allows to store it as an array of numbers, 
which can be easily loaded into a computer memory or saved on a storage. Sampling often occurs at **uniform intervals**, as shown in the following picture:

As an example to illustrate sampling and filtering consider the following function:

<p style="text-align:center"> $f_{Signal}(x) = sin(2x) + sin(x) + 2 $ </p> 

![The function $ f_{Signal}(x) = sin(2x) + sin(x) + 2$](signal.png).

This function is continuous with **period** $ T= 2\pi $ and **frequency $ F = 1/T = 1/2\pi $**.

As sum of two $sin$ functions the **maximum frequency in the signal is $1/\pi$**, that is the frequency of $sin(2x)$. 

Now sample this signal with a rate of **4 times** this maximum frequency, as illustrated in the following picture. 

![The circles represents the discrete samples of the signal at the sample rate of 4 time its maximum frequency](sampled-signal.png).

> A foundamenmtal result in signal processing is the **Nyquist–Shannon sampling theorem**: the sampling frequency has to be **more than the double of the maximum frequency** in a 
bandwidth-limited signal in order to obtain a **perfect reconstruction**. Anyway this perfect reconstruction occurs **only if the ideal sinc filter is used**.

<br/>

## Reconstructing the continuous-time signal from its discrete samples

**Convolving a filter function** with the discrete samples reconstructs a continuous-time signal. In the following Desmos graph three filters are used: **Box**, **Tent** anc **Sinc**, 
Note that the quality of the recostruction is different according to the filter used.

## The box filter
This filter is shown in the next graph:

<p style="text-align:center"><iframe src="https://www.desmos.com/calculator/qerzonkacq" width="100%" height="500"></iframe></p>

This is the worst filter: a recostruction made with it has high frequency artifacts, the "steps" of the recostructed function. Anyway it's really fast and pratical.

<p style="text-align:center"><iframe src="https://www.desmos.com/calculator/qcimfux1da" width="100%" height="500"></iframe></p>

## The tent filter

The next is the tent filter:

<p style="text-align:center"><iframe src="https://www.desmos.com/calculator/irddopjidk" width="100%" height="500"></iframe></p>

This filter is better then the Box, anyway the reconstructed signal is still not smooth. Applying a tent filter is the same as **linear interpolate** between the sampled points.

<p style="text-align:center"><iframe src="https://www.desmos.com/calculator/utvcbbfawd" width="100%" height="500"></iframe></p>

## The sinc filter

The Sinc filter assures a **perfect recostruction** of a bandwidth-limited signal, if the sample rate is more than the double of the maximum frequency in the signal. 

<p style="text-align:center"><iframe src="https://www.desmos.com/calculator/jpb1p2mqss" width="100%" height="500"></iframe></p>

We can see in the following demo how the recostruction  made at four time the maximum frequency is **nearly perfect**. Anyway using the sinc filter is **impratical** in real
world application, because its domain extends from $-\infty$ to $\infty$. 

<p style="text-align:center"><iframe src="https://www.desmos.com/calculator/exctrhdwkx" width="100%" height="500"></iframe></p>

<br/>
	
*That's it for now. See you next time !!!*