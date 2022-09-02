![Figure 1: we shoot a primary ray through the center of the pixel to check for a possible object intersection. When we find one we then cast a shadow ray to find out if the point is illuminated or in shadow]()

![Figure 2: the small sphere cast a shadow on the large sphere. The shadow ray intersects the small sphere before it gets to the light.]()

![Figure 3: to render a frame, we shoot a primary ray for each pixel of the frame buffer]()

We have covered everything there is to say! We are now prepared to write our first ray-tracer. You should now be able to guess how the ray-tracing algorithm works.

First of all, take a moment to notice that the propagation of light in nature is just a countless number of rays emitted from light sources that bounce around until they hit the surface of our eye. Ray-tracing is, therefore, elegant in the way that it is based directly on what actually happens around us. Apart from the fact that it follows the path of light in the reverse order, it is nothing less that a perfect nature simulator.

The ray-tracing algorithm takes an image made of pixels. For each pixel in the image, it shoots a primary ray into the scene. The direction of that primary ray is obtained by tracing a line from the eye to the center of that pixel. Once we have that primary ray's direction set, we check every object of the scene to see if it intersects with any of them. In some cases, the primary ray will intersect more than one object. When that happens, we select the object whose intersection point is the closest to the eye. We then shoot a shadow ray from the intersection point to the light (Figure 6, top). If this particular ray does not intersect an object on its way to the light, the hit point is illuminated. If it does intersect with another object, that object casts a shadow on it (figure 2).

If we repeat this operation for every pixel, we obtain a two-dimensional representation of our three-dimensional scene (figure 3).

Here is an implementation of the algorithm in pseudocode:

```
for (int j = 0; j < imageHeight; ++j) { 
    for (int i = 0; i < imageWidth; ++i) { 
        // compute primary ray direction
        Ray primRay; 
        computePrimRay(i, j, &primRay); 
        // shoot prim ray in the scene and search for intersection
        Point pHit; 
        Normal nHit; 
        float minDist = INFINITY; 
        Object object = NULL; 
        for (int k = 0; k < objects.size(); ++k) { 
            if (Intersect(objects[k], primRay, &pHit, &nHit)) { 
                float distance = Distance(eyePosition, pHit); 
                if (distance < minDistance) { 
                    object = objects[k]; 
                    minDistance = distance;  //update min distance 
                } 
            } 
        } 
        if (object != NULL) { 
            // compute illumination
            Ray shadowRay; 
            shadowRay.direction = lightPosition - pHit; 
            bool isShadow = false; 
            for (int k = 0; k < objects.size(); ++k) { 
                if (Intersect(objects[k], shadowRay)) { 
                    isInShadow = true; 
                    break; 
                } 
            } 
        } 
        if (!isInShadow) 
            pixels[i][j] = object->color * light.brightness; 
        else 
            pixels[i][j] = 0; 
    } 
} 
```

The beauty of ray-tracing, as one can see, is that it takes just a few lines to code; one could certainly write a basic ray-tracer in 200 lines. Unlike other algorithms, such as a scanline renderer, ray-tracing takes very little effort to implement.

This technique was first described by Arthur Appel in 1969 by a paper entitled "Some Techniques for Shading Machine Renderings of Solids". So, if this algorithm is so wonderful why didn't it replace all the other rendering algorithms? The main reason, at the time (and even today to some extent), was speed. As Appel mentions in his paper:

> This method is very time-consuming, usually requiring for useful results, several thousand times as much calculation time as a wireframe drawing. About one-half of this time is devoted to determining the point-to-point correspondence of the projection and the scene.

In other words, it is slow (but as Kajiya - one of the most influential researchers of all computer graphics history -once said: "ray tracing is not slow - computers are"). It is extremely time-consuming to find the intersection between rays and geometry. For decades, the algorithm's speed has been the main drawback of ray-tracing. However, as computers become faster, it is less and less of an issue. Although one thing must still be said: comparatively to other techniques, like the z-buffer algorithm, ray-tracing is still much slower. However, today, with fast computers, we can compute a frame that used to take one hour in a few minutes or less. Real-time and interactive ray-tracers are a hot topic.

To summarize, it is important to remember (again) that the rendering routine can be looked at as two separate processes. One step determines if a point is visible at a particular pixel (the visibility part), and the second shades that point (the shading part). Unfortunately, both the two steps require expensive and time-consuming ray-geometry intersection tests. The algorithm is elegant and powerful but forces us to trade rendering time for accuracy and vice versa. Since Appel published his paper a lot of research has been done to accelerate the ray-object intersection routines. By combining these acceleration schemes with the new technology in computers, it has become easier to use ray-tracing to the point where it has been used in nearly every production rendering software.

