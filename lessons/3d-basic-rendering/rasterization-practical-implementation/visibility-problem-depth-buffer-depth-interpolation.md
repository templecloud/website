In the second chapter of this lesson, we learned that in the third coordinate of the projected point (the point in screen space) we store the original vertex z-coordinate (the z-coordinate of the point in camera space):

$$
\begin{array}{l}
P_{screen}.x = \dfrac{ near * P_{camera}.x }{ -P_{camera}.z}\\
P_{screen}.y = \dfrac{ near * P_{camera}.z }{ -P_{camera}.z}\\
P_{screen}.z = -P_{camera}.z\\
\end{array}
$$

Finding the z-coordinate of a point on the surface of the triangle is useful when a pixel overlaps more than one triangle. And the way we find that z-coordinate is by interpolating the original vertices z-coordinates using the barycentric coordinates that we learned about in the previous chapter. In other words, we can treat the z-coordinates of the triangle vertices as any other vertex attribute, and interpolate them the same way we interpolated colors in the previous chapter. Before we look into the details of how this z-coordinate is computed, let's start to explain why we need to do so.

## The Depth-Buffer or Z-Buffer Algorithm and Hidden Surface Removal

![Figure 1: when a pixel overlaps a triangle, this pixel corresponds to a point on the surface of the triangle (noted P in this figure).](/images/rasterization/depth-buffer2.png?)

![Figure 2: when a pixel overlaps several triangles, we can use the points on the triangle's z-coordinate to find which one of these triangles is the closest to the camera.](/images/rasterization/depth-buffer1.png?)

When a pixel overlaps a point, what we see through that pixel is a small area on the surface of a triangle, which for simplification we will reduce to a single point (denoted P in figure 1). Thus each pixel covering a triangle corresponds to a point on the surface of that triangle. Of course, if a pixel covers more than one triangle, we then have several of these points. The problem when this happens is to find which one of these points is visible. We have illustrated this concept in 2D in figure 2. We could test triangles from back to front (this technique would require sorting triangles by decreasing depth first) but this doesn't always work when triangle intersects each other (figure 2, bottom). The only reliable solution is to compute the depth of each triangle a pixel overlaps, and then compare these depth values to find out which one is the closest to the camera. If you look at figure 2, you can see that a pixel in the image overlaps two triangles in P1 and P2. However, the P1 z-coordinate (Z1) is lower than the P2 z-coordinate (Z2) thus we can deduce that P1 is in front of P2. Note that this technique is needed because triangles are tested in a "random" order. As mentioned before we could sort out triangles in decreasing depth order but this is not good enough. Generally, they are just tested in the order they are specified in the program, and for this reason, a triangle T1 that is closer to the camera can be tested before a triangle T2 that is further away. If we were not comparing these triangles' depth, then we would end up in this case seeing the triangle which was tested last (T2) when in fact we should be seeing T1. As mentioned many times before, this is called the **visibility problem** or **hidden surface problem**. Algorithms for ordering objects so that they are drawn correctly are called visible surface algorithms or hidden surface removal algorithms. The depth-buffer or z-buffer algorithm that we are going to study next belongs to this category of algorithms.

One solution to the visibility problem is to use a **depth-buffer** or **z-buffer**. A depth-buffer is nothing more than a two-dimensional array of floats that has the same dimension as the frame-buffer and that is used to store the depth of the object as the triangles are being rasterized. When this array is created, we initialize each pixel in the array with a very large number. If we find that a pixel overlaps the current triangle, we do as follows:

- We first compute the z-coordinate or depth of the point on the triangle that the pixel overlaps.
- We then compare that current triangle depth with the value stored in the depth buffer for that pixel.
- If we find that the value stored in the depth-buffer is greater than the depth of the point on the triangle, then the new point is closer to the observer or the camera than the point stored in the depth buffer at that pixel location. The value stored in the depth-buffer is then replaced with the new depth, and the frame-buffer is updated with the current triangle color. On the other hand, if the value stored in the depth-buffer is smaller than the current depth sample, then the triangle that the pixel overlaps is hidden by the object whose depth is currently stored in the depth-buffer.

Note that once all triangles have been processed, the depth-buffer contains "some sort" of image, that represents the "distance" between the visible parts of the objects in the scene and the camera (this is not a distance but the z-coordinate of each point visible through the camera). The depth buffer is essentially useful to solve the visibility problem, however, it can also be used in post-processing to do things such as 2D depth of field, adding fog, etc. All these effects are better done in 3D but applying them in 2D is often faster but the result is not always as accurate as what you can get in 3D.

Here is an implementation of the depth-buffer algorithm in pseudo-code:

```
float *depthBuffer = new float [imageWidth * imageHeight];
// Initialize depth-buffer with a very large number
for (uint32_t y = 0; y &lt imageHeight; ++y)
    for (uint32_t x = 0; x &lt imageWidth; ++x)
        depthBuffer[y][x] = INFINITY;

for (each triangle in the scene) {
    // Project triangle vertices
    ...
    // Compute 2D triangle bounding-box
    ...
    for (uint32_t y = bbox.min.y; y &lt= bbox.max.y; ++y) {
        for (uint32_t x = bbox.min.x; x &lt= bbox.max.x; ++x) {
           if (pixelOverlapsTriangle(i + 0.5, j + 0.5) {
                // Compute the z-coordinate of the point on the triangle surface
                float z = computeDepth(...);
                // Current point is closest than object stored in depth/frame-buffer
                if (z &lt depthBuffer[y][x]) {
                     // Update depth-buffer with that depth
                     depthBuffer[y][x] = z;
                     frameBuffer[y][x] = triangleColor;
                }
            }
        } 
    } 
}
```

## Finding Z by Interpolation

![Figure 3: can we find the depth of P by interpolating the z coordinates of the triangles vertices z-coordinates using barycentric coordinates?](/images/rasterization/interpolate-depth.png?)

![Figure 4: finding the y-coordinate of a point by linear interpolation.](/images/rasterization/depth-interpolation1.png?)

Hopefully, the principle of the depth-buffer is simple and easy to understand. All we need to do now is explained how depth values are computed. First, let's repeat one more time what that depth value is. When a pixel overlaps a triangle, it overlaps a small surface on the surface of the triangle, which as mentioned in the introduction we will reduce to a point for simplification (point P in figure 1). What we want to find here, is this point z-coordinate. As also mentioned earlier in this chapter, if we know the triangle vertices' z-coordinate (which we do, they are stored in the projected point z-coordinate), all we need to do is interpolate these coordinates using P's barycentric coordinates (figure 4):

$$P.z = \lambda_0 * V0.z + \lambda_1 * V1.z + \lambda_2 * V2.z.$$

Technically this sounds reasonable, though unfortunately, it doesn't work. Let's see why. The problem is not in the formula itself which is perfectly fine. The problem is that once the vertices of a triangle are projected onto the canvas (once we have performed the perspective divide), then z, the value we want to interpolate, doesn't vary linearly anymore across the surface of the 2D triangle. This is easier to demonstrate with a 2D example.

The secret lies in figure 4. Imagine that we want to find the "image" of a line defined in 2D space by two vertices V0 and V1. The canvas is represented by the horizontal green line. This line is one unit away (along the z-axis) from the coordinate system origin. If we trace lines from V0 and V1 to the origin, then we intersect the green lines in two points (denoted V0' and V1' in the figure). The z-coordinate of this point is 1 since they lie on the canvas which is 1 unit away from the origin. The x-coordinate of the points can easily be computed using perspective projection. We just need to divide the original vertex x-coordinates by their z-coordinate. We get:

$$
\begin{array}{l}
V0'.x = \dfrac{V0.x}{V0.z} = \dfrac{-4}{2} = -2,\\
V1'.x = \dfrac{V1.x}{V1.z} = \dfrac{2}{5} = 0.4.
\end{array}
$$

The goal of the exercise is to find the z-coordinate of P, a point on the line defined by V0 and V1. In this example, all we know about P is the position of its projection P', on the green line. The coordinates of P' are {0,1}. The problem is similar to trying to find the z-coordinate of a point on the triangle that a pixel overlaps. In our example, P' would be the pixel and P would be the point on the triangle that the pixel overlaps. What we need to do now, is compute the "barycentric coordinate" of P' with respect to V0' and V1'. Let's call the resulting value \(\lambda\). Like our triangle barycentric coordinates, \(\lambda\) is also in the range [0,1]. To find \(\lambda\), we just need to take the distance between V0' and P' (along the x-axis), and divide this number by the distance between V0' and V1'. If linearly interpolating the z-coordinates of the original vertices V0 and V1 using \(\lambda\) to find the depth of P works, then we should get the number 4 (we can easily see by just looking at the illustration that the coordinates of P are {0,4}). Let's first compute \(\lambda\):

$$\lambda=\dfrac{P'x - V0'.x}{V1'.x - V0'.x} = \dfrac{0--2}{0.4--2}= \dfrac{2}{2.4} = 0.833.$$

If we now linearly interpolate V0 and V1 z-coordinate to find the P z-coordinate we get:

$$P.z = V0.z * (1-\lambda) + V1.z * \lambda\ = 2 * 1.666 + 5 * 0.833 = 4.5.$$

This is not the value we expect! Interpolating the original vertices z-coordinates, using P's "barycentric coordinates" or \(\lambda\) in this example, to find P z-coordinate doesn't work. Why? The reason is simple. <b>Perspective projection preserves lines but does not preserve distances</b>. It's quite easy to see in figure 4, that the ratio of the distance between V0 and P over the distance between V0 and V1 (0.666) is not the same as the ratio of the distance between V0' and P' over the distance between V0' and V1' (0.833). If \(\lambda\) was equal to 0.666 it would work fine, but here is the problem, it's equal to 0.833 instead! So, how do we find the z-coordinate of P?

The solution to the problem is to compute the inverse of the P z-coordinate by interpolating the inverse of the vertices V0 and V1 z-coordinates using \\(\\lambda\\). In other words, the solution is:

$$\dfrac{1}{P.z} = \color{purple}{\dfrac{1}{V0.z} * (1-\lambda) + \dfrac{1}{V1.z} * \lambda}.$$

Let's check that it works:

$$\dfrac{1}{P.z} = \dfrac{1}{V0.z} * (1-\lambda) + \dfrac{1}{V1.z} * \lambda = \dfrac{1}{2} * (1-2/2.4)+ \dfrac{1}{5} * (2/2.4) = 0.25.$$

If now take the inverse of this result, we get for P z-coordinate the value 4. Which is the correct result! As mentioned before, the solution is to linearly interpolate the vertex's z-coordinates using barycentric coordinates, and invert the resulting number to find the depth of P (its z-coordinate). In the case of our triangle, the formula is:

$$\dfrac{1}{P.z} = \dfrac{1}{V0.z} * \lambda_0 + \dfrac{1}{V1.z} * \lambda_1 + \dfrac{1}{V2.z} * \lambda_2.$$

![Figure 5: perspective projection preserves lines but not distances.](/images/rasterization/perspective-correct.png?)

Let's now look into this problem more formally. Why do we need to interpolate the vertex's inverse z-coordinates? The formal explanation is a bit complicated and you can skip it if you want. Let's consider a line in camera space defined by two vertices whose coordinates are denoted \((X_0,Z_0)\) and \((X_1,Z_1)\). The projection of these vertices on the screen is denoted \(S_0\) and \(S_1\) respectively (in our example, we will assume that the distance between the camera origin and the canvas is 1 as shown in figure 5). Let's call S a point on the line defined by \(S_0\) and \(S_1\). S has a corresponding point P on the 2D line whose coordinates are (X,Z = 1) (we assume in this example that the screen or the vertical line on which the points are projected is 1 unit away from the coordinate system origin). Finally, the parameters \(t\) and \(q\) are defined such that:

$$
\begin{array}{l}
P = P_0 * (1-t) + P_1 * t,\\
S = S_0 * (1-q) + S_1 * q.\\
\end{array}
$$

Which we can also write as:

$$
\begin{array}{l}
P = P_0 + t * (P_1 - P_0),\\
S = S_0 + q * (S_1 - S_0).\\
\end{array}
$$

The (X,Z) coordinates of point P can thus be computed by interpolation (equation 1):

$$(X,Z) = (X_0 + t * (X_1 - X_0), Z_0 + t * (Z_1 - Z_0)).$$

Similarly (equation 2):

$$S = S_0 + q * (S_1 - S_0).$$

S is a 1D point (it has been projected on the screen) thus it has no z-coordinate. S can also be computed as:

$$S = \dfrac{X}{Z}.$$

Therefore:

$$Z = \dfrac{X}{S}.$$

If we replace the numerator with equation 1 and the denominator with equation 2, then we get (equation 3):

$$Z = \dfrac{\color{red}{X_0} + t * (\color{green}{X_1} - \color{red}{X_0})}{S_0 + q * (S_1 - S_0)}$$

We also have:

$$\begin{array}{l}
S_0 = \dfrac{X_0}{Z_0},\\
S_1 = \dfrac{X_1}{Z_1}.
\end{array}
$$

Therefore (equation 4):

$$\begin{array}{l}
\color{red}{X_0 = S_0 * Z_0},\\
\color{green}{X_1 = S_1 * Z_1}.
\end{array}
$$

If now replace \(X_0\) and \(X_1\) in equation 3 with equation 4, we get (equation 5):

$$Z = \dfrac{\color{red}{S_0 * Z_0} + t * (\color{green}{S_1 * Z_1} - \color{red}{S_0 * Z_0})}{S_0 + q * (S_1 - S_0)}$$

Remember from equation 1 that (equation 6):

$$Z = Z_0 + t * (Z_1 - Z_0).$$

If we combine equations 5 and 6 we get:

$$Z_0 + t * (Z_1 - Z_0) = \dfrac{\color{red}{S_0 * Z_0} + t * (\color{green}{S_1 * Z_1} - \color{red}{S_0 * Z_0})}{S_0 + q * (S_1 - S_0)}.$$

Which can be simplified to:

$$
\begin{array}{l}
(Z_0 + t (Z_1 - Z_0))(S_0 + q(S_1 - S_0))=S_0Z_0 + t(S_1Z_1 - S_0Z_0),\\
Z_0S_0 + Z_0q(S_1 - S_0)+t(Z_1 - Z_0)S_0+t (Z_1 - Z_0)q(S_1 - S_0)=S_0Z_0 + t (S_1Z_1 - S_0Z_0),\\
t[(Z_1 - Z_0)S_0 + (Z_1 - Z_0)q(S_1 - S_0) -(S_1Z_1 - S_0Z_0)] =-qZ_0(S_1 - S_0),\\
t[Z_1S_0 - Z_0S_0 + (Z_1 - Z_0)q(S_1 - S_0) - S_1Z_1 + S_0Z_0] =-qZ_0(S_1 - S_0),\\
t(S_1 - S_0)[Z_1 - q(Z_1 - Z_0)]=qZ_0(S_1 - S_0),\\
t[qZ_0 +(1-q)Z_1]=qZ_0.
\end{array}
$$

We can now express the parameter \(t\) in terms of \(q\):

$$t=\dfrac{qZ_0}{qZ_0 +(1-q)Z_1}.$$

If we substitute for t in equation 6, we get:

$$
\begin{array}{l}
Z &= Z_0 + t * (Z_1 - Z_0) = Z_0 + \dfrac{qZ_0(Z_1 - Z_0)}{qZ_0 +(1-q)Z_1},\\
&= \dfrac{qZ_0^2 + (1-q)Z_0Z_1 + qZ_0Z_1 - qZ_0^2}{qZ_0 +(1-q)Z_1},\\
&= \dfrac{Z_0Z_1}{qZ_0 +(1-q)Z_1},\\
&= \dfrac{1}{\dfrac{q}{Z_1} + \dfrac{(1-q)}{Z_0}},\\
&= \dfrac{1}{\dfrac{1}{Z_0} +q (\dfrac{1}{Z1} - \dfrac{1}{Z_0})}.\\
\end{array}
$$

And from there you can write:

$$\begin{array}{l}
\dfrac{1}{Z} &= \dfrac{1}{Z_0} +q (\dfrac{1}{Z1} - \dfrac{1}{Z_0}) = \color{purple}{\dfrac{1}{Z_0}(1-q) + \dfrac{1}{Z_1}q}.
\end{array}
$$

Which is the formula we wanted to end up with.

<details>
You can use a different approach to explain the depth interpolation issue (but we prefer the one above). You can see the triangle (in 3D or camera space) lying on a plane. The plane equation is (equation 1):

$$AX + BY + CZ = D.$$

We know that:

$$
\begin{array}{l}
X_{screen} = \dfrac{X_{camera}}{Z_{camera}},\\
Y_{screen} = \dfrac{Y_{camera}}{Z_{camera}}.\\
\end{array}
$$

Thus:

$$
\begin{array}{l}
X_{camera} = X_{screen}Z_{camera},\\
Y_{camera} = Y_{screen}Z_{camera}.
\end{array}
$$

If we substitute these two equations in equation 1 and solve for \(Z_{camera}\), we get:

$$
\begin{array}{l}
AX_{screen}Z_{camera} + BY_{screen}Z_{camera} + CZ_{camera} = D,\\
Z_{camera}(AX_{screen} + BY_{screen} + C) = D,\\
\dfrac{D}{Z_{camera}} = AX_{screen} + BY_{screen} + C,\\
\dfrac{1}{Z_{camera}} = \dfrac{A}{D}X_{screen} + \dfrac{B}{D}Y_{screen} + \dfrac{C}{D},\\
\dfrac{1}{Z_{camera}} = {A'}X_{screen} + {B'}Y_{screen} + {C'},\\
\end{array}
$$

With: \(A'=\dfrac{A}{D}\), \(B'=\dfrac{B}{D}\), \(C'=\dfrac{C}{D}\).

What this equation shows is that \(1/Z_{camera}\) is an affine function of \(X_{camera}\) and \(Y_{camera}\) which can be interpolated linearly across the surface of the projected triangle (the triangle in screen, NDC or raster space).
</details>

## Other Visible Surface Algorithms

As mentioned in the introduction, the z-buffer algorithm belongs to the family of hidden surface removal or visible surface algorithms. These algorithms can be divided into two categories: the **object space** and **image space algorithms**. The ["painter's" algorithm](http://en.wikipedia.org/wiki/Painter's_algorithm) which we haven't talked about in this lesson belongs to the former, while the z-buffer algorithm belongs to the latter type. The concept behind the painter's algorithm is roughly to paint or draw objects from back to front. This technique requires objects to be sorted in depth. As explained earlier in this chapter, first objects are passed down to the renderer in arbitrary order, and then when two triangles intersect each other, it becomes difficult to figure out which one is in front of the other (thus deciding which one should be drawn first). This algorithm is not used anymore but the z-buffer is very common (GPUs use it).