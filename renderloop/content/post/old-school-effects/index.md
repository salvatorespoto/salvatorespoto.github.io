---
title: "Nostalgia: old schoold demoscene effects in Shadertoy" # Title of the blog post.
date: 2025-02-01T20:28:24+02:00 # Date of post creation.
description: "Old schoold demoscene effects in Shadertoy." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "" # Sets featured image on blog post.
featureImageAlt: '' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Computer Graphics
tags:
  - DemoScene
  - Assembly
  - ShaderToy
  - Second Reality
  - Future Crew
# comment: false # Disable comment if false.
---

Nostalgia time! The early '90s were a golden era for the demoscene. In 1993, *Second Reality* hit like a bombshell for all of us messing around with computers and coding. No 3D acceleration back then, just CPU and raw code. *Future Crew* dropped their demo at Assembly, and a wave of inspiration spread across Europe. Every kid into programming, PCs, and video games started dreaming about real-time graphics, trying to replicate the magic of that legendary Finnish group. *Second Reality* was the absolute peak of real-time graphics, and those guys were legit rockstars.

<br/>
<div align="center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/iw17c70uJes" frameborder="0" allowfullscreen></iframe>
  <p><em>Second Reality (1993) by Future Crew.</em></p>
</div>
<br/><br/>

{{% notice info "Info" %}}
On this page you'll find shaders written in [Shadertoy](https://shadertoy.com/ "ShaderToy"). 
[Read about use the interactive content in this site](/post/howto-interactive-content)
{{% /notice %}}

Some of the effects shown in the demo were already classics, popping up everywhere. Feeling nostalgic, I rewrote a few of them in Shadertoy a while back. Of course, nowadays, it’s much more simple  to do in modern GLSL compared to the raw assembly of the early '90s.

Here are some effects that were staples of the demoscene and often seen in cracked game intros. You can spot some of these in Second Reality too.


# Fire
Legend has it this effect was discovered by accident. I kinda believe it.
Fire |
--------|
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/WddcWj?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>

# Plasma
Super simple to implement (nowadays). Back then, it was everywhere. Just a bunch of sine functions summed together. Second Reality implements a very fine version of this effect.
Plasma |
--------|
  <iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/wsdcDj?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
  

# Moire patterns
Check out the Second Reality version.
Moire |
--------|
  <iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/wdcyWX?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>

# Rotozoom
Also in Second Reality ;).
Rotozoom |
--------|
  <iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/wsdyW2?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>


*See you at Assembly 1994!*

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