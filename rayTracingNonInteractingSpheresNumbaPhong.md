---
layout: default
title: Ray Tracing with Numba and the Phong Illumination Model
---

<script type="text/javascript">
MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']],
    displayMath: [['$$', '$$'], ['\\[', '\\]']],
  }
};
</script>
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/tex-mml-chtml.js">
</script>

The code demonstrates a simple ray-tracing implementation using Numba to render non-interacting spheres under the Phong illumination model. The code is GPU-accelerated using Numba's CUDA library to achieve efficient rendering.

### Overview

The ray-tracing algorithm renders spheres based on the Phong illumination model, characterized by ambient, diffuse, and specular components. The implementation allows an arbitrary number of spheres, each defined by:

- center position $$(x, y, z)$$;
- radius $$r$$;
- a single point light source illuminating the scene with $$(R, G, B)$$ color intensity $$\underline{I}_i=(I_{i_R}, I_{i_G}, I_{i_B})$$;
- ambient light color intensity represented as an RGB vector $$\underline{I}_a=(I_{a_R}, I_{a_G}, I_{a_B})$$;
- a vector reflection coefficient $$\underline{k}_d=(k_{d_R}, k_{d_G}, k_{d_B})$$ modelling diffuse reflection at the frequencies of each color (R, G or B), with $$0\leq k_{d_G} \leq 1$$, $$i=R, G, B$$ and determines how much light the object's surface reflects ($$0$$ (no reflection) to $$1$$ (full reflection));;
- a vector reflection coefficient $$\underline{k}_s=(k_{s_R}, k_{s_G}, k_{s_B})$$ modelling specular reflection at the frequencies of each color (R, G or B), with $$0\leq k_{s_G} \leq 1$$, $$i=R, G, B$$ and determines how much light the object's surface reflects ($$0$$ (no reflection) to $$1$$ (full reflection));;
- a vector reflection coefficient $$\underline{k}_a=(k_{a_R}, k_{a_G}, k_{a_B})$$ modelling ambient reflection at the frequencies of each color (R, G or B), with $$0\leq k_{a_G} \leq 1$$, $$i=R, G, B$$ and determines how much ambient light the object's surface reflects ($$0$$ (no reflection) to $$1$$ (full reflection));
- shininess $$\alpha$$, a scalar value characterizing the surface roughness, the value of which generally falls between $$10$$ and $$50$$;
- non-interacting spheres, meaning no mutual reflections or shadowing.

---

### The Phong Illumination Model

#### Ambient Lighting
Ambient light simulates light bouncing in the scene between non-rendered and rendered objects, so that parts of the object not directly exposed to the light source do not stay black. In other words, the ambient component simulates an indirect light scattered in all directions by the environment of average color $$\underline{I}_a$$ uniformly illuminating the object. It is modeled as:

$$\underline{k}_a \odot \underline{I}_a $$

#### Diffuse Lighting (Lambertian diffusion)
The diffuse component accounts for light scattered uniformly in all directions from a surface. It depends on the angle between the surface normal and the light direction and is independent from the viewing point since light is diffused uniformly

$$\max((\hat{n}\cdot \hat{l}),0) \underline{k}_d \odot \underline{I}_i $$

where $$\hat{n}$$ is the normal at hit point and $$\hat{l}$$ is the unit vector from the hit point to the point light source. The $$(\hat{n}\cdot \hat{l})$$ scalar product represents the projection of a unit-area portion of the impinging planar wavefront over the objecs' surface modelled as the tangent plane at the hit point. It should be noticed that, if $$(\hat{n}\cdot \hat{l})>0$$, the light is striking the surface at an angle less than $$90^\circ$$, contributing positively to the illumination. If, on the other side, $$(\hat{n}\cdot \hat{l})\leq 0$$, the light is coming from behind or parallel to the surface, which should not contribute to the visible illumination, namely, the light source should be shadowed. By taking $$\max((\hat{n}\cdot \hat{l}),0)$$, we ensure that only light falling on the visible side of the surface contributes to the diffuse component.

#### Specular Lighting
The specular component models the mirror-like reflection of light and is determined by the angle between the view direction and the specular reflection direction. It is modelled as

$$\max((\hat{r}\cdot \hat{v})^\alpha,0) \underline{k}_s \odot \underline{I}_i $$

where $$\hat{v}$$ is the view direction from the hit point to the observation point and $$\hat{r}$$ is the specular reflection direction. The specular reflection direction can be computed as

$$\hat{r}=2(\hat{n} \cdot \hat{l})\hat{n}-\hat{l}$$

It should be noticed that, if  $$(\hat{r}\cdot \hat{v}) > 0$$, the reflected light is visible to the viewer, contributing to the specular highlight. If, on the other side, $$(\hat{r}\cdot \hat{v})\leq 0$$, the reflection is directed away from the viewer, so it does not contribute to the highlight. Using $$\max((\hat{r}\cdot \hat{v})^\alpha,0)$$ ensures that specular highlights are computed only for visible reflections.



### Bibliography

J. Gomes, L. Velho, M.C. Sousa, "Computer Graphics: Theory and Practice", CRC Press, 2012.

F. Ganovelli, M. Corsini, S. Pattanaik, M. Di Benedetto, "Introduction to Computer Graphycs: A Practical Learning Approach", CRC Press, 2015.
