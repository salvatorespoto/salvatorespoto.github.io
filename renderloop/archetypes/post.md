---
title: "{{ replace .Name "-" " " | title }}" # Title of the blog post.
date: {{ .Date }} # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: true # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
featureImageAlt: 'Description of image' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - Tag_name1
  - Tag_name2
# comment: false # Disable comment if false.
---

**Insert Lead paragraph here.**

{{% notice info "Info" %}}
On this page you'll find shaders written in [Shadertoy](https://shadertoy.com/ "ShaderToy") and functions graph made in [Desmos](https://desmos.com/ "Desmos"). 
[Read about use the interactive content in this site](/post/howto-interactive-content)
{{% /notice %}}

## Section

Reference example <cite>Reference [^1]</cite>. 
[^1]: Reference

Example ShaderToy Shader
2D Metaballs |
--------|
	<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/3s3yWf?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
	
Example Desmos demo
<iframe src="https://www.desmos.com/calculator/ax07mzr2gh" width="100%" height="300" style="border: 1px solid #ccc" frameborder=0></iframe>
	
Example Latex formula
$$f(x,y) = \frac{1}{r}$$ 
Inline latex $r=\sqrt{x^2+y^2}$

Example Table
One scalar field | The metaball |Two fields summed | The metaball
--------|-|-|-
![One scalar field centered in the origin](/images/posts/meta-balls/one-charge.png) | ![The border isoline of the metaball](/images/posts/meta-balls/one-charge-isoline.png) | ![Two fields summed](/images/posts/meta-balls/two-charges.png) | ![The border isoline of the metaball](/images/posts/meta-balls/two-charges-isoline.png) 

Example Citation
>*The metaball is the portion of the scalar field where its value is above a threshold.*

Example Glsl code
```glsl
// This function is called for each pixel
void mainImage( out vec4 fragColor, in vec2 fragCoord)
{

}
```

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