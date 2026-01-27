Blue noise vs white noise: in the blue one dots are placed not closer than a specified proximity parameter.

Blue noise is hard to generate from random numbers because as there are more and more points, the lower the chance of a new point hitting a spot where it can fit.
Ideas:
- tile the area
- mark tiles as filled when a dot hits them
- proceed to generate random dots outside the bounds of filled tiles
other idea:
- add points onto a graph
- as a new edge is added, place a dot on that edge randomly accounting for the confines of the proximity parameter around the vertices of the edge

Don Mitchell - paper on the blue noise generation.

Casey's approach:
- pick a point
- create exclusion zone around it
- generate a random point in the donut just outside that exclusion zone
- repeat steps 1-3 a bunch of times
- go to each of the generated points and do the same for them recursively
- exit condition???

Complications:
 - checking the exclusion zone hit must happen on the triangulated surface in the world, not on the rectangle. Ray must be intersected with triangle. There are about probably hundreds of truangles
cross-product - векторное умножение
dot-product - скалярное умножение

Back of the envelope calculations:
- 50 math operations per one ray-triangle intersection check
- 10 000 triangles
- 500 000 math operations per point attempt
- ~ 10 000 points would give us 5B math ops
Comparing it to a back-of-the-envelope calculation of a modern CPU's throughput:
- CPU clock speed 3 Ghz
- assume interactive frame-rate of 30 fps
- 100M CPU cycles per frame
- x8 multiplie-data: modern CPU are single instruction multiple data, so we can assume that each cycle is about 8 times that 100M
- x2 float-multipliers: in one CPU there are two chips that are capable of float-pointer multiplication, so the number is about 1.6B
- x4 cores: and that was just one core, so in theory you can multiply by 4: 6.4B
That's presuming that CPU does not waste cycles doing preparatory work and data feeding. Prolly it's like 10 times slower in actuality(???)

CPUs also overlap operations (skylake arch from intel that does mult in a specific fashion, a newer arch is AMD Zen):
- latency: how much ticks it takes for CPU to come back with an answer
- throughput: throughput accounts for overlap. How much does each additional thing cost after the CPU started executing an operation?
Simple explainer:
- latency - 1 min
- throughput 1 sec
- by stacking 60+ instructions gets us latency of 1 sec per operation (SORT OF)
Skylake example:
- latency of 4
- throughput 0.5
![[simple code high performance 2026-01-08 15.00.55.excalidraw|600]]

Where to look for how much time asm commands take to run on CPUs: https://www.uops.info/

NOW TO GETTING THE DATA THAT WE MULTIPLY

How much data we have? Do all those triangles fit into L1 cache?

| level | capacity | latency in cycles | peak bandwidth | sustained bandwidth |
| ----- | -------- | ----------------- | -------------- | ------------------- |
| L1    | 32 Kb    | 4                 |                | 81 bytes/c          |
| L2    | 256 Kb   | 12                |                | 29 bytes/c          |
| L3    | 2 Mb     | 44                |                |                     |
| Main  | ???      | 100+              |                |                     |
Use sustained bandwidth to assess how much float-point numbers we can feed into the multiplier
Prolly we need L2 with the amount of triangle data: 
- 10k triangles is 3 vertices, so 9 floats.
- that is 90 Kb of data: > 32 kB and < 256 kB
- 10k points is 30kb
- total is 100 kb
- bandwidth is 29 bytes/cycle, so 7 float-point numbers per cycle can be loaded to CPU

So reading from the L2 cache would mean... 100kB / 28 B/c ~ 3000 cycles to feed all the data to the CPU + unknown amount of cycles to load data to L1 cache + 5 B cycles to run all math ops.
so it seems like we CAN actually go under 30 ms?..

What could I possibly expect from the CPU knowing its specs? is the question we're trying to understand in general.

This whole process is condensed in FLOPS (floating point operations?)

see also:
- KDTree - a way to organize the space partition in a tree for more efficient ray-casting
- quad tree
- collision meshes
- SIMD - SSE (4 floats wide), AVX (8 floats wide), AVX-512 (16 floats wide)
	- AVX is what modern CPUs can do
- barycentric coordinates
- n-way blending
- cpp intrinsics - for 8x muls, subs and stuff
- https://en.wikipedia.org/wiki/M%C3%B6ller%E2%80%93Trumbore_intersection_algorithm

see everything that the code has to do in the build you're optimising

the more the code reuse the more the possible ill-fitness of it to the task you're completing

---

Then the code is analysed. And it turns out that a BUNCH of stuff should be done in PREpass.

Final focus is to get the z coordinate of the intersection of the cast ray and the index of the hit triangle.

Here's how we can make sure it's SIMDed using barycentric coordinates.

In each pass
- a point is checked against 8 triangles (see the n-way blending below)
- one of 8 indices updates ONLY if the checked triangle is better(??) than the previously evaluated one with regards to the cast ray
	- instead of `Res = A if A < B else B` the vectorized form is `Res = (m & A) | (~m & B)` <- this is the `blendv`
- after the SIMDed routine is completed the code extracts the best out of the 8 triangles and returns a result

See also:
- intrinsics
- https://godbolt.org/

Do the SIMD before the multi-thread


---

n-way blending

take a triangle and put it into barycentric coordinate (affine transform!!).
That gives you a way of running only 3 tests for whether a point is inside the triangle or outside.

The tests:
- $u >= 0$
- $b >= 0$
- $u + v <= 1$

The transform itself is three steps:
- translation (base-vertex of the triangle becomes placed at zero)
- multiply by matrices twice to make the triangle a right triangle (how are those transforms comprised? well...)