# A-Prism-Net-Search-Space-Representation-for-Multi-Objective-Building-Spatial-Design

Support materials for paper submission on EMO 2023 conference

This repository containts main code "Prism Net representation.ipynb" written in Python 3 and text files with final results based on which the figures in the paper were obtained. 

The code:

The code starts with section of initialization. "poly_init" is a variable that containts initial population for optimization. It has 10 diverse building designs set up manually, each of them has 3 spaces located on 2 levels. Also the algorithm requires a few more predefined values: 3 volumes of spaces, which real volumes of spaces will strive for in the optimization process, parameters of the box V (defined in section assumption in the paper as x_V, y_V and z_V) inside which a building should be located, minimal allowed angle of spaces needed for Constraint 7, and height of each of 2 levels. Also this section contains the function "full_height" which bassically just calculates the sum of heights of all levels, and random seed so that the results are reproducible.

Next section is called "Useful functions". Functions needed for constraints and mutations are included in this section. Most of them are used for more than one constraint, that's why we decided to put them separately. These functions are described in the code either by comments or by title. 

Section "Constraints" contains functions "C0" - "C8". They correspond with constraints described in the paper, but let us look closeer on each of them. Also some additional functions are included in this section to simplify code review. 

Constraint 0 (C0: No duplicates and degenerate cells, all levels and spaces are present without passes): 
Due to the fact that this representation serves to describe potential building designs, we propose to immediately eliminate possible errors of optimization algorithms by prohibiting duplicate cells and cells having a degenerate projection. Also we would like to have all spaces and levels without passes. We consider two cells as duplicates if they have the same set of vertices and they are located on the same level. Degenerate triangles are triangles in which three vertices lie on the same line. To check whether vertices of cell belong to the same line we suggest to compare the area of the triangle with zero to avoid potential division on zero. 

Constraint 1 ("C1: No overlaping (intersection of 2 cells can be either a point or a whole edge)" in code or "Constraint 1: Non Overlap" in paper):
Two cells on the same level should not overlap each other. It means that two triangles can intersect each other only in two cases: their intersection is a common vertex, or it is a common side of both triangles. To achieve it we need to check two conditions.\
The first condition states that on the projection each vertex of any triangle should not lie in the interior of any other triangle on the same level. To check whether this condition is met for a vertex (x_0, y_0) and a triangle with vertices (x_1, y_1), (x_2, y_2), (x_3, y_3) it is useful to calculate three auxiliary values t_1, t_2 and t_3:\
t1 = (x_1-x_0) * (y_2-y_1)-(x_2-x_1) * (y_1-y_0),\
t2 = (x_2-x_0) * (y_3-y_2)-(x_3-x_2) * (y_2-y_0),\
t3 = (x_3-x_0) * (y_1-y_3)-(x_1-x_3) * (y_3-y_0).\
If t_1, t_2 and t_3 have the same sign, then the vertex (x_0, y_0) is in the interior of the triangle, so the constraint is violated. If at least on of these three values is equal to 0 and both others have the same sign, then the vertex is on the side, so we need to check additionally if it coincides with any of the triangle vertices. If it does not coincide, then the constraint is violated. In all other cases the condition is met.\
But the first condition is not a sufficient condition for the constraint to hold, for instance, if two trianges have 2 coinciding vertices and third vertices are on the same side, but not inside each other, there is a violation of constraint we want to achieve but at the same time it does not violate the first condition. Hence, we need an additional test. It states that sides of the triangles located on the same level should not intersect each other in their interior. 

Constraint 2 ("C2: Complete coverage" in code or "Constraint 2: Complete coverage" in paper):
Text in paper states that "Every point in the volume V is covered by at least one cell", but actually it is an implication of the thing which is checked in the code. We introduce the constraint verifying that all cells (active and not active) cover the whole predefined volume V and do not go beyond its borders. To check it we need to test two conditions. The first one states that on every level searching through all vertices of cells minimum x-coordinate and minimum y-coordinate should be equal to $0$ and maximum x-coordinate and maximum y-coordinate should be equal to x_V and y_V correspondingly. The second condition checks the complete coverage of the building with cells. Each side of any triangle that does not lie on the outer contour of the building should be the part of exactly two triangles. And each side of any triangle that lies on the outer contour of the building should be the part of exactly one triangle.

Constraint 3 ("C3: Spaces should be connected" in code or "Constraint 3: Connectivity of spaces" in paper):
The first case (for cells located on the same level) is described in paper. To consider the second case, we need to introduce the notion of external boundaries of a space. Each space consists of several cells. External boundaries of a space are the set of sides of triangles making up this space on specific level, sides which either belong to the outer contour of the whole building or which are also part of some other space. The important detail in this notion is that the "chain" of sides continuing each other is considered as one external boundary. For understanding the second case, it is very important not to confuse external boundaries with sides and edges. By "sides" we mean sides of triangle cells (in 2-dimensional projection) or segments connecting vertices in 3-dimensional figures, such as parallelepiped, and by "edges" we mean a pair of nodes in a graph. The second case of the constraint might come up when there are cells with the same s-value located on several levels. We allow situations like this in our representation, but only if the space has the same projection on every level, i.e. if we consider parts of the space belonging to the same level, or "layers" of the space, then all layers should have the same external boundaries in 2-dimensional projection. Another necessary condition is that there should not be a floor between such layers of space on which this space is not represented. 

Constraint 4 ("С4: Cells which have the same non-zero color value should form a convex space" in code or "Constraint 4: Convex polygon" in paper):
For each space we sort its vertices clockwise and use function "is_convex_polygon" from https://stackoverflow.com/questions/471962/how-do-i-efficiently-determine-if-a-polygon-is-convex-non-convex-or-complex

Constraint 5 ("C5: Spaces should be connected to the ground" in code or "Constraint 5: Spaces  should be connected to the ground" in paper):
Here we use library Geometry3D https://pypi.org/project/Geometry3D/ to construct polyhedron out of every space and then we check whether spaces intersect each other using function "intersection" from the same library. This is how we construct the dual graph described in the paper and then we check if all vertices are connected to the marked vertices (corresponding to the spaces on the ground level) using library https://networkx.org/. 

Constraint 6 ("C6: No cavities" in code or "Constraint 6: No cavities" in paper):
Here we obtain the set of cell with zero s-value having the side on the border of V and we mark them. Then we check if every zero-cell is connected to the marked cells through sides of cells. 

Constraint 7 ("C7: No sharp angles in spaces (user inputs an angle, optional)" in code or "Constraint 7: Non sharp angles (optional)" in paper):
It calculates all angles in each space using cosine theorem and checks if it is bigger than predefined volume. 

Constraint 8 ("C8: Maximum number of triangles (technical)" in code):
This constraint is needed to limit the number of appearing cells during optimization. Experimentally it was obtained that it is enough to limit it with number of cells in the initial design multiplied by 1.5. This coefficient can be changed. 

Next sections are "Mutation operator" and "Objectives". Objectives and all types of mutation are described in paper in detail and code contatins many comments to understand it better. 

The optimization part contains useful functions for obtainig the results in a proper way. Text files in this repository "res099.txt", "res03.txt" and "resdiff.txt" are all 60 populations for each of 30 runs for NSGA-II algorithm with mutation rate 0.99, 0.33 and "local search" correspondingly. Similarly, the "ressms099.txt", "ressms03.txt" and "ressmsdiff.txt" files relate to the algorithm SMS-EMOA. Based on this files the result figures in paper were obtained. 





