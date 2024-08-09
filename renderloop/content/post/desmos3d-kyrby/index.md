---
title: "3D Modeling with Math for Fun" # Title of the blog post.
date: 2023-01-25T16:08:01+02:00 # Date of post creation.
description: "Kirby modeled with math functions in Desmos." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "featured.png" # Sets featured image on blog post. 
featureImageAlt: 'Desmos 3d Kirby Drawing' # Alternative text for featured image.
featureImageCap: 'Desmos 3d Kirby Drawing' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Math
tags:
  - Desmos
  - Math
# comment: false # Disable comment if false.
---

### 3D Modelins with Math for Fun.
Desmos recently added 3D graphing functionalities, so I had fun modeling Kirby with some stars around him. 
This was obviously inspired by the incredible work of [Inigo Quilez](https://iquilezles.org), who is renowned for his exceptional math modeling, 
especially on Shadertoy. You can check out an example of his amazing work [here](https://www.youtube.com/watch?v=8--5LwHRhjk).

For this scene, I mainly used distorted ellipsoids, both implicit and explicit. 
I found that implicit shapes are easier to blend smoothly. I then added some stars using a math function specifically for that, which only requires a few parameter settings.

Here’s the result (it takes some seconds to render ;): 

---
<iframe src="https://www.desmos.com/3d/fnmfv5vo4a?lang=es" width="100%" height="600" style="border: 1px solid #ccc" frameborder=0></iframe>

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
