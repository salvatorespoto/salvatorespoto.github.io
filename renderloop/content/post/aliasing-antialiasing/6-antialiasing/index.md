---
title: "Signal Processing in Computer Graphics #6: AntiAliasing" # Title of the blog post.
date: 2024-09-16T20:28:24+02:00 # Date of post creation.
description: "Signal Processing in Computer Graphics: SSAA SuperSampling." # Description used for search engine.
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
  - Alias
  - Aliasing
  - AntiAliasing
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

Anti-aliasing is a technique used in computer graphics to reduce the alias we have described in the previous chapters.  

Here there is an example in ShaderToy of three popular anti-aliasing algorithms: SSAA (Super-Sample Anti-Aliasing), MSAA (Multi-Sample Anti-Aliasing), and TAA (Temporal Anti-Aliasing).  
Please check the shader code for further details.

## SSAA (Super-Sample Anti-Aliasing)

Super-Sample Anti-Aliasing (SSAA) is one of the simplest forms of anti-aliasing. It works by rendering the scene at a higher resolution and then downscaling it to the desired display resolution. This method provides high-quality results but is computationally expensive.  
The ShaderToy example alternates between a scene with SSAA enabled and one without it every few seconds.

An example in Shadertoy of SSAA|
--------|
	<iframe width="100%" height="460" frameborder="0" src="https://www.shadertoy.com/embed/ssVfWz?gui=true&amp;t=10&amp;paused=false&amp;muted=false" allowfullscreen=""></iframe>


## MSAA (Multi-Sample Anti-Aliasing)

Multi-Sample Anti-Aliasing (MSAA) improves performance compared to SSAA by only sampling multiple points within polygons where edges are detected, instead of the entire scene. This makes MSAA a more efficient solution, offering a good balance between performance and quality.  
The ShaderToy example shows the same scene without MSAA on the left, and with it on the right.

An example in Shadertoy of MSAA|
--------|
	<iframe width="100%" height="460" frameborder="0" src="https://www.shadertoy.com/embed/7t3yR2?gui=true&amp;t=10&amp;paused=false&amp;muted=false" allowfullscreen=""></iframe>


## TAA (Temporal Anti-Aliasing)

Temporal Anti-Aliasing (TAA) uses information from previous frames to smooth out jagged edges. It is highly effective at reducing aliasing in motion, but it may cause ghosting artifacts in some cases. TAA is widely used in modern games due to its ability to provide high-quality results with relatively low computational cost.  

The ShaderToy example alternates between a scene with TAA enabled and one without it every few seconds.

An example in Shadertoy of TAA|
--------|
	<iframe width="100%" height="460" frameborder="0" src="https://www.shadertoy.com/embed/st3yRN?gui=true&amp;t=10&amp;paused=false&amp;muted=false" allowfullscreen=""></iframe>
	
<br />

*And these notes end here. It was just a brief journey into the realm of aliasing. Hope you enjoyed it, see you!*