In the previous chapter, we mentioned that the rendering process could be looked at as a two steps process:

- projecting 3D shapes on the surface of a canvas and determining which part of these surfaces are visible from a given point of view,
- simulating the way light propagates through space, which combined with a description of the way light interacts with the materials objects are made of, will give these objects their final appearance (their color, their brightness, their texture, etc.).

In this chapter we will only review the first step in more detail, and more precisely explain how each one of these problems (projecting the objects' shape on the surface of the canvas and the visibility problem) is typically solved. While many solutions may be used, we will only look at the most common ones. This is just an overall presentation. Each method will be studied in a separate lesson and an implementation of these algorithms provided (in a self-contained C++ program).

## Going from 3D to 2D: the Projection Matrix

![Figure 1: to create an image of a cube, we just need to extend lines from the objects corners towards the eye and find the intersection of these lines with a flat surface (the canvas) perpendicular to the line of sight.](/images/rendering-3d-scene-overview/perspective4.png?)

An image is just a representation of a 3D scene on a flat surface: the surface of a canvas or the screen. As explained in the previous chapter, to create an image that looks like reality to our brain, we need to simulate the way an image of the world is formed in our eyes. The principle is quite simple. We just need to extend lines from the corners of the object towards the eye and find the intersection of these lines with a flat surface perpendicular to the line of sight. By connecting these points to each other to reform the edges of the object, we get a **wireframe** representation of the scene.

<details>
It is important to note, that this sort of construction is in away a completely arbitrary way of flattening a three-dimensional world onto a two-dimensional surface. The technique we just described gives us what is called in drawing, a one-point perspective projection, and this is generally how we do things in CG because this is how the eyes and also cameras work (cameras were designed to produce images similar to the sort of images our eyes create). But in the art world, nothing stops you from coming up with totally different rules. You can in particular get images with several (two, three, four) points perspective.
</details>

One of the main important visual properties of this sort of projection is that an object gets smaller as it moves further away from the eye (the rear edges of a box are smaller than the front edges). This effect is called **foreshortening**.

![Figure 2: the line of sight passes through the centre of the canvas.](/images/rendering-3d-scene-overview/frustum2.png?)

![Figure 3: the size of the canvas can be changed. Making it smaller reduces the field of view.](/images/rendering-3d-scene-overview/frustum1.png?)

There are two two important things to note about this type of projection. First, the eye is in the center of the canvas. In other words, the line of sight always passes through the middle of the image (figure 2). Note also that the size of the canvas itself is something we can change. We can more easily understand what the impact of changing the size of the canvas has if we draw the viewing frustum (figure 3). The **frustum** is the pyramid defined by tracing lines from each corner of the canvas towards the eye, and extending these lines further down into the scene (as far as the eye can see). It is also referred to as the viewing frustum or viewing volume. You can easily see that the only objects visible to the camera are those which are contained within the volume of that pyramid. By changing the size of the canvas we can either extend that volume or make it smaller. The larger the volume the more of the scene we see. If you are familiar with the concept of focal length in photography, then you will have recognized that this has the same effect as changing the focal length of photographic lenses. Another way of saying this is that by changing the size of the canvas, we change the field of view.

![Figure 4: when the canvas becomes infinitesimally small, the lines of the frustum become orthogonal to the canvas. When then get what we call an orthographic projection. The game SimCity uses a form of orthographic view which gives it a unique look.](/images/rendering-3d-scene-overview/ortho.png?)

Something interesting happens when the canvas becomes infinitesimally small: the lines forming the frustum, end up parallel to each other (they are orthogonal to the canvas). This is of course impossible in reality, but not impossible in the virtual world of a computer. In this particular case, you get what we call an **orthographic projection**. It's important to note that orthographic projection is a form of perspective projection, only one in which the size of the canvas is virtually zero. This has for effect to cancel out the **foreshortening effect**: the size of the edges of objects are preserved when projected to the screen.

![Figure 5: P' is the projection of P on the canvas. The coordinates of P' can easily be computed using the property of similar triangles.](/images/rendering-3d-scene-overview/projection.png?)

Geometrically, computing the intersection point of these lines with the screen is incredibly simple. If you look at the adjacent figure (where P is the point projected onto the canvas, and P' is this projected point), you can see that the angle \\(\\angle ABC\\) and \\(\\angle AB'C'\\) is the same. A is defined as the eye, AB the distance of the point P along the z-axis (P's z-coordinate), and BC the distance of the point P along the y-axis (P's y coordinate). B'C' is the y coordinate of P', and AB' is the z-coordinate of P' (and also the distance of the eye to the canvas). When two triangles have the same angle, we say that they are **similar**. Similar triangles have an interesting property: the ratio of the lengths of their corresponding sides is constant. Based on this property, we can write that: 

$${ BC \over AB } = { B'C' \over AB' }$$

If we assume that the canvas is located 1 unit away from the eye (in other words that AB' equals 1 (this is purely a convention to simplify this demonstration), and if we substitute AB, BC, AB' and B'C' with their respective points' coordinates, we get: 

$${ BC \over AB } = { B'C' \over 1 } \rightarrow P'.y = { P.y \over P.z }.$$

In other words, to find the y coordinate of the projected point, you simply need to divide the point y-coordinate by its z-coordinate. The same principle can be used to compute the x coordinate of P':

$$P'.x = { P.x \over P.z }.$$

This is a very simple and yet this is an extremely important relationship in computer graphics, known as the **perspective divide** or z-divide (if you were on a desert island and needed to remember something about computer graphics, that would probably be this equation).

In computer graphics, we generally perform this operation using what we call a **perspective projection matrix**. As its name indicates, it's a matrix that when applied to points, projects them to the screen. In the next lesson, we will explain step by step how and why this matrix works, and learn how to build and use it.

But wait! The problem is that whether you need the perspective projection depends on the technique you use to sort out the visibility problem. Anticipating what we will learn in the second part of this chapter, algorithms for solving the visibility problems come into two main categories:

- Rasterisation,
- Ray-tracing.

Algorithms of the first category relies on projecting P onto the screen to compute P'. For these algorithms, the perspective projection matrix is therefore needed. In ray tracing, rather than projecting the geometry onto the screen, we trace a ray passing through P' and look for P. We don't need to project P anymore with this approach since we already know P', which means that in ray tracing, the perspective projection is technically not needed (and therefore never used).

<details>
We will study the two algorithms in detail in the next chapters and the next lessons. However, it is important to understand the difference between the two and how they work at this point. As explained before, the geometry needs to be projected onto the surface of the canvas. To do so, P is projected along an "implicit" line (implicit because we never really need to build this line as we need to with ray tracing) connecting P to the eye. You can see the process as if you were moving a point along that line from P to the eye until it lies on the canvas. That point would be P'. In this approach, you know P, but you don't know P'. You compute it using the projection approach. But you can also look at the problem the other way around. You can wonder whether, for any point on the canvas (say P' - which by default we will assume is in the center of the pixel), there is a point P on the surface of the geometry that projects onto P'. The solution to this problem is to explicitly this time create a ray from the eye to P', extend or project this ray down into the scene, and find out if this ray intersects any 3D geometry. If it does, then the intersection point is P. Hopefully, you can now see more distinctively the difference between rasterization (we know P, we compute P') and ray tracing (we know P', we look for P).
</details>

The advantage of the rasterization approach over ray tracing is mainly sped. Computing the intersection of rays with geometry is a computationally expensive operation. This intersection time also grows linearly with the amount of geometry contained in the scene, as we will see in one of the next lessons. On the other hand, the projection process is incredibly simple, relies on basic math operations (multiplications, divisions, etc.), and can be aggressively optimized (especially if special hardware is designed for this purpose which is the case with GPUs). Graphics cards are almost all using an algorithm based on the rasterization approach (which is one of the reasons they can render 3D scenes so quickly, at interactive frame rates). When real-time rendering APIs such as OpenGL or DirectX is used, the projection matrix needs to be dealt with. Even if you are only interested in ray tracing, you should know about it for at least a historical reason: it is one of the most important techniques in rendering and the most commonly used technique for producing real-time 3D computer graphics. Plus, it is likely at some point that you will have to deal with the GPU anyway, and real-time rendering APIs do not compute this matrix for you. You will have to do it yourself.

<details>
The concept of rasterisation is really important in rendering. As we learned in this chapter, the projection of P onto the screen, can be computed by dividing the point's coordinates x and y by the point's z-coordinate. As you may guess, all initial coordinates are real numbers - floats for instance - thus P' coordinates are also real numbers. However pixel coordinates need to be integers, thereby, to store the color of P's in the image, we will need to convert its coordinates to pixel coordinates - in other words from floats to integers. We say that the point's coordinates are converted from screen space to raster space. More information can be found on this process in the lesson on rays and cameras.
</details>

The next three lessons are devoted to studying the construction of the orthographic and perspective matrix, and how to use them in OpenGL to display images and 3D geometry.