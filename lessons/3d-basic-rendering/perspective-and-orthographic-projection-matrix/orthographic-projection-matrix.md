## What Do I Need Orthographic Projection For?

The **orthographic projection** (also sometimes called oblique projection) is simpler than the other type of projections and learning about it is a good way of apprehending how the perspective projection matrix works. You might think that orthographic projections are of no use today. Indeed, what people strive for whether in films or games, is photorealism for which perspective projection is generally used. Why bother then with learning about parallel projection? Images rendered with an orthographic camera can give a certain look to a video game that is better (sometimes) than the more natural look produced with perspective projection. Famous games such the Sims or Sim City adopted this look. Orthographic projection can also be used to render shadow maps, or render orthographic views of a 3D model for practical reasons: an architect for example may need to produce blueprints from the 3D model of a house or building designed in a CAD software.

In this chapter we will learn how to create a matrix that project a point in camera space to a point projected onto the image plane of an orthographic camera.

The goal of this orthographic projection matrix is to actually remap all coordinates contained within a certain bounding box in 3D space into the **canonical viewing volume** (we introduced this concept already in chapter 2). If you wonder what that original box is, then just imagine that this is a bounding box surrounding all the objects contained in your scene. Very simply, you can loop over all the objects in your scene, compute there bounding box and extent the dimensions of the global bounding box so that it contains all the bounding volumes of these objects. Once you have the bounding box for the scene, then the goal of the orthographic matrix is to remap it to a canonical view volume. This volume is a box which minimum and maximum extents are respectively (-1, -1, -1) and (1, 1, 1) (or (-1,-1,0) and (1,1,1) depending on the convention you are using). In other words, the x- and y-coordinate of the projected point are remapped from wherever they were before the projection, to the range [-1,1]. The z-coordinate is remapped to the range [-1,1] (or [0,1]). Note that both bounding boxes (the scene bounding box and the canonical view volume) are AABBs (axis-aligned bounding boxes) which simplifies the remapping process a lot.

![Figure 1: the orthographic projection matrix remaps the scene bounding volume to the canonical view volume. All points contained in the scene bounding volume have their projected xy coordinates "normalized" (they lie within the range [-1,1])](/images/perspective-matrix/ortho-proj3.png?)

Once we have computed the scene bounding box, we need to project the minimum and maximum extents of this bounding box onto the image plane of the camera. In the case of an orthographic projection (or parallel projection) this is trivial. The x- and y-coordinates of any point expressed in camera space and the x- and y-coordinates of the same points projected on the image plane are the same. You need to potentially extend the projection of the minimum and maximum extents of the bounding box onto the screen in order for the screen window itself to be either square or have the same ratio than the image aspect ratio (check the code of the test program below to see how this can be done). Remember also that the canvas or screen is centred around the screen coordinate system origin (figure 2).

![Figure 2: setting up the screen coordinates from the scene bounding box.](/images/perspective-matrix/ortho-proj4.png?)

We will name these screen coordinates l, r, t, b which stand for left, right, top and bottom.

We now need to remap the left and right screen coordinates (l, r) to -1 and 1 and do the same for the bottom and right coordinates. Let see how we can do that. Let's assume that x is any point contained within the range [l,r]. We can write:

$$l \le x \le r.$$

We can remove l from all the terms and write:

$$0 \le {x - l} \le {r - l}.$$

And if we want the term on the right to be 1 instead of r we can divide everything by r which gives:

$$0 \le \dfrac{x - l}{r - l} \le 1.$$

We can also multiply everything by 2 (you will understand why soon) which gives us:

$$0 \le 2 \dfrac{x - l}{r - l} \le 2.$$

And substract -1 to all the terms which gives us:

$$-1 \le { 2 \dfrac{x - l}{r - l} - 1} \le 1.$$

You can now see that the term in the middle is contained between the lower limit -1 and the upper limit 1\. We have managed to remap the term in the middle to the range [-1,1]. Let's develop this formula further:

$$-1 \le { 2 \dfrac{x - l}{r - l} -  \dfrac{r-l}{r-l}} \le 1.$$

$$-1 \le \dfrac{2x - 2l - r + l}{r - l} \le 1.$$

$$-1 \le \dfrac{2x - l - r}{r - l} \le 1.$$

$$-1 \le \dfrac{2x}{r - l} -  \dfrac{r + l}{r - l} \le 1.$$

We now have the formula to transform x.

$$x'= \dfrac{2x}{r - l} - \dfrac{r + l}{r - l}.$$

We need to write this formula, in the form of a matrix (if you have read the third chapter, this step should be easy to understand):

$$
\begin{bmatrix}
\dfrac{2}{r - l} &amp; 0 &amp; 0 &amp; 0 \\
0 &amp; 1 &amp; 0 &amp; 0 \\
0 &amp; 0 &amp; 1 &amp; 0\\ -\dfrac{r + l}{r - l} &amp; 0 &amp; 0 &amp; 1
\end{bmatrix}
$$

