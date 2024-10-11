---
title: "Signal Processing in Computer Graphics #1: Overview" # Title of the blog post.
date: 2024-09-10T20:28:24+02:00 # Date of post creation.
description: "Signal Processing in Computer Graphic: Overview." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
series: "Signal Processing in Computer Graphics"
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "featured.png" # Sets featured image on blog post.
featureImageAlt: '' # Alternative text for featured image.
featureImageCap: '' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "thumbnail.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
showRelatedInArticle: false

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

Rendering an image is basically a sampling process: a continuous image, with its geometry, colors, and lighting, has to be sampled and turned into pixels on the screen. 
These brief notes are a quick intro to signal processing, focusing on aliasing and some algorithms to reduce it.

* [Introduction](/post/aliasing-antialiasing/2-introduction)
* [Signals](/post/aliasing-antialiasing/3-signals)
* [Sampling](/post/aliasing-antialiasing/4-sampling-filtering)
* [Texture Filtering](/post/aliasing-antialiasing/5-texture-filtering)
* [Aliasing](/post/aliasing-antialiasing/6-aliasing)
* [SSAA](/post/aliasing-antialiasing/7-SSAA)
* [MSAA](/post/aliasing-antialiasing/8-MSAA)
* [TAA](/post/aliasing-antialiasing/9-TAA)

*Hope you'll find these arguments interesting !!!*