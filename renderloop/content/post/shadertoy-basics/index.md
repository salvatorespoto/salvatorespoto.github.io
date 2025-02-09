---
title: "Shadertoy: Basic Things You Can Do" # Title of the blog post.
date: 2025-02-08T20:28:24+02:00 # Date of post creation.
description: "Some of the basic things you can do in Shadertoy." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "" # Sets featured image on blog post.
featureImageAlt: '' # Alternative text for featured image.
featureImageCap: 'This is the featured image.' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "thumbnail.png" # Designate a separate image for social media sharing.
codeMaxLines: 60 # Override global value for how many lines within a code block before auto-collapsing.
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

**This is a collection of simple shaders in Shadertoy that show how to draw basic elements using basic functions like step and smoothstep. If you're new to Shadertoy or shader programming in general, you might find these useful.**

In the following paragraphs, you'll find shaders that illustrate some simple functions and basic drawing techniques. The code includes plenty of comments to guide you through the implementation.

The shader code is pasted below each implementation, but if you click on the shader's name, you'll be taken to Shadertoy, where you can experiment with it.

{{% notice info "Info" %}}
On this page you'll find shaders written in [Shadertoy](https://shadertoy.com/ "ShaderToy"). 
[Read about use the interactive content in this blog](/post/howto-interactive-content)
{{% /notice %}}

When reading the following shaders, keep in mind that each shader is executed for every pixel independently, with only the pixel’s coordinates as input.  

If the drawing logic seems cumbersome, the key point is that a shader can only control the color of the pixel it is executed on—it **cannot** modify other pixels on the screen. This means the logic is somewhat reversed: instead of actively placing shapes, the shader must determine whether the desired drawing overlaps its own coordinates and, if so, output the appropriate color.

