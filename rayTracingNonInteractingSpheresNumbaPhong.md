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
- a single point light source illuminating the scene with $$(R, G, B)$$ color intensity $$\underline{I}_i=(I_{i_r}, I_{i_g}, I_{i_b})$$;
- ambient light color intensity represented as an RGB vector $$\underline{I}_a=(I_{a_r}, I_{a_g}, I_{a_b})$$;
- a reflection coefficient $$0\leq k_d\leq 1$$ modelling diffuse reflection;
- a reflection coefficient $$0\leq k_s\leq 1$$ modelling specular reflection;
- a reflection coefficient $$0\leq k_a\leq 1$$ modelling ambient reflection and determines how much ambient light the object's surface reflects ($$0$$ (no reflection) to $$1$$ (full reflection));
- shininess $$\alpha$$, a scalar value characterizing the surface roughness, the value of which generally falls between $$10$$ and $$50$$;
- non-interacting spheres, meaning no mutual reflections or shadowing.

---

### The Phong Illumination Model

#### Ambient Lighting
Ambient light simulates light bouncing in the scene between non-rendered and rendered objects, so that parts of the object not directly exposed to the light source do not stay black. In other words, the ambient component simulates an indirect light scattered in all directions by the environment of average color $$\underline{I}_a$$ uniformly illuminating the object. It is modeled as:

$$k_a \underline{I}_a $$

#### Diffuse Lighting (Lambertian diffusion)
The diffuse component accounts for light scattered uniformly in all directions from a surface. It depends on the angle between the surface normal and the light direction and is independent from the viewing point since light is diffused uniformly

$$k_d (\hat{n}\cdot \hat{l}) \underline{I}_i $$

where $$\hat{n}$$ is the normal at hit point and $$\hat{l}$$ is the unit vector from the hit point to the point light source.

#### Specular Lighting
The specular component models the mirror-like reflection of light and is determined by the angle between the view direction  and the reflection direction :

where:

 is the specular reflection coefficient.

 is the shininess coefficient controlling the sharpness of the highlight.

 is the specular light intensity.

The reflection direction  is computed as:

#### Final Illumination Model
The final color of a pixel is a combination of the three components:

Each component is clamped to the range  to ensure valid RGB values.

---

### Implementation Highlights

GPU Acceleration: The code uses Numba's CUDA support to accelerate computations.

Flexibility: The number of spheres can be adjusted easily by modifying the input data.

Scene Parameters:

A single light source with a fixed position.

Spheres are non-interacting; there are no shadows or reflections.


The ray-sphere intersection is derived from the parametric equation of a ray and the implicit equation of a sphere:

#### Mathematical Derivation

1. **Ray Equation**:
   
   $$\mathbf{R}(t) = \mathbf{O} + t \cdot \mathbf{D}$$
   
   where:
   - $\mathbf{R}(t)$: A point on the ray at parameter $t$,
   - $\mathbf{O}$: Ray origin,
   - $\mathbf{D}$: Ray direction (normalized vector),
   - $t$: Parameter along the ray.

3. **Sphere Equation**:

   $$\|\mathbf{P} - \mathbf{C}\|^2 = R^2$$
   
   where:
   - $\mathbf{P}$: A point on the sphere,
   - $\mathbf{C}$: Center of the sphere,
   - $R$: Radius of the sphere.

5. **Intersection**:
   Substituting the ray equation into the sphere equation:

   $$\|\mathbf{O} + t \cdot \mathbf{D} - \mathbf{C}\|^2 = R^2$$
   
   Expanding and simplifying:
   
   $$t^2 \cdot (\mathbf{D} \cdot \mathbf{D}) + 2t \cdot (\mathbf{D} \cdot (\mathbf{O} - \mathbf{C})) + \| \mathbf{O} - \mathbf{C} \|^2 - R^2 = 0$$

   This is a quadratic equation:
   
   $$at^2 + bt + c = 0$$
   
   where:

   $$a = \mathbf{D} \cdot \mathbf{D}, \quad b = 2 \cdot (\mathbf{D} \cdot (\mathbf{O} - \mathbf{C})), \quad c = \| \mathbf{O} - \mathbf{C} \|^2 - R^2$$

   The discriminant determines intersection:

   $$\Delta = b^2 - 4ac$$

   If $\Delta > 0$, the ray intersects the sphere, and the smallest $t > 0$ gives the nearest intersection point:

   $$t = \frac{-b - \sqrt{\Delta}}{2a}$$

---

### 3. Color Computation

The color of each pixel is computed based on the surface normal at the intersection point. The surface normal $\mathbf{N}$ is:

$$\mathbf{N} = \frac{\mathbf{P} - \mathbf{C}}{R}$$

where $\mathbf{P}$ is the intersection point.

The computed normal is used to derive the color using the following mapping:

$$\text{Color} = 0.5 \cdot (\mathbf{N} + 1.0)$$

This maps the normal components from the range $[-1, 1]$ to $[0, 1]$ for RGB representation.

---

### 4. Customizability

The `ray_sphere_intersection` function can be replaced with other geometric intersection tests, such as ray-triangle intersections, to render scenes with arbitrary geometries.

#### Example: Ray-Triangle Intersection
A similar quadratic solution can be derived for triangle meshes, enabling more complex object rendering.


