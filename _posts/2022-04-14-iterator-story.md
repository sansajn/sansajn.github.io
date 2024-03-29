---
layout: post
title: "Implementing custom iterator (the story behind)"
date: 2022-04-14 8:00:00 +0100
tags: [c++, iterator, series, test]
comments: true
author: Adam Hlavatovic
---

# Implementing custom iterator (the story behind)

Guide to implement *random access* iterator working with parallel `std::transform` algorithm to support parallelization for pixel algorithms (such as raytracer) ...

I've recently go through great raytracing related book where "raytracing" loop was implemented more or less this way

```c++
RGBColor	L;

Ray ray;
ray.o = eye;

int n = sqrt(num_samples);

for (int r = 0; r < vres; r++) {			// up
	for (int c = 0; c < hres; c++) {		// across
		L = black;
		
		for (int p = 0; p < n; p++)		// up pixel
			for (int q = 0; q < n; q++) {	// across pixel
				Point2D sp = sampler->sample_unit_square();
				pp.x = c - 0.5 * hres + sp.x;
				pp.y = r - 0.5 * vres + sp.y;
				ray.d = get_direction(pp);
				L += tracer->trace_ray(ray, depth);
			}
										
		L /= num_samples;
		display_pixel(r, c, L);
	}
}
```

It is simple, we calculate light for each pixel in output image by casting ray in "right" direction.

The book was written in year 2006 so the raytracer is written in *C++98* without any parallelization support. Because calculating light for each pixel is independend to other pixels around raytracing loop is ideal candidate for parallelization. 

In *C++17* we can implement parallel raytracing with replacing our for based loop with parallel `std::transform` algorithm. We "only" need iterator allow as to iterate through each pixel and function calculate light for that pixel. Our new raytracing loop implementation now become

```c++
auto pixel_rng = pixel_pos_view(vp.hres, vp.vres);
transform(std::execution::par, begin(pixel_rng), end(pixel_rng), begin(pixels),
	[&](pair<size_t, size_t> pixel_pos){  // (c,r)
		RGBColor L{black};
		Ray ray;
		ray.o = eye;

		auto & [c, r] = pixel_pos;
		for (int p = 0; p < n; ++p) {  // up pixel
			for (int q = 0; q < n; ++q) {  // across pixel
				Point2D sp = sampler->sample_unit_square();
				Point2D pp;  // sample point on a pixel
				pp.x = c - 0.5 * hres + sp.x;
				pp.y = r - 0.5 * vres + sp.y;
				ray.d = get_direction(pp);
				L += tracer->trace_ray(ray, 0);
			}
		}

		L /= num_samples;

		return map_to_pixel_color(L);
	});
```

> **note**: some weeks after the implementation I've saw Matt Godbolt in his [*Path Tracing Three Ways: A Study of C++ Style*](https://www.youtube.com/watch?app=desktop&v=HG6c4Kwbv4I) CppCon talk where he described `cartesian_product` view serving for similar purpose ...

where dereferencing our new pixel position iterator returns `pair<size_t, size_t>` representing pixel position as *(column, row)* pair. The lambda function takes pixel position pair and calculate light for pixel on that position.

This post series will describe how to implement pixel position iterator capable parallelizing pixel algorithms like raytracer described in the book.

The first post is this *the story behind* post, then I will describe *input*, *forward*, *bidirectional* and *random access* iterator implementation each in separate post and raytracer engine integration at the end.

For now see [github](https://github.com/sansajn/test/tree/master/c%2B%2B/iterator) repository for the current iterator implementation.
