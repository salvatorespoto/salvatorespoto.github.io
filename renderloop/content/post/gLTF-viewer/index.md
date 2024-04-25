---
title: "A gLTF Viewer in DirectX 12" # Title of the blog post.
date: 2023-01-25T16:08:01+02:00 # Date of post creation.
description: "A gLTF viewer in C++ and DX12." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "featured.png" # Sets featured image on blog post. 
featureImageAlt: 'gLTF Viewer' # Alternative text for featured image.
featureImageCap: 'gLTF Viewer' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - 3D Rendering
tags:
  - Directx 12
  - C++
# comment: false # Disable comment if false.
---

### A viewer of the glTF file format, implemented in DirectX 12 and C++.

Here's a viewer for the glTF file format, built using DirectX 12 and C++. I wrote this software to explore the **DirectX 12 API**. 
**glTF** is a standard file format for three-dimensional scenes and models.

A the following link there are the [glTF specification](https://github.com/KhronosGroup/glTF/tree/master/specification/2.0) from the **Khronos Group**. 

The glTFViewer can **edit** the **vertex**, **geometry** and **fragment shaders on the fly**.

The video below shows the application in action:

{{< youtube tEVuwpKdP4A >}}

The glTF format includes many features, but this viewer only supports some of them.

#### Current supported features:

* **Scenes**: 
	* scene 
	* nodes hierarchy
	* nodes transformation
	
* **Meshes**: 
	* geometry
	* all the attributes are supported: position, normals, tangents, textcoords, etc.), 
	* instantiation
	
* **Materials**: 
	* textures
	* images 
	* samples
	* additional maps: normal, occlusion, emission
	
* **glTF Shading model**

#### Unsupported features:
* Ray Tracing
* Sparse accessors 
* Animations
* Skinning
* Morphing
* Cameras
* Interpolation

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