The process for the y-coordinate is exactly the same. You just need to replace, r and l with t and b (top and bottom). The matrix becomes:

$$
\begin{bmatrix}
\dfrac{2}{r - l} &amp; 0 &amp; 0 &amp; 0 \\
0 &amp; \dfrac{2}{t - b}  &amp; 0 &amp; 0 \\
0 &amp; 0 &amp; 1 &amp; 0\\ -\dfrac{r + l}{r - l} &amp; -\dfrac{t + b}{t - b} &amp; 0 &amp; 1
\end{bmatrix}
$$

And finally to complete our orthographic projection matrix, we need to remap the z coordinates from -1 to 1\. We will use the same principle to find a formula for z. We start with the following condition:

$$n \le -z \le f.$$

Don't forget that because we use a right hand coordinate system, the z-coordinates of all points visible by the camera are negative, which is the reason we use -z instead of z. We set the term on the left to 0:

$$n \le -z \le f.$$ $$0 \le -z - n \le f - n.$$

Then divide everything by (f-n) to normalize the term on the right:

$$0 \le \dfrac{(-z - n)}{(f-n)} \le 1.$$

Multiply all the terms by two:

$$0 \le 2 \dfrac{(-z-n)}{(f-n)} \le 2.$$

Remove one:

$$-1 \le 2 \dfrac{(-z-n)}{(f-n)} -1 \le 1.$$

Which we can re-write as:

$$-1 \le 2\dfrac{(-z-n)}{(f-n)} - \dfrac{f-n}{(f-n)} \le 1.$$

If we develop the terms we get:

$$-1 \le \dfrac {(-2z -2n -f + n)}{(fn-n)} \le 1.$$

Re-arranging the terms give:

$$z'=\dfrac{-2z}{(f-n)} - \dfrac{f+n}{(f-n)}.$$

Let's add these two terms to the matrix:

$$
\small \begin{bmatrix}
\dfrac{2}{r - l} &amp; 0 &amp; 0 &amp; 0 \\
0 &amp; \dfrac{2}{t - b} &amp; 0 &amp; 0 \\
0 &amp; 0 &amp; {\color{\red}{ \dfrac{-2}{(f-n)}}} &amp; 0\\ -\dfrac{r + l}{r - l} &amp; -\dfrac{t + b}{t - b} &amp; {\color{\red}{ -\dfrac{(f + n)}{(f-n)}}} &amp; 1
\end{bmatrix}
$$

Remember that that OpenGL uses a column-major convention to encode matrix. Scratchapixel uses a right-major notation. To get from one notation to the other you need to transpose the matrix. Here is the final OpenGL orthographic matrix as you will see it in text books:

$$
\begin{bmatrix}
\dfrac{2}{r - l} &amp; 0 &amp; 0 &amp; -\dfrac{r + l}{r - l} \\
0 &amp; \dfrac{2}{t - b} &amp; 0 &amp;  -\dfrac{t + b}{t - b} \\
0 &amp; 0 &amp; {\color{\red}{\dfrac{-2}{(f-n)}}} &amp; {\color{\red}{-\dfrac{f+n}{(f-n)}}}\\ 
0 &amp;0 &amp; 0 &amp; 1
\end{bmatrix}
$$

## Text Program

As usual, we will test the matrix with a simple test program. We will re-use the same code than the one we used to test the simple and the OpenGL perspective projection matrix. We have replaced the function `glFrustum` with a function called `glOrtho`, which as its name suggests, is used to set an OpenGL orthographic matrix. The screen coordinates (the l, r, t and b parameters of the function) are computed as follows (lines 63-83): the bounding box of the scene is computed (we simply iterate over all vertices in the scene and extend the minimum and maximum extend variables accordingly). The bounding box minimum and maximum extents are then transformed from world to camera space. Of the two resulting points in camera space, we then find the maximum dimension in both x and y. We finally set the screen coordinates to the maximum of these two values. This guarantees that the screen coordinates form a square (you may need to multiply the left and right coordinates by the image aspect ratio if the latter is different than 1) and that the screen or canvas itself is centred around the screen space coordinate system origin.

The rest of the code is as usual. We loop over the vertex in the scene. Vertices are transformed from world to camera space and are then projected onto the screen using the OpenGL orthographic projection matrix.

