## Interpolating Vertex Attributes

All we need to get a basis rasterizer working is to know how to project triangles onto the screen, convert the projected coordinates to raster space, then rasterize triangles, and potentially use a depth-buffer to solve the visibility problem. That would already be enough to create images of 3D scenes, which would be both perspective correct, and in which the visibility problem would be solved (objects that are hidden by others, are indeed not appearing in front of objects that are supposed to occlude them). This is already a good result. The code we have presented to do this is functional, but could be optimized greatly; however, optimizing the rasterization algorithm is not something we will look into in this lesson.

## Perspective Correct Vertex Attribute Interpolation

So, what are we going to talk about in this chapter? In the chapter devoted to rasterization, we talked about using barycentric coordinates to interpolate vertex data or vertex attributes (which is the most common name). We can use barycentric coordinates to interpolate depth values of course, and this is one of their main function, but they can also be used to interpolate vertex attributes; vertex attributes play a very important role in rendering, especially when it comes to lighting and shading. We will provide more information on how vertex attributes are used in shading when we get to shading in this section. But you don't need to know anything about shading to understand the concept of **perspective correct interpolation** and vertex attributes.

You might wonder why we don't talk about this topic in shading if it then relates to shading more specifically. Vertex attributes do relate to shading more specifically but the topic of perspective correct interpolation relates more specifically to the topic of rasterization.

As you already know, we need to store the z-coordinates of the original vertices (camera space vertices) in the z-coordinate of our projected vertices (screen space vertices). This is required so that we can compute the depth of points lying across the surface of the projected triangle. Depth, as explained in the last chapter, is required to solve the visibility problem and is computed by linearly interpolating the reciprocal of the triangle vertices z-coordinates using barycentric coordinates. Though the same technique can be used to interpolate any other variable we want as long as the value of this variable is defined at the triangle's vertices, similarly to the way we've stored in the projected point z-coordinates, the vertex's original z-coordinates. For example, it is very common to store at the triangle vertices a color. The other two most common variables or attributes (which is the proper term in CG) stored at the triangle vertices are texture coordinates and normals. Texture coordinates are 2D coordinates used for texturing (a technique we will study in this section). Normals are used in shading and define the orientation of the surface (check the lessons on shading to learn more about normals and smooth shading in particular). In this lesson, we will more specifically use color and texture coordinates to illustrate the problem of perspective-correct interpolation.

