We have covered everything there is to say! We are now prepared to write our first ray-tracer. You should now be able to guess how the ray-tracing algorithm works.

First of all, take a moment to notice that the propagation of light in nature is just a countless number of rays emitted from light sources that bounce around until they hit the surface of our eye. Ray-tracing is, therefore, elegant in the way that it is based directly on what actually happens around us. Apart from the fact that it follows the path of light in the reverse order, it is nothing less that a perfect nature simulator.

The ray-tracing algorithm takes an image made of pixels. For each pixel in the image, it shoots a primary ray into the scene. The direction of that primary ray is obtained by tracing a line from the eye to the center of that pixel. Once we have that primary ray's direction set, we check every object of the scene to see if it intersects with any of them. In some cases, the primary ray will intersect more than one object. When that happens, we select the object whose intersection point is the closest to the eye. We then shoot a shadow ray from the intersection point to the light (Figure 6, top). If this particular ray does not intersect an object on its way to the light, the hit point is illuminated. If it does intersect with another object, that object casts a shadow on it (figure 2).

If we repeat this operation for every pixel, we obtain a two-dimensional representation of our three-dimensional scene (figure 3).

Here is an implementation of the algorithm in pseudocode: