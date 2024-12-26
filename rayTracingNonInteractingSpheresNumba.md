---
layout: default
title: GPU Ray Tracing with Numba for Multiple Non-Interacting Spheres
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

The code demonstrates a simple GPU-based ray tracing implementation using **Numba**. The implementation traces rays from a light source to a collection of spheres. No sphere-sphere interaction is accounted for.

### Key Features

1. **Kernel function** written in Numba, utilizing device functions for: a) Vector normalization; b) Dot product computation; c) Ray-sphere intersection tests.
2. Customizable ray-object intersection logic, making the rendering pipeline adaptable to other geometric primitives.
3. Color computation based on the surface normal at the intersection point.

---

### 1. Kernel Function

The `render` function is the main kernel that performs ray tracing. It works as follows:
- Computes the origin and direction of rays for each pixel in the image.
- Tests for intersections with all spheres in the scene using the `ray_sphere_intersection` device function.
- Calculates the color based on the surface normal at the intersection point.

#### Key Features
- The **ray origin** is located at the center of the scene in the $(x, y)$ plane $(0,0)$, and slightly in front of the camera along the $z$-axis at $z = -1$. The ray direction is computed from the pixel's normalized screen coordinates.
- **Device functions** enable modular computations:
  - `normalize_vector`: Normalizes a vector.
  - `dot_product`: Computes the dot product of two vectors.
  - `ray_sphere_intersection`: Tests whether a ray intersects a sphere and calculates the intersection distance.

---

### 2. Ray-Sphere Intersection Test

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

The color of each pixel is computed based on the surface normal at the intersection point. The surface normal \(\mathbf{N}\) is:
\[
\mathbf{N} = \frac{\mathbf{P} - \mathbf{C}}{R}
\]
where \(\mathbf{P}\) is the intersection point.

The computed normal is used to derive the color using the following mapping:
\[
\text{Color} = 0.5 \cdot (\mathbf{N} + 1.0)
\]
This maps the normal components from the range \([-1, 1]\) to \([0, 1]\) for RGB representation.

---

## 4. Customizability

The `ray_sphere_intersection` function can be replaced with other geometric intersection tests, such as ray-triangle intersections, to render scenes with arbitrary geometries.

### Example: Ray-Triangle Intersection
A similar quadratic solution can be derived for triangle meshes, enabling more complex object rendering.

---

## 5. Running the Code

### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/<username>/<repository-name>.git
   cd <repository-name>
   ```
2. Install dependencies:
   ```bash
   pip install numba numpy matplotlib
   ```

3. Run the script:
   ```bash
   python ray_tracing.py
   ```

### Output
The rendered image will display spheres with colors mapped from their normals.

---

## 6. Future Enhancements

1. Add lighting models (Phong, Lambertian) for realistic shading.
2. Implement additional geometric primitives (e.g., triangles, planes).
3. Optimize performance for larger scenes using advanced GPU techniques.

Feel free to modify the code to experiment with new rendering techniques and geometric primitives!