```
#include <cstdio> 
#include <cstdlib> 
#include <fstream> 
#include <limits> 
#include "geometry.h" 
#include "vertexdata.h" 
 
// set the OpenGL orthographic projection matrix
void glOrtho( 
    const float &b, const float &t, const float &l, const float &r, 
    const float &n, const float &f, 
    Matrix44f &M) 
{ 
    // set OpenGL perspective projection matrix
    M[0][0] = 2 / (r - l); 
    M[0][1] = 0; 
    M[0][2] = 0; 
    M[0][3] = 0; 
 
    M[1][0] = 0; 
    M[1][1] = 2 / (t - b); 
    M[1][2] = 0; 
    M[1][3] = 0; 
 
    M[2][0] = 0; 
    M[2][1] = 0; 
    M[2][2] = -2 / (f - n); 
    M[2][3] = 0; 
 
    M[3][0] = -(r + l) / (r - l); 
    M[3][1] = -(t + b) / (t - b); 
    M[3][2] = -(f + n) / (f - n); 
    M[3][3] = 1; 
} 
 
void multPointMatrix(const Vec3f &in, Vec3f &out, const Matrix44f &M) 
{ 
    //out = in * Mproj;
    out.x   = in.x * M[0][0] + in.y * M[1][0] + in.z * M[2][0] + /* in.z = 1 */ M[3][0]; 
    out.y   = in.x * M[0][1] + in.y * M[1][1] + in.z * M[2][1] + /* in.z = 1 */ M[3][1]; 
    out.z   = in.x * M[0][2] + in.y * M[1][2] + in.z * M[2][2] + /* in.z = 1 */ M[3][2]; 
    float w = in.x * M[0][3] + in.y * M[1][3] + in.z * M[2][3] + /* in.z = 1 */ M[3][3]; 
 
    // normalize if w is different than 1 (convert from homogeneous to Cartesian coordinates)
    if (w != 1) { 
        out.x /= w; 
        out.y /= w; 
        out.z /= w; 
    } 
} 
 
int main(int argc, char **argv) 
{ 
    uint32_t imageWidth = 512, imageHeight = 512; 
    Matrix44f Mproj; 
    Matrix44f worldToCamera = {0.95424, 0.20371, -0.218924, 0, 0, 0.732087, 0.681211, 0, 0.299041, -0.650039, 0.698587, 0, -0.553677, -3.920548, -62.68137, 1}; 
 
    float near = 0.1; 
    float far = 100; 
    float imageAspectRatio = imageWidth / (float)imageHeight;  // 1 if the image is square 
 
    // compute the scene bounding box
    const float kInfinity = std::numeric_limits<float>::max(); 
    Vec3f minWorld(kInfinity), maxWorld(-kInfinity); 
    for (uint32_t i = 0; i < numVertices; ++i) { 
        if (vertices[i].x < minWorld.x) minWorld.x = vertices[i].x; 
        if (vertices[i].y < minWorld.y) minWorld.y = vertices[i].y; 
        if (vertices[i].z < minWorld.z) minWorld.z = vertices[i].z; 
        if (vertices[i].x > maxWorld.x) maxWorld.x = vertices[i].x; 
        if (vertices[i].y > maxWorld.y) maxWorld.y = vertices[i].y; 
        if (vertices[i].z > maxWorld.z) maxWorld.z = vertices[i].z; 
    } 
 
    Vec3f minCamera, maxCamera; 
    multPointMatrix(minWorld, minCamera, worldToCamera); 
    multPointMatrix(maxWorld, maxCamera, worldToCamera); 
 
    float maxx = std::max(fabs(minCamera.x), fabs(maxCamera.x)); 
    float maxy = std::max(fabs(minCamera.y), fabs(maxCamera.y)); 
    float max = std::max(maxx, maxy); 
    float r = max * imageAspectRatio, t = max; 
    float l = -r, b = -t; 
 
    glOrtho(b, t, l, r, near, far, Mproj); 
    unsigned char *buffer = new unsigned char[imageWidth * imageHeight]; 
    memset(buffer, 0x0, imageWidth * imageHeight); 
    for (uint32_t i = 0; i < numVertices; ++i) { 
        Vec3f vertCamera, projectedVert; 
        multPointMatrix(vertices[i], vertCamera, worldToCamera); 
        multPointMatrix(vertCamera, projectedVert, Mproj); 
        if (projectedVert.x < -imageAspectRatio || projectedVert.x > imageAspectRatio || projectedVert.y < -1 || projectedVert.y > 1) continue; 
        // convert to raster space and mark the position of the vertex in the image with a simple dot
        uint32_t x = std::min(imageWidth - 1, (uint32_t)((projectedVert.x + 1) * 0.5 * imageWidth)); 
        uint32_t y = std::min(imageHeight - 1, (uint32_t)((1 - (projectedVert.y + 1) * 0.5) * imageHeight)); 
        buffer[y * imageWidth + x] = 255; 
    } 
    // export to image
    std::ofstream ofs; 
    ofs.open("./out.ppm"); 
    ofs << "P5\n" << imageWidth << " " << imageHeight << "\n255\n"; 
    ofs.write((char*)buffer, imageWidth * imageHeight); 
    ofs.close(); 
    delete [] buffer; 
 
    return 0; 
}
```

As usual, we check the result of our program against a render of the same geometry using the same camera and the same render settings. Overlaying the program's output on the reference render shows that the program produces the same result.

![](/images/perspective-matrix/ortho-render.png?)