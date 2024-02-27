# Custom Inertia Tensor In Unity

In Unity's default rigid body system, a single body inertia tensor is used to calculate all rotational and translational motion for a object. However, in the domain of space simulation, we might encounter an object that is made of multiple objects e.g., space station with many individual modules or a spaceship with internal fuel tanks. The goal of the project is such that each module or internal component like fuel tank etc can have one single tensor representing it, the tensors can then be combined together using the parallel axis theorem into a singular spacecraft. The position of each internal object or module would dictate how the spacecraft behaves rotationally and translationally. Additionally, an add force method has to apply force to the combined spacecraft inertia tensor like a reaction control system.

# Spacecraft Principal axis 

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/b3b96666-4ef8-4d49-90ec-0f6311911762)

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/a408d52e-4779-42c6-a736-08adc797e6f5)

# Reaction Control

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/0740b0d8-e442-49f4-b039-b022d5a356c1)
\
# The Inertia Tensor

In order to efficiently calculate the resultant rate of rotation, we implemented a tensor of inertia of rank 2. First let's consider the moment of inertia of a simple cube about the cubes center of mass in each of the 3 axis:
![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/6a1beda2-e362-Where:
I is the moment of inertia (kg.m^2)
M is the mass of the cube 
A is the length of cube side

Suppose we want to calculate the composite inertia of an irregular shape like a spacecraft, then we might be able to approximate the number of certain size cubes that we can fit in the spacecraft’s volume and sum up its moments of inertia, to obtain a composite moment of inertia. 

First consider the example on the right, the right most engine pod’s mass is represented by the cube. When the engine pod is attached to the spacecraft, the center of mass for that pod is now the center of mass of the composite spacecraft and the engine pod’s mass is just part of the spacecraft’s total mass. The pod’s mass can be denoted as dm (small mass). 
![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/d69abbe9-934d-47e1-a657-4e9cae1244b0)

For each point mass (box) its angular momentum can be defined by expending the inertia equation and converting it to the equation shown below.

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/1eb66e95-9008-436b-878b-ad596ee103f5)

  
Where:
H_g is the angular momentum about point G or the center of mass
Ω is the rotational rotation or angular velocity
r’ is the distance of the center of that cube to center of mass of spacecraft
dm is the mass of the cube


Next we can rewrite the equation for each of the 3 axes of freedom based on the center of mass of the spacecraft. 

 ![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/ac463df4-5e46-4b4d-aedf-8e6337d376b8)

Where:
x , y, z are components of the distance vector r’. 
I ,j ,k are unit vector components of the direction. The Sum of j + j + k is always = 1 for any rotation.

The moments of inertia with respect to x y and z axis for a cube about the spacecraft's center of mass can be expressed as moments of inertia in Ixx , Iyy and Izz components. 

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/730a644e-9d69-4b2c-90b3-a84ed3c8d36a)


For the moment of inertia between axes like between x and y, we can use the convention Ixy etc..  This accounts for how mass is distributed about about x and y axis simultaneously. 

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/42d3dc1d-2e22-4343-b89d-4a593c5acaac)

Then we can substitute this to the previous equation to obtain Hg where the moments of inertia about the ass axis can be calculated given i j k components of the direction unit vector. H_g= 

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/62184d33-17f8-4bc7-88e3-7656896450be)

We can extract the angular velocity from the above and write it in tensor form.

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/10f7d91f-6866-4e52-ac3c-b874938c0e12)

Where:
Hg.. is the angular momentum
Ixx … is the moment of inertia about each axis and the intermediate axis
Ω is the rotational rotation or angular velocity

For our application in unity, we can simplify it to not account for the intermediate axis and just use the principal axis inertia tensor. This simplification is a good tradeoff between realism but faster development cycle as we can integrate with unity’s rigidbody system.  

# Tensor In Editor 
![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/00dbb180-706e-4648-9c5d-731b00744023)

# Composite center of mass
The first step is to obtain the spacecraft composite center of mass, which depends on how mass is distributed across the spacecraft modules. Each module has its own mass and center of mass. The composite center of mass about each axis can be derived from the equation below :

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/011efc00-ef5c-4cda-9e46-4a0cb11d44e8)

Where:
X Y and Z are distances from a common reference point, this would be geometric center of the command module section (transform.position)
X Y Z Bar is the center of mass of each module about that common reference point
M is the mass.

Using a recursion search we can find out all child modules attached to the command module, and by summing up all centers of mass for each module, we can obtain the composite center of mass as shown below.

# Composite Inertia Tensor

To obtain the composite inertia tensor of the spacecraft, we first need to find the individual inertia tensor for each module. Then the parallel axis theorem can be used to sum up the tensors for a composite tensor. It states that if the inertia tensor is known relative to a center of mass, then the inertia can be calculated about a new abratory point, which could be the center of mass of the player ship instead. This can be achieved using the inertia tensor generalization of the parallel axis theorem.

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/cede7c87-700b-42a5-98ef-a5f7cb3343b4)

Where:
J is the inertia tensor calculated about that new abratory  point
I is the inertia tensor calculated by unity’s rigidbody about the modules own center of mass
R is the displacement or offset vector from the module’s center of mass to the ship's composite center of mass.
E is the identity matrix
X is the outer product
M is the mass of the specific module

In unity, the inertia tensor is represented in the vector form which represents the diagonal inertia matrix, we need to convert the vector form tensor to a 3x3 matrix, however since unity only has 4x4 matrix, we add in a additional column that is equal to zero. m00, m11 and m22 are the diagonal components of the matrix.

Additionally unity’s matrix 4x4 only supports matrix to vector operations, matrix to matrix operations lille addition, multiplication and the inverse of matrix is not supported. Some custom effort is put into adding these operations in order to assist in the computation. 


![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/bc3f604d-a6ae-48ac-ac52-12e54f0de6fd)

Matrix to matrix addition, subtraction, and multiplication.

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/6efd90cf-3287-473f-ade6-12c7d268e92f)

Inverse of a matrix.

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/fb55759c-294f-4438-88e7-cbcae637a599)

Outer product of a vector.

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/b7487cc8-a80f-45b7-a253-5be61f69227a)

Translated Tensor

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/10d0d13d-8c27-4c7e-a7c9-7d6f61aea435)

# Torque

Torque is used to calculate reaction force applied to the spacecraft. 
Using the relationship below:

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/4463ed0e-1b85-42e0-a2bd-3df2e3d414ce)

Where:
Tau is the torque vector x, y, z relative to local axis of the command module
R is the displacement vector relative to the composite center of mass of the ship
force is the negative of the thrust (since the reaction force on the spacecraft is opposite of the thrusting direction)
X is the cross product


To calculate how that torque affects the spaceship in terms of changing its angular using the following formula earlier, where Hg is the angular momentum:

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/adaba12c-c6a0-4e01-9a58-6cdaff729637)

We can replace angular momentum with torque and multiplied by inverse of inertia tensor to obtain angular acceleration, or delta Omega. Delta omega will be the acceleration to angular velocity caused by the application of a force vector, for instance a thruster over time.  

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/3c6781e0-b3e2-4ce7-8009-8b6cd51b13a0)

When the delta omega value in the local playership frame can be then used to calculate which thrusters must fire for each of the rotational axis. The largest component of delta omega means that force is most efficiently translated to that rotational axis . Thruster mapping for each axis can be determined. 


# Translational movement

Translational movement can be easily calculated given that we treat the playership composite as a point mass. Then the translational movement would simply be the sum of all forces using the vector form of F = M.a.

The inertia tensor implements a add force at position function, this is similar to the rigid body add force function that comes pre built into unity’s rigidbody’s system, however this function adds forces based on the composite inertia tensor as well as the composite center of mass. Both rotational acceleration translational acceleration is calculated for each time step or delta time. 

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/d654c024-6d4a-4dde-b655-7ccd1ce49133)

# Handling of position and rotation of gameobject

All motion of gameobject is handled by the inertia tensor script. With the script, word position, world velocity are stored in double precision format and run continuously in each fixed update. When another function calls the add force at the position function, the function updates the angular acceleration and acceleration. 

For each fixed update cycle, multiple add force at position can be called and it simply adds up all the acceleration. In each inertia tensor fixed update, the acceleration is integrated and added to the world position and world velocity. Note that this world position and world velocity is always the floating origin position, where its initial value is zero. The player’s displacement from origin is calculated and applied to all other ship objects in the scene to transform them to the player's frame, allowing the player ship to be always in game origin. The player’s position and velocity in the planetarium is stored in the orbit.cs heliocentric position and heliocentric velocity. 

![image](https://github.com/Jacob19999/unity_inertia_tensor/assets/26366586/b34c8d00-523e-480b-9718-e3b223c7a489)


