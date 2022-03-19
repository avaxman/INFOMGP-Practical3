# Practical 3: Finite-Element Soft-Body Deformation

## Handout date: 20/Mar/2022

## Deadline: 5/Apr/2022 08:59AM


The third practical is about simulating soft bodies with the finite-element method learned in class, where the material is covered in Lecture 9 and its associated lecture notes (about soft-body simulation). The system will handle basic collision constraints for you, and you will not be tested on them in the basic version.


The objective of the practical are:

1. Implement the full linear finite-element dynamic equation integration, to create soft-body internal forces on a tetrahedral mesh.
</br>
2. Extend the framework with some chosen effects.  

This is the repository for the skeleton on which you will build your practical. Using CMake allows you to work and submit your code in all platforms. This is essentially the same system as in the previous practical, in terms of compilation and setup.

## Background

The practical runs in the same time loop (that can also be run step-by-step) as the previous practical. The objects are not limited to convex ones; every triangle mesh can be given as input. The software automatically converts a surface OFF mesh into a tetrahedral mesh with [Tetgen](http://wias-berlin.de/software/index.jsp?id=TetGen&lang=1). A tetrahedral mesh can be provided by the scene files with the .MESH format. There are no ambient forces but gravity in the basic version.

The first considerable difference in the representation of the mesh from the previous practicals is that ```origPositions``` replaces ```origV``` as a big vector in the ordering of $xyzxyz$ (3 number per vertex, total size $3\left| V \right|$), instead of a matrix where vertex coordinates are on rows. This is because you will need this form for the FEM formulation. The rest of the code, including constraints resolution, is adapted to this. Moreover, the scene keeps a huge vector of all stacked coordinate vectors of all meshes, to make the constraint resolution a flat process that does not distinguish meshes---but you do not necessarily have to touch this in the practical, so it's FYI.

The second considerable difference is that every vertex now has its own volume and mass, and reacts in the world as its own independent object, for the purpose of collisions and constraints. The relationship between the different vertices of the same mesh is generally (unless constrained) done by the forces enacted on them by the FEM system. Thus, there is no more handling of center-of-mass nor inertia tensor. As an example, in a collision only the colliding vertices move by impulses; the deformation this causes will in turn generate internal forces (computed by the FEM system) that will move the entire object. As linear FE is far from perfect, this should cause considerable visual scaling artifacts that you can witness in the demo.

The objects move in time steps by altering each vertex of the tet mesh, where the deformation is computed by solving the finite-element equation for movement as learnt in class (*Lecture 9*). For this, you will set up the mass $M$, damping $D$, and stiffness $K$ matrices at the beginning of time, and only again whenever the time step-size changes. Moreover, you will set up the Cholesky solver that factorizes the left-hand side $A$ of the entire system *once* in every *change* of time step (so it might be only once in the beginning of time unless you are keen to play with $\Delta t$).

Note that every vertex has one velocity; this practical will not explicitly model rigid-body behavior with angular velocity in the basic setting.

##The Time Loop

The algorithm will perform the following within each time loop:

1. Integrate velocities and positions from external and internal forces (the finite-element stage).
</br>
2. Collect collision and other constraints if active.
</br>
3. Resolve velocities and then positions to valid ones according to the constraints.
</br>

## Finite-Element Integration

Finite-element integration consists of two main steps that you will implement:

### Matrix constrution

At the beginning of time, or any change to the time step, you have to construct the basic matrices $M$, $K$ and $D$, and consequently the left-hand side matrix $A=M+\Delta t D+\Delta t^2 K$. Those all have to be sparse matrices (of the data type ``Eigen::SparseMatrix<double>``). You have to fill them in using COO triplet format with `Eigen::Triplet<double>` as learnt in class. Then, you can see the Cholesky decomposition code preparing the solver. This is done in the function `createGlobalMatrices()`.

### Integration

At each time step, you have to create the right-hand side from current positions and velocities, and solve the system to get the new velocities (and consequently positions). This is in the usual function `integrateVelocities()`, calling the `solve()` function of the Cholesky solver. As this function only changes right-hand side, it should be quite cheap in time. Note: *without any* refactorization in each time step! 

The scene file contains the necessary material parameters: Young's Modulus, Poisson's ratio, and mass density. You need to produce the proper masses and stiffness tensors from them. Damping parameters $\alpha$ and $\beta$ are given as inputs to the function, and hardcoded as in `main.cpp`.


### Extensions

The above will earn you $70\%$ of the grade. To get a full $100$, you must choose 1 of these 3 extension options, and augment the practical. Some will require minor adaptations to the GUI or the function structure which are easy to do. 


1. Other than FEM, constrain the boundary edges of the mesh (the edges on the faces) to have flexiblity bounds as constraints, so they do not stretch or compress more than these bounds. **Level: easy**.
 </br>
 2. Allow to apply random user-prescribed deformations of objects. For instance, instantaenous "squeezing" of an object to see it deform back after a time. **Level: easy-intermediate**. 
 </br>
3. Implement the corotational elements method as learnt in class. That would require to use a solving method that does not rely on matrix factorization, like conjugate gradients. This is available through Eigen, but you'll have to figure out the details and proper initialization. **Level: intermediate-hard**.
</br>
4. Tweak the visual artifacts of colliding only the touching vertices by sending impulses to the entire body, but weaker than the ones the colliding vertex gets. For instance, use some inverse distance weighting to send velocity impulses to the other vertices. **Level: easy**.
 </br>

You may invent your own extension as substitute to **one** in the list above, but it needs approval on the Lecturer's behalf **beforehand**.


## Installation

The installation follows the exact path as the previous practical. It is repeated for completeness.

The skeleton uses the following dependencies: [libigl](http://libigl.github.io/libigl/), and consequently [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page), for the representation and viewing of geometry, and [libccd](https://github.com/danfis/libccd) for collision detection (now between individual terahedra). Again, everything is bundled as either submodules, or just incorporated code within the environment, and you do not have to take care of any installation details. To get the library, use:

```bash
git clone --recursive https://github.com/avaxman/INFOMGP-Practical3.git
```

to compile the environment, go into the `practical3` folder and enter in a terminal (macOS/Linux):

```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ../
make
```

In windows, you need to use [cmake-gui](https://cmake.org/runningcmake/). Pressing twice ``configure`` and then ``generate`` will generate a Visual Studio solution in which you can work. The active soution should be ``practical3_bin``. *Note*: it only seems to work in 64-bit mode. 32-bit mode might give alignment errors.

## Working with the repository

All the code you need to update is in the ``practical3`` folder. Please do not attempt to commit any changes to the repository. <span style="color:red">You may ONLY fork the repository for your convenience and work on it if you can somehow make the forked repository PRIVATE afterwards</span>. Solutions to the practical which are published online in any public manner will **disqualify** the students! submission will be done in the "classical" department style of submission servers, published separately.


## The coding environment and GUI

The GUI is very similar to the one you used in the previous practicals.

Most of the action, and where you will have to write code, happens in `scene.h`. The functions in `scene.h` implement the time loop and the algorithmic steps, while `constraint.h` implements the `Constraint` class that evaluates and constraints and resolves them. 

The code you have to complete is always marked as:

```cpp
/***************
TODO
***************/
```

For the most part, the description of the functions will tell you what exactly you need to put in, according to what you need to implement as described above.

###Input

The input is the same as in previous practicals; you will notice that there are no user constraints in the given data, since we focus on the FEM part. Use `emptyconstraints.txt` in case of these scenes.

The program is loaded by giving two TXT files as arguments to the executable (three arguments in total; the first being the data folder again): one describing the scene, and another the user-prescribed distance constraints. It is similar to the format in the first practical, but with extensions. That means that there is no backward compatibility; you will have to update files you already made (a bit).

The format of the scene file is:

```
#num_objects
object1.off/mesh     density1   young1  poisson1     is_fixed1    COM1     o1
object2.off/mesh     density2   young2  poisson2     is_fixed2    COM2     o2
...
```

Where:

1. ``objectX.off/mesh`` - either an OFF file in the same manner as the first practical, or a tetahedral MESH file. OFF files will be automatically tetrahedralized upon loading, so you don't have to spend the effort creating tetrahedral meshes offline.
</br>
2. ``density`` - the uniform density of the object. 
</br>
2. ``Young`` - Young's modulus, used to create the stiffness tensor. Use very big values; look up values for general materials, which are usually expressed in Mega or Giga Pascals. The value you give here will be interpreted as Pascal.
</br>
3. ``poisson`` - Poisson's ratio, which describes the incompressibility of the object. Values should be between $[0,0.4999...]$---note that exact 0.5 will produce infinite stiffness that will kill your simulation by dividing by zero, so avoid it.
</br>
4. ``is_fixed`` - if the object should be immobile (fixed in space) or not.
</br>
5. ``COM`` - the initial position in space where the object would be translated to. That means, where the COM is at time $t=0$.
</br>
6. ``o`` - the initial orientation of the object, expressed as a quaternion that rotates the geometry to $o*object*inv(o)$ at time $t=0$.


The constraints file has the following format:

```
#num_constraints

m1 v1 m2 v2

```

Where it creates a distance constraints between vertex ``v1`` on mesh ``m1`` to vertex ``v2`` on mesh ``m2`` in their initial configuration (after COM+orientation transformations). For your convenience, there is an ``emptyconstraints.txt`` file, if you do not want to use constraints.


###Indexing

We use two indexing methods for vertices and coordinates:

1. **Local indexing**: the vertices in each mesh are kept in one big column vector of $3\left| V \right|$ coordinates in $x_1y_1z_1x_2y_2z_2x_3y_3z_3...$ format (dominant order by vertices and subdominant order by coordinate). In general, corresponding quantities like velocities, forces, and impulses will be encoded the same way. Quantities that are the same for all $xyz$ coordinates of a single vertex---like mass, Voronoi volumes, etc.---are encoded in a vector of $\left| V \right|$ with one quantity per vertex. The comments in the code will tell you which is which.
</br>
2. **Global indexing**: this is the indexing used by the scene class, which aggregates all vertex-coordinates in the scene. The index is a flat single column vector of **all** coordinates in the scene in the same format as the local indexing. As such, the individual coordinates of every mesh are a continuous segment within this global vector; the ``globalOffset`` variable in each mesh indicates where this segment begins. The same goes again for impulses and velocities---and inverse masses in this case! (unlike the mesh class which keeps one inverse mass quantity per vertex). That is because the scene class handles the constraints resolution, so this encoding is much more comfortable; the mass fitting for each coordinate. At any rate, check the comments attached to the variable definitions.

Two functions: ``global2mesh()`` and ``mesh2global()`` update the values back and forth between the scene and the individual meshes.


## Submission

The relevant part of code of the practical has to be submitted in the appropriate assignment slot on Blackboard.

The practical must be done **in pairs**. Doing it alone requires a-priori permission. Any other combination (more than 2 people, or some fractional number) is not allowed. 

The practical will be checked during the lecture time of the deadline date. Every pair will have 10 minutes to shortly present their practical, and be tested by the lecturer with some fresh scene files. In addition, the lecturer will ask every person a short question that should be easy to answer if this person was fully involved in the exercise. We might need more than the allocated two hours for the lecture time; you will very soon receive a notification. The registration for time slots will appear soon.

<span style="color:red">**Very important:**</span> come to the submission with a compiled and working exercise on your own computers. If you cannot do so, tell the lecturer **in advance**. Do not put the command-line arguments in code, so that we waste time on compilation with every scene (is there any logic in doing so? it was common for some reason in the submissions).

## Frequently Asked Questions

Here are detailed answers to common questions. Please read through whenever you have a problem, since in most cases someone else would have had it as well.



# Success!