## The Step Function
Probably the simplest and most commonly used function in Shadertoy shaders. The **[step(edge, x)](https://registry.khronos.org/OpenGL-Refpages/gl4/html/step.xhtml)** function is returns *0* if *x* is less than *edge* and *1.0* otherwise. It creates a sharp transition.
```glsl
// The step function: zero in the left half of the screen, 1.0 on the right
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Step return 0 if fragCoord.x is less then the half screen resolution, 1.0 otherwise
    float c = step(iResolution.x/2.0, fragCoord.x);
    fragColor = vec4(c, c, c, 0.0);
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/ttGSRD?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

## The Smoothstep Function  
The "smooth" version of the previous one. With this function **([smoothstep](https://registry.khronos.org/OpenGL-Refpages/gl4/html/smoothstep.xhtml))** the transition is smooth. It's probably the second most basic function used in Shadertoy shaders.
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Performs the Hermite interpolation.
    // See https://registry.khronos.org/OpenGL-Refpages/gl4/html/smoothstep.xhtml

    // Smooth transition from left to right
    float c = smoothstep(0., iResolution.x, fragCoord.x);
    
    fragColor = vec4(c,c,c,1.);
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/WtVXRW?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

## Centered Normalized Coordinates  
When working in Shadertoy, it's often useful to convert the input pixel coordinates into a more manageable form, like normalized coordinates. In this example, the pixel coordinates are converted to the range **[-Aspect Ratio, Aspect Ratio]×[-1,1]** (width × height).
Aspect Ration is screen_width / screen_height.

This coordinate system is more convenient because:  
- The origin \((0,0)\) is at the center of the image.  
- The y-axis ranges from \(-1\) to \(1\).  
- The x-axis is scaled to preserve the aspect ratio, so its values fall within [-aspectRatio, aspectRatio].
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Normalizing coordinates: the center of the screen is (0,0)
    // and the coordinates range from -1 to 1 along the y axis
    vec2 U = ( 2. * fragCoord - iResolution.xy ) / iResolution.y;
    
    // Compute the length of the vectors in normalized coordiantes (that it's the distance from the center)
    float f = length(U);
    
    // Draw the length as a shade of grey, black is near, while white is far
    fragColor = vec4(f,f,f,1.);
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/3lKXDR?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

## Draw a Point  
The most basic of all drawings: a point— and not even an antialiased one! For the antialiased version, use **smoothstep** instead of **step**.
```glsl
// Draw a circular point P with a radius of r
#define drawPoint(P, r) step( length(U - P), r)

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Normalizing coordinates: the center of the screen is (0,0)
    //  and the coordinates range from -1 to 1 along the y axis
    vec2 U = ( 2. * fragCoord - iResolution.xy ) / iResolution.y;
    
    // Coordinates of the points
    vec2 P1 = vec2(-1.,0.);
    vec2 P2 = vec2(-.5,0.);
    vec2 P3 = vec2(0.,0.);
    vec2 P4 = vec2(.5,0.);
    vec2 P5 = vec2(1.,0.);
    
    // The size of 1 pixel in normalized coordinates
    float px_size = 2. / iResolution.y;
    
    // Draw the points: if this pixel belongs to one of the points, then one of the drawPoint 
    // will return 1., and we save this info. For this reason we getting the maximum value of each call.
    // If the pixel doesn't belong to any point, all the functions will return 0.0
    float f = drawPoint( P1, px_size);
    f = max(f, drawPoint( P2, px_size * 2.));
    f = max(f,drawPoint( P3, px_size * 4.));
    f = max(f,drawPoint( P4, px_size * 8.));
    f = max(f,drawPoint( P5, px_size * 16.));
    
    fragColor = vec4(f,f,f,1.);
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/WlVXWz?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

## Draw a Segment  
How do you draw a segment? Here it is! The key is to **compute the distance from the pixel to the segment line**. If this distance is 0, the pixel might belong to the segment, but only if it lies between the segment's endpoints, A and B. So we need to check that as well.

```glsl
// Returns 1.0 if the point is closed to the segment less than the thickness
#define drawLine(A, B, r) smoothstep(r,0.,distanceFromSegment(U, A, B))

// Draw a segment between A and B, with thickness r.
// It computes the distance of a point from the segment line and return it. If this distance is 0 
// the point belong to the line (and so to the segment)
float distanceFromSegment(vec2 U, vec2 A, vec2 B) 
{
	vec2 UA = U - A;
    vec2 BA = (B - A);
    
    float s = dot(UA, BA) / length(BA);   // scalar projection of U-A on B-A
    s = s / length(BA); 				  // normalize the projection value in the range [0,1], 
    								      //  a value of 0 means the projection correspond to A, 1 to B,
    									  //  in between the projection is inside the segment, 
                                          //  outside [0,1] the projection doesn't belong to the segment.
    s = clamp(s, 0., 1.);                 // If the scalar projection is outside [0,1], its value is clamped to 
                                          //  0 or 1 ...
    return length(UA - s*BA);          	  // ... so here we compute the distance of U from its projection if it is
                                          // inside the segment, or from the extreme points A or B if it is outside
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Normalizing coordinates: the center of the screen is (0,0)
    // and the coordinates range from -1 to 1 along the y axis
	  vec2 U = ( 2. * fragCoord - iResolution.xy ) / iResolution.y;
     
    // The size of 1 pixel in normalized coordinates
    float px_size = 2. / iResolution.y;
    
    // Thickness
    float thickness = px_size;
    
    // The segments points
    vec2 A = vec2(-1.0,-0.5);
    vec2 B = vec2(-0.5,0.5);
    vec2 C = vec2(0.0,-0.5);
    vec2 D = vec2(0.5,0.5);
    vec2 E = vec2(1.0,-0.5);
    
    // If the point belong at one line the value will be 1., otherwise 0.
    float f = drawLine(A, B, thickness)
        	 +drawLine(B, C, thickness)
        	 +drawLine(C, D, thickness)
        	 +drawLine(D, E, thickness);
    
    fragColor = vec4(f,f,f,1.);
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/wlKXWz?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

## Draw a Function  
Now let's use everything we've learned to draw some functions. Note also how the centered normalized coordinates are **scaled up** (zoom out) here.
```glsl
// Draw a function
// The functions are scaled or translated so they fit better in the viewport

#define f_sin(x) sin(6. * x)      // The sin function scaled along the x-axis to make it more visible
#define f_x(x) x                // A line
#define f_log(x) log(x)           // Logarithm
#define f_exp(x) exp(x - 1.) - 1. // Exponential function translated on the x and y axes
#define f_ss(x) smoothstep(-1., 1., x)


// Draw a circular point P with a radius of r
#define drawPoint(P, r) smoothstep( r/iResolution.y, 0., abs(length(U - P)))

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    fragColor = vec4(0);
    
    // Normalizing coordinates: the center of the screen is (0,0)
    // and the coordinates range from -1 to 1 along the y axis
	  vec2 U = ( 2. * fragCoord - iResolution.xy ) / iResolution.y;
    
    // This scales up the coordinates (zoom out)
    U *= 4.;
    
    // The position of the point to be drawn
    vec2 P = vec2(U.x, f_ss(U.x));
    float thickness = 10.;
    fragColor = vec4(drawPoint(P, thickness)); // Smoothstep (White)
    
    P = vec2(U.x, f_sin(U.x));
    thickness = 25.;
    // Sum up all the result from drawpoint, if this pixel fall alt least in one of the graphs will be drawn
    fragColor += vec4(0.,0.,drawPoint(P, thickness),1.); // sin(6x) (BLUE)
    
    P = vec2(U.x, f_log(U.x));
    thickness = 25.;
    fragColor += vec4(drawPoint(P, thickness),0.,0.,1.); // log(x) (RED)
    
    P = vec2(U.x, f_exp(U.x));
    thickness = 25.;
    fragColor += vec4(0.,drawPoint(P, thickness),0.,1.); // exp(x-1)-1 (GREEN)
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/wtVXWR?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

## A Checkerboard Pattern
Now let's get a small taste of the power of math in shaders: draw a checkerboard pattern with just **one line of code** (the last one, by the way). Try to figure out why it works!
```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // Change coordinates from [0,iResolution.x]x[0,iResolution.y] to [-1,1]x[-1,-1]
    vec2 UV = (2.*fragCoord-iResolution.xy)/iResolution.x;
    
    // Scale coordinate a bit, to get a bigger checkboard
    UV *= 10.;
    
    // Shift checboard to the right
    UV.x -= iTime;
    
    // Generate checkboard pattern (1 line!!)
    fragColor = vec4(mod(floor(UV.x)+floor(UV.y),2.));
}
```
<iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/fttyDl?gui=true&t=10&paused=false&muted=false" allowfullscreen></iframe>
<br/><br/><br/>

*Shadertoy for Fun and Profit!*

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