An image of a 3D scene can be generated in multiple ways, but of course, any way you choose should produce the same image for any given scene. In most cases, the goal of rendering is to create a photo-realistic image (non-photorealistic rendering or NPR is also possible). But what does it mean, and how can this be achieved? Photorealistic means essentially that we need to create an image so "real" that it looks like a photograph or (if photography didn't exist) that it would look like reality to our eyes (like the reflection of the world off the surface of a mirror). How do we do that? By understanding the laws of physics that make objects appear the way they do, and simulating these laws on the computer. In other words, rendering is nothing else than simulating the laws of physics responsible for making up the world we live in, as it appears to us. Many laws are contributing to making up this world, but fewer contribute to how it looks. For example, gravity, which plays a role in making objects fall (gravity is used in solid-body simulation), has little to do with the way an orange looks like. Thus, in rendering, we will be interested in what makes objects look the way they do, which is essentially the result of the way light propagates through space and interacts with objects (or matter more precisely). This is what we will be simulating.

## Perspective Projection and the Visibility Problem

But first, we must understand and reproduce how objects look to our eyes. Not so much in terms of their appearance but more in terms of their shape and their size with respect to their distance to the eye. The human eye is an optical system that converges light rays (light reflected from an object) to a focus point.

![Figure 1: the human eye is an optical system that converges light rays (light reflected from an object) to a focus point. As a result, by geometric construction, objects which are further away from our eyes, do appear smaller than those which are at close distance.](/images/rendering-3d-scene-overview/perspective1.png?)

As a result, by geometric construction, objects which are further away from our eyes, do appear smaller than those which are at a close distance (assuming all objects have the same size). Or to say it differently, an object appears smaller as we move away from it. Again this is the pure result of the way our eyes are designed. But because we are accustomed to seeing the world that way, it makes sense to produce images that have the same effect: something called the **foreshortening effect**. Cameras and photographic lenses were designed to produce images of that sort. More than simulating the laws of physics, photorealistic rendering, is also about simulating the way our visual system works. We need to produce images of the world on a flat surface, similar to the way images are created in our eyes (which is mostly the result of the way our eyes are designed - we are not too sure about how it works in the brain but this is not important for us).

How do we do that? A basic method consists of tracing lines from the corner of objects to the eye and finding the intersection of these lines with the surface of an imaginary canvas (a flat surface on which the image will be drawn, such as a sheet of paper or the surface of the screen) perpendicular to the line of sight (Figure 2).

![Figure 2: to create an image of the box, we trace lines from the corners of the object to the eye. We then connect the points where these lines intersect an imaginary plane (the canvas) to recreate the edges of the cube. This is an example of perspective projection.](/images/rendering-3d-scene-overview/perspective2.png?)

These intersection points can then be connected, to recreate the edges of the objects. The process by which a 3D point is projected onto the surface of the canvas (by the process we just described) is called **perspective projection**. Figure 3 shows what a box looks like when this technique is used to "trace" an image of that object on a flat surface (the canvas).

![Figure 3: image of a cube created using perspective projection.](/images/rendering-3d-scene-overview/perspective3.png?)

This sort of rendering in computer graphics is called a wireframe because only the edges of the objects are drawn. This image though is not photo-real. If the box was opaque, the front faces of the box (at most three of these faces) should occlude or hide the rear ones, which is not the case in this image (and if more objects were in the scene, they would potentially occlude each other). Thus, one of the problems we need to figure out in rendering is not only how we should be projecting the geometry onto the scene, but also how we should determine which part of the geometry is visible and which part is hidden, something known as the visibility problem (determining which surfaces and parts of surfaces are not visible from a certain viewpoint). This process in computer graphics is known under many names: **hidden surface elimination**, **hidden surface determination** (also known as hidden surface removal, **occlusion culling**, and **visible surface determination**. Why so many names? Because this is one of the first major problems in rendering, and for this particular reason, a lot of research was made in this area in the early ages of computer graphics (and a lot of different names were given to the different algorithms that resulted from this research). Because it requires finding out whether a given surface is hidden or visible, you can look at the problem in two different ways: do I design an algorithm that looks for hidden surfaces (and remove them), or do I design one in which I focus on finding the visible ones. Of course, this should produce the same image at the end but can lead to designing different algorithms (in which one might be better than the others).

The visibility problem can be solved in many different ways, but they generally fall within two main categories. In historical-chronological order:

- Rasterization,
- Ray-tracing.

Rasterization is not a common name, but for those of you who are already familiar with hidden surface elimination algorithms, it includes the z-buffer and painter's algorithms among others. Almost all graphics cards (GPUs) use an algorithm from this category (likely z-buffering). Both methods will be detailed in the next chapter.

## Shading

Even though we haven't explained how the visibility problem can be solved, let's assume for now that we know how to flatten a 3D scene onto a flat surface (using perspective projection) and determine which part of the geometry is visible from a certain viewpoint. This is a big step towards generating a photorealistic image but what else do we need? Objects are not only defined by their shape but also by their appearance (this time not in terms of how big they appear on the scene, but in terms of their look, color, texture, and how bright they are). Furthermore, objects are only visible to the human eye because light is bouncing off their surface. How can we define the appearance of an object? The appearance of an object can be defined as the way the material this object is made of, interacts with light itself. Light is emitted by light sources (such as the sun, a light bulb, the flame of a candle, etc.) and travels in a straight line. When it comes in contact with an object, two things might happen to it. It can either be absorbed by the object or it can be reflected in the environment. When light is reflected off the surface of an object, it keeps traveling (potentially in a different direction than the direction it came from initially) until it either comes in contact with another object (in which case the process repeats, light is either absorbed or reflected) or reach our eyes (when it reaches our eyes, the photoreceptors the surface of the eye is made of convert light into an electrical signal which is sent to the brain).

![Figure 4: an object appears yellow under white light because it absorbs most of the blue light and reflects green and red light which combined to form a yellow color.](/images/rendering-3d-scene-overview/lemon.png?)

- **Absorption** gives objects their unique color. White light (check the lesson on color in the section Introduction to Computer Graphics) is composed of all colors making up the visible spectrum. When white light strikes an object, some of these light colors are absorbed while others are reflected. Mixed, these reflected colors define the color of the object. Under sunlight, if an object appears yellow, you can assume that it absorbs blue light and reflects a combination of red and green light, which combined form the yellow color. A black object absorbs all light colors. A white object reflects them all. The color of an object is unique to the way the material this object is made of absorbs light (it is a unique property of that material).
- **Reflection**. We already know that an object reflects light colors which it doesn't absorb, but in which direction is this light reflected? It happens that the answer to this question is both simple and very complex. At the object level, light behaves no differently than a tennis ball when it bounces back from the surface of a solid object. It simply travels along a direction similar to the direction it came in but flipped around a vector perpendicular to the orientation of the surface at the impact point. In computer graphics, we call this direction a **normal**: the outgoing direction is a **reflection** of the incoming direction with respect to the normal. At the atomic level, when a photon interacts with an atom, the photon can either be absorbed or re-emitted by the atom in any new random direction. The re-emission of a photon by an atom is called **scattering**. We will speak about this term again in a very short while.

In CG, we generally won't try to simulate the way light interacts with atoms, but the way it behaves at the object level. However, things are not that simple. Because if the maths involved in computing the new direction of a tennis ball bouncing off the surface of an object are simple, the problem is that surfaces at the microscopic level (not the atomic level) are generally not flat at all, which causes light to bounce in all sort of (almost random in some cases) directions. From the distance we generally look at common objects (a car, a pillow, a fruit), we don't see the microscopic structure of objects, although it has a considerable impact on the way it reflects light and thus the way they look. However, we are not going to represent objects at the microscopic level, for obvious reasons (the amount of geometry needed would simply not fit within the memory of any conventional or non-conventional for that matter, computer). What do we do then? The solution to this problem is to come up with another mathematical model, for simulating the way light interacts with any given material at the microscopic level. This, in short, is the role played by what we call a **shader** in computer graphics. A shader is an implementation of a mathematical model designed to simulate the way light interacts with matter at the microscopic level.

## Light Transport

Rendering is mostly about simulating the way light travels in space. Light is emitted from light sources, and is reflected off the surface of objects, and some of that light eventually reaches our eyes. This is how and why we see objects around us. As mentioned in the introduction to ray tracing, it is not very efficient to follow the path of light from a light source to the eye. When a photon hits an object, we do not know the direction this photon will have after it has been reflected off the surface of the object. It might travel towards the eyes, but since the eye is itself very small, it is more likely to miss it. While it's not impossible to write a program in which we simulate the transport of light as it occurs in nature (this method is called **forward tracing**), it is, as mentioned before, never done in practice because of its inefficiency.

![Figure 5: in the real world, light travel travels from light sources (the sun, light bulbs, the flame of a candle, etc.) to the eye. This is called forward tracing (left). However, in computer graphics and rendering, it's more efficient to simulate the path of light the other way around, from the eye to the object, to the light source. This is called backward tracing.](/images/rendering-3d-scene-overview/lighttransport.png?)

A much more efficient solution is to follow the path of light, the other way around, from the eye to the light source. Because we follow the natural path of light backward, we call this approach **backward tracing**.

Both terms are sometimes swapped in the CG literature. Almost all renderers follow light from the eye to the emission source. Because in computer graphics, it is the 'default' implementation, some people prefer to call this method, forward tracing. However, in Scratchapixel, we will use forward for when light goes from the source to the eye, and backward when we follow its path the other way around.

The main point here is that rendering is for the most part about simulating the way light propagates through space. This is not a simple problem, not because we don't understand it well, but because if we were to simulate what truly happens in nature, there would be so many photons (or light particles) to follow the path of, that it would take a very long time to get an image. Thus in practice, we follow the path of very few photons instead, just to keep the render time down, but the final image is not as accurate as it would be if the paths of all photons were simulated. Finding a good tradeoff between photo-realism and render time is the crux of rendering. In rendering, a light transport algorithm is an algorithm designed to simulate the way light travels in space to produce an image of a 3D scene that matches "reality" as closely as possible.

When light bounces off a diffuse surface and illuminates other objects around it, we call this effect **indirect diffuse**. Light can also be reflected off the surface of shiny objects, creating caustics (the disco ball effect). Unfortunately, it is very hard to come up with an algorithm capable of simulating all these effects at once (using a single light transport algorithm to simulate them all). It is in practice, often necessary to simulate these effects independently.

Light transport is central to rendering and is a very large field of research.

## Summary

In this chapter, we learned that rendering can essentially be seen as an essential two steps process:

- The perspective projection and visibility problem on one hand,
- And the simulation of light (light transport) as well the simulation of the appearance of objects (shading) on the other.

<details>
Have you ever heard the term **graphics or rendering pipeline**? The term is more often used in the context of real-time rendering APIs (such as OpenGL, DirectX, or Metal). The rendering process as explained in this chapter can be decomposed into at least two steps, visibility, and shading. Both steps though can be decomposed into smaller steps or stages (which is the term more commonly used). Steps or stages are generally executed in sequential order (the input of any given stage generally depends on the output of the preceding stage). This sequence of stages forms what we call the rendering pipeline.
</details>

You must always keep this distinction in mind. When you study a particular technique always try to think whether it relates to one or the other. Most lessons from this section (and the advanced rendering section) fall within one of these categories:

|-table{Projection/Visibility Problem,Light Transport/Shading}
|-row
|-cell
- Perspetive Projection Matrix
- Rays and Cameras
- Rendering a Triangle with Ray Tracing
- Rendering Simple Shapes with Ray Tracing
- Rendering a Mesh Using Ray Tracing
- Transform Objects using Matrices
- Rendering the Utah Teapot
- The Rasterisation Algorithm
- ...
|-cell
- The Rendering Equation
- Light Transport Algorithms: e.g. Path Tracing
- Area Lights
- Texturing
- Motion Blur
- Depth of Field
- ...
|-

We will briefly detail both steps in the next chapters.