![Figure 1: Finding the value of Cp requires to interpolate the colors defined at the triangle vertices using P's barycentric coordinates.](/images/rasterization/persp-correct-interpo2.png?)

As mentioned in the chapter on the rasterization stage, we can specify colors or anything else we want for the triangle vertices. These attributes can be interpolated using barycentric coordinates to find what the value of these attributes should be for any point inside the triangle. In other words, **vertex attributes must be interpolated across the surface of a triangle when it is rasterized**. The process is as follows:

- You can assign as many vertexes attributes to the triangle's vertices as you want. They are defined on the original 3D triangle (in camera space). In our example, we will assign two vertex attributes, one for color and one for texture coordinates.
- The triangle is projected onto the screen (the triangle's vertices are converted from camera space to raster space).
- While in screen space, the triangle is "rasterized". If a pixel overlaps the triangle, the barycentric coordinates of that pixel are computed.
- Colors (or texture coordinates) defined at the triangle corners or vertices are interpolated using the previously computed barycentric coordinates using the following formula:

  $$C_P = \lambda_0 * C_0 + \lambda_1 * C_1 + \lambda_2 * C_2$$

  Where \(\lambda_0\), \(\lambda_1\) and \(\lambda_2\) are the pixel's barycentric coordinates, and \(C_0\), \(C_1\) and \(C_2\) are the colors for the triangle vertices. The result \(C_P\) is assigned to the current pixel in the frame-buffer. The same can be done to compute the texture coordinates of the point on the triangle that the pixel overlaps. 

  $$ST_P = \lambda_0 * ST_0 + \lambda_1 * ST_1 + \lambda_2 * ST_2$$

  These coordinates are used for texturing (check the lesson on Texture Mapping in this section if you wish to learn about texture coordinates and texturing).

![Figure 2: in 3D space, the point P is in the middle of the quad, though in the perspective view, this point doesn't seem to be in the middle of the geometry. In 3D space, point P on the green triangle's edge is exactly in the middle of V1 and V2. But in the bottom image, we can see that P is closer to V1 than it is to V2.](/images/rasterization/persp-correct-interpo1.png?)

This technique though just doesn't work. To understand why let's see what happens to a point located in the middle of a 3D quad. As you can see in the top view of figure 2, we have a quad and point P, is clearly in the middle of that quad (P is located at the intersection of the quad's diagonals). Though, when we look at this good from a random viewpoint it is easy to see that depending on the quad's orientation with respect to the camera, P doesn't appear to be in the center of the quad anymore. This is due to a perspective projection which as mentioned before, preserves lines but doesn't preserve distances. Though remember that barycentric coordinates are computed in screen space. Imagine that the quad is made of two triangles. In 3D space, P is located at an equal distance between V1-V2 thus somehow its barycentric coordinates in 3D space are (0,0.5,0.5). Though, in screen space, since P is closer to V1 than it is to V2, then \(\lambda_1\) is greater than \(\lambda_2\) (and \lambda_0 is equal to 0). The problem though is that these are the coordinates that are used to interpolate the triangle's vertex attributes. If V1 is white and V2 is black then the color at P should be 0.5. But if \(\lambda_1\) is greater than \(\lambda_2\) then we will get a value greater than 0.5. Thus clearly the technique we are using to interpolate vertex attributes doesn't work. Let's assume like in figure 1 that \(\lambda_1\) and \(\lambda_2\) are equal to 0.666 and 0.334 respectively. If we interpolate the triangle's vertex colors, we get:

$$C_P = \lambda_0 * C_0 + \lambda_1 * C_1 + \lambda_2 * C_2 = 0 * C_0 + 0.666 * 1 + 0.334 * 0 = 0.666.$$

We get 0.666 for the color of P and not 0.5 as we should. There is a problem and this problem relates to some extent to what we learned in the previous chapter regarding the interpolation of the vertex's z-coordinates.

![Figure 3: the ratio \((Z-Z_0)/(Z_1-Z_0)\) matches the ratio \((C-C_0)/(C_1-C_0)\). We already know from the previous chapter how to compute \(Z\). We can use these two equations to solve for \(C\).](/images/rasterization/persp-correct-interpo3.png?)

Hopefully finding the right solution is not hard. Let's imagine that we have a triangle with two z-coordinates \(Z_0\) and \(Z_1\) on each side of the triangle as shown in figure 3. If we connect these two points we can interpolate the z-coordinate of a point on this line using linear interpolation. We can do the same thing with two values of a vertex attributes \(C_0\) and \(C_1\) defined at the same positions on the triangle as \(Z_0\) and \(Z_1\) respectively. Technically, because both \(Z\) and \(C\) are computed using linear interpolation, we can write the following equality (equation 1):

$$\dfrac{Z - Z_0}{Z_1 - Z_0} = \dfrac {C-C_0}{C_1 - C_0}.$$

We also know from the last chapter that (equation 2):

$$Z = \dfrac{1}{\dfrac{1}{Z_0}(1-q) + \dfrac{1}{Z_1}q}.$$

The first thing we are going to do is substitute the equation for Z (equation 2) on the left-hand side of equation 1. The trick to simplifying the resulting equation is to multiply the numerator and denominator of equation 2 by \\(Z\_0Z\_1\\) to get rid of the \\(1/Z\_0\\) and \\(1/Z\_1\\) terms (this is of course not explained anywhere but here on Scratchapixel):

$$
\begin{align}
\dfrac{\dfrac{1}{\dfrac{1}{Z_0}(1-q)+\dfrac{1}{Z_1}q} - Z_0}{(Z_1 - Z_0)} & = \dfrac{\dfrac{Z_0Z_1}{Z_1(1-q)+Z_0q} - Z_0}{(Z_1 - Z_0)}\\
& = \dfrac{\dfrac{Z_0Z_1 - Z_0(Z_1(1-q) + Z_0q)}{Z_1(1-q) + Z_0q}}{Z_1 - Z_0}\\
& = \dfrac{\dfrac{Z_0Z_1q - Z_0^2q}{Z_1(1-q) + Z_0q}}{Z_1 - Z_0}\\
& = \dfrac{\dfrac{Z_0q(Z_1 - Z_0)}{Z_1(1-q) + Z_0q}}{Z_1 - Z_0}\\
& = \dfrac{Z_0q}{Z_1(1-q) + Z_0q}\\
& = \dfrac{Z_0q}{q(Z_0 - Z_1) + Z_1}.\\
\end{align}
$$

We can now solve for \(C\):

$$
\begin{array}{l}
\dfrac{Z_0q}{q(Z_0 - Z_1) + Z_1} = \dfrac {C-C_0}{C_1 - C_0},\\
C  = \dfrac{Z_0q}{q(Z_0 - Z_1) + Z_1}(C_1 - C_0) + C_0,\\
C  = \dfrac{Z_0q(C_1 - C_0) + C_0(q(Z_0 - Z_1) + Z_1)}{q(Z_0 - Z_1) + Z_1},\\
C  = \dfrac{Z_0qC_1 - Z_0qC_0 + C_0qZ_0 - C_0qZ_1 + C_0Z_1}{q(Z_0 - Z_1) + Z_1},\\
C  = \dfrac{Z_0qC_1 - C_0qZ_1 + C_0Z_1}{q(Z_0 - Z_1) + Z_1},\\
C  = \dfrac{C_0Z_1(1-q) + C_1Z_0q}{Z_1(1 - q) + Z_0q}.\\
\end{array}
$$

If we now multiply the numerator and the denominator by \(1/Z_0Z_1\), we can then extract a factor of \(Z\) from the right-hand side of the equation:

$$
\begin{array}{l}
C  = \dfrac{\dfrac{1}{Z_0Z_1}(C_0Z_1(1-q) + C_1Z_0q)}{\dfrac{1}{Z_0Z_1}(Z_1(1 - q) + Z_0q)},\\
C  = \dfrac{\dfrac{C_0}{Z_0}(1-q) + \dfrac{C_1}{Z_1}q}{\dfrac{1}{Z_0}(1 - q) + \dfrac{1}{Z_1}q},\\
C = Z \left [\dfrac{C_0}{Z_0}(1-q) + \dfrac{C_1}{Z_1}q \right ].
\end{array}
$$

It took a while to get to that result, but this a very fundamental equation in rasterization (which by the way is rarely explained anywhere. It is sometimes explained but the steps to get to that result are rarely provided) because it is used to interpolate vertex attributes which is a very important and common feature in rendering. What the equation says is that to interpolate a vertex attribute correctly, we first need to divide the vertex attribute value by the z-coordinate of the vertex it is defined to, then linearly interpolate them using q (which in our case, will be the barycentric coordinates of the pixel on the 2D triangle), and then finally multiply the result by \(Z\), which is the depth of the point on the triangle, that the pixel overlaps (the depth of the point in camera space where the vertex attribute is being interpolated). Here is another version of the code we used in chapter three, that shows an example of **perspective correct vertex attribute interpolation**:

```
// compile with:
// c++ -o raster3d raster3d.cpp
// for naive vertex attribute interpolation and:
// c++ -o raster3d raster3d.cpp -D PERSP_CORRECT
// for perspective correct interpolation 

// (c) www.scratchapixel.com

#include &ltcstdio&gt
#include &ltcstdlib&gt
#include &ltfstream&gt

typedef float Vec2[2];
typedef float Vec3[3];
typedef unsigned char Rgb[3];

inline
float edgeFunction(const Vec3 &a, const Vec3 &b, const Vec3 &c)
{ return (c[0] - a[0]) * (b[1] - a[1]) - (c[1] - a[1]) * (b[0] - a[0]); }

int main(int argc, char **argv)
{
    Vec3 v2 = { -48, -10,  82};
    Vec3 v1 = {  29, -15,  44};
    Vec3 v0 = {  13,  34, 114};
    Vec3 c2 = {1, 0, 0};
    Vec3 c1 = {0, 1, 0};
    Vec3 c0 = {0, 0, 1};
    
    const uint32_t w = 512;
    const uint32_t h = 512;
    
    // project triangle onto the screen
    v0[0] /= v0[2], v0[1] /= v0[2];
    v1[0] /= v1[2], v1[1] /= v1[2];
    v2[0] /= v2[2], v2[1] /= v2[2];
    // convert from screen space to NDC then raster (in one go)
    v0[0] = (1 + v0[0]) * 0.5 * w, v0[1] = (1 + v0[1]) * 0.5 * h;
    v1[0] = (1 + v1[0]) * 0.5 * w, v1[1] = (1 + v1[1]) * 0.5 * h;
    v2[0] = (1 + v2[0]) * 0.5 * w, v2[1] = (1 + v2[1]) * 0.5 * h;

#ifdef PERSP_CORRECT
    // divide vertex-attribute by the vertex z-coordinate
    c0[0] /= v0[2], c0[1] /= v0[2], c0[2] /= v0[2];
    c1[0] /= v1[2], c1[1] /= v1[2], c1[2] /= v1[2];
    c2[0] /= v2[2], c2[1] /= v2[2], c2[2] /= v2[2];
    // pre-compute 1 over z
    v0[2] = 1 / v0[2], v1[2] = 1 / v1[2], v2[2] = 1 / v2[2];
#endif
    
    Rgb *framebuffer = new Rgb[w * h];
    memset(framebuffer, 0x0, w * h * 3);
    
    float area = edgeFunction(v0, v1, v2);
    
    for (uint32_t j = 0; j &lt h; ++j) {
        for (uint32_t i = 0; i &lt w; ++i) {
            Vec3 p = {i + 0.5, h - j + 0.5, 0};
            float w0 = edgeFunction(v1, v2, p);
            float w1 = edgeFunction(v2, v0, p);
            float w2 = edgeFunction(v0, v1, p);
            if (w0 &gt= 0 && w1 &gt= 0 && w2 &gt= 0) {
                w0 /= area;
                w1 /= area;
                w2 /= area;
                float r = w0 * c0[0] + w1 * c1[0] + w2 * c2[0];
                float g = w0 * c0[1] + w1 * c1[1] + w2 * c2[1];
                float b = w0 * c0[2] + w1 * c1[2] + w2 * c2[2];
#ifdef PERSP_CORRECT
                float z = 1 / (w0 * v0[2] + w1 * v1[2] + w2 * v2[2]);
                // if we use perspective-correct interpolation we need to
                // multiply the result of this interpolation by z, the depth
                // of the point on the 3D triangle that the pixel overlaps.
                r *= z, g *= z, b *= z;
#endif
                framebuffer[j * w + i][0] = (unsigned char)(r * 255);
                framebuffer[j * w + i][1] = (unsigned char)(g * 255);
                framebuffer[j * w + i][2] = (unsigned char)(b * 255);
            }
        }
    }
    
    std::ofstream ofs;
    ofs.open("./raster2d.ppm");
    ofs << "P6\n" &lt&lt w &lt&lt " " &lt&lt h &lt&lt "\n255\n";
    ofs.write((char*)framebuffer, w * h * 3);
    ofs.close();
    
    delete [] framebuffer;
    
    return 0;
}
```

Computing the sample death requires to use of the reciprocal of the vertex's z-coordinates. For this reason, we can pre-compute these values before we loop over all pixels (line 52). If we decide to use perspective correct interpolation, then the vertex attribute values are divided by the z-coordinate of the vertex they are associated with (lines 48-50). The following image shows on the left, an image computed without perspective correct interpolation, an image with (middle) and the content of the z-buffer (as a greyscale image. The closer the object is to the screen, the brighter). The difference is subtle though you can see in the left image how each color seems to roughly fill up the same area. This is because colors in this case are interpolated within the "space" of the 2D triangle (as if the triangle was a flat surface parallel to the plane of the screen). However, if you inspect the triangle vertices (and the depth buffer), you will notice that the triangle is not at all parallel to the screen (but oriented with a certain angle). Because the vertex "painted" in green is closer to the camera than the other two, this part of the triangle fills up a larger part of the screen which is visible in the middle image (the green area is larger than the blue or red area). The image in the middle shows the correct interpolation and what you will get if you render this triangle with a graphics API such as OpenGL or Direct3D.

![](/images/rasterization/persp-correct-interpo4.png?)

The difference between correct and incorrect perspective interpolation is even more visible when applied to texturing. In the next example, we assigned texture coordinates to the triangle vertices as vertex attributes, and use these coordinates to create a checkerboard pattern for the triangle. Rendering the triangle with or without perspective correct interpolation is left as an exercise. In the image below you can see the result. As you can see, it also matches an image of the same triangle with the same pattern rendered in Maya. Hopefully, our code so far seems to do the right thing. As with color, all you need to do (and this is true of all vertex attributes) is to divide the texture coordinates (which are generally denoted ST coordinates) by the z-coordinate of the vertex they are associated to, and then later in the code, multiply the texture coordinate interpolated value by Z. Here are the changes we made to the code:

```
...
int main(int argc, char **argv)
{
    Vec3 v2 = { -48, -10,  82};
    Vec3 v1 = {  29, -15,  44};
    Vec3 v0 = {  13,  34, 114};
    ...
    Vec2 st2 = { 0, 0 };
    Vec2 st1 = { 1, 0 };
    Vec2 st0 = { 0, 1 };
    
    ...

#ifdef PERSP_CORRECT
    // divide vertex-attribute by the vertex z-coordinate
    c0[0] /= v0[2], c0[1] /= v0[2], c0[2] /= v0[2];
    c1[0] /= v1[2], c1[1] /= v1[2], c1[2] /= v1[2];
    c2[0] /= v2[2], c2[1] /= v2[2], c2[2] /= v2[2];

    st0[0] /= v0[2], st0[1] /= v0[2];
    st1[0] /= v1[2], st1[1] /= v1[2];
    st2[0] /= v2[2], st2[1] /= v2[2];

    // pre-compute 1 over z
    v0[2] = 1 / v0[2], v1[2] = 1 / v1[2], v2[2] = 1 / v2[2];
#endif
    
    ...
    
    for (uint32_t j = 0; j &lt h; ++j) {
        for (uint32_t i = 0; i &lt w; ++i) {
            ...
            if (w0 &gt= 0 && w1 &gt= 0 && w2 &gt= 0) {
                ...
                float s = w0 * st0[0] + w1 * st1[0] + w2 * st2[0];
                float t = w0 * st0[1] + w1 * st1[1] + w2 * st2[1];
#ifdef PERSP_CORRECT
                float z = 1 / (w0 * v0[2] + w1 * v1[2] + w2 * v2[2]);
                // if we use perspective correct interpolation we need to
                // multiply the result of this interpolation by z, the depth
                // of the point on the 3D triangle that the pixel overlaps.
                s *= z, t *= z;
#endif
                const int M = 10;
                // checkerboard pattern
                float p = (fmod(s * M, 1.0) > 0.5) ^ (fmod(t * M, 1.0) < 0.5);
                framebuffer[j * w + i][0] = (unsigned char)(p * 255);
                framebuffer[j * w + i][1] = (unsigned char)(p * 255);
                framebuffer[j * w + i][2] = (unsigned char)(p * 255);
                ...
            }
        }
    }
    
    ...
    
    return 0;
}
```

Here is the result you should get:

![](/images/rasterization/persp-correct-interpo5.png?)

## What's Next?

In the final chapter of this lesson, we will briefly speak about ways of improving the rasterization algorithm (though we won't implement these techniques specifically) and explain how the final code of our lesson works.