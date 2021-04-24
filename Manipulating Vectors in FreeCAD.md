## Manipulating Vectors in FreeCAD

Vectors are ubiquitous in FreeCAD, describing point locations and displacements.

```
from FreeCAD import Vector
v1 = Vector(1, 2, 3)
v2 = Vector(4, 5, 6)
```

creates a vector *v1* whose x-, y-, and z-components are 1, 2, 3 , respectively.

The result of adding the two vectors, tip to tail is given by

```
v1.add(v2)
```

but much more conveniently by:

```
v1 + V2
```

because the + operator has been overloaded, resulting the vector `Vector(5, 7, 9)`

We can also subtract vectors, and multiply or divide them by scalars, so we can write:

```
2*v2 - v1/2
```

obtaining `Vector(7.5, 9, 10.5)`

The length of the vector v1 is given by Pythogoras' theorem (in 3D):

```
import math
math.sqrt(v1.x * v1.x + v1.y * v1.y + v1.y * v1.y)
```

or much more conveniently by the builtin method

```
v1.Length
```

We can use a vector to represent a direction in space. For this purpose, since the (non-zero!) length of the vector is then immaterial, it is customary to use unit vectors, whose length is 1.  (In FreeCAD dialogs for axes, you can use un-normalized directions. The code normalizes it for you.)  We can create a unit vector by normalizing any vector in the desired direction:

```
v1.normalize() # unit vector in the direction of v1
```

This gives us alternative way of creating vectors. If we'd like, for instance,  to create a vector in the direction of *v1* with the length of *v2*, we could use:

```
v2.Length * v1.normalize()
```

### Dot and cross products of vectors

Other than addition and subtraction, there are other geometrically meaningful ways of combining two vectors.

One is the dot product [](https://en.wikipedia.org/wiki/Dot_product)

```
v1.dot(v2)  # or v2.dot(v1)
```

This can be shown to be equal to the product of their two lengths with the cosine of the angle between them.  It is thus, in some sense, the projection of one vector on the other. It can be used to calculate the angle between the two (non-zero) vectors:

```
angle = math.acos(v1.dot(v2)/(v1.Length * v2.Length))
```

giving the angle in radians.

this method is built in to FreeCAD

```
angle = v1.getAngle(v2)
```

The *cross product* of two vectors `v1` and `v2` creates a third vector, perpendicular to both of them, that is, normal to the plane containing *v1* and *v2* https://en.wikipedia.org/wiki/Cross_product

Its length is given by the product of the lengths of `v1` and `v2` with the sine of the angle between them. It thus vanishes if `v1` and `v2` are parallel.

Another way of stating this is that the length of `v1.cross(v2)` is the area of the parallelogram defined by `v1` and `v2`. Note that `v1.cross(v2)`and `v2.cross(v1)` have opposite signs, but `v1.dot(v2)` and `v2.dot(v1)`are equal.

A test to check if, within numerical error, two (non-zero) vectors are parallel could be written:

```
def isParallel(v1, v2):
    return (v1.cross(v2)).Length <= 1e-7 * v1.Length *  v2.Length
```

### Rotations

Another operation we might want to perform on a vector is to rotate it. There are many ways to specify a rotation.

-- Rotation object-- four floats (a quaternion)

 FreeCAD's internal representation of rotations (a, b, c, d)  = a **i** + b **j** + c **k** + d

  where  d = cos(theta/2),  (a,b,c) = sin(theta/2)*unit_vector

represents a rotation of theta about the unit_vector axis.  It is unlikely you will need to work with these.

-- three floats (yaw, pitch, roll)

   Uses Euler Angles.

​    [https://wiki.freecadweb.org/Placement](https://wiki.freecadweb.org/Placement)

-- Vector (rotation axis) and float (rotation angle)

  see below

-- two Vectors (two axes)

   `Rotation(v1, v2)`rotates `v1` into `v2` **in the v1 - v2 plane**. (There are an infinity of other possible rotation axes)

-- Matrix object
-- 16 floats (4x4 matrix)

   The 3x3 submatrix in the top-left is the rotation part. The rest represents (unused) translation.

-- 9 floats (3x3 matrix)

https://en.wikipedia.org/wiki/Rotation_matrix#In_three_dimensions

-- 3 vectors + optional string

 `rot = FreeCAD.Rotation(Vector(0,1,0),Vector(-1,0,0),Vector(0,0,1),'ZXY')`

rotates x->y,  y->-x and z->z . Because of potential numerical error, the target triad may not be exactly orthogonal, the optional string 'ZXY'  gives the priority order of the calculation.

Of these, axis and angle is the most commonly used rotation constructor. To rotate `v1` around the axis given by `v2` by 30 degrees, we would use:

```
from FreeCAD import Rotation
rot = Rotation(v2, 30)
rotVec = rot.multVec(v1)
```

Note that we input the angle in degrees, but if we query it with `rot.Angle` we get the internal value in radians.

### Successive rotations and Euler Angles

  https://wiki.freecadweb.org/Placement#Position_and_Yaw.2C_Pitch_and_Roll

   One of the properties of quaternions that makes them so useful is that the product of two quaternions represents the result of succesive rotations to which they correspond.  The order matters!  The result of two rotations is generally not the same if the rotations are made in the reverse order.

  Yaw, pitch and roll through three *Euler* angles is a decomposition of a general rotation into three successive rotations about coordinate axes.

 First we rotate by the yaw angle about the z-axis.

Then pitch up by pitch angle about the *new* y-axis.

Finally, we roll about the *new* x axis by the roll angle.  In FreeCAD

```
rot = Rotation(10, 20 ,30) # create rotation with yaw =10, pitch = 20 and roll = 30 degrees
ryaw = Rotation(Vector(0, 0, 1), 10)
rpitch = Rotation(Vector(0, 1, 0), 20)
rroll = Rotation(Vector(0, 0, 1), 30)
rypr = ryaw.multiply(rpitch.multiply(rroll))  #creating rotation by multiplying quaternions
rot.isSame(rypr, 1e-15) # True  

```

What is a easily confused here is that in above case the successive rotations are about the *new* rotated axes. If instead, we make successive rotations about fixed coordinate axes, *not* following the body, the multiplications are made in the opposite order!

```
rz = Rotation(Vector(0, 0, 1), 90)
rx = Rotation(Vector(1, 0 ,0), 90)
rxy = Rotation(Vector(1, 1, 1), 120) #this is you what you should get if you do rz then rx
rxy1 = rz.multiply(rx)  # note the opposite order
rxy.isSame(rxy1, 1e-15) # True  1e-15 is a tolerance that allows for finite precision error
```



### Some other Vector Methods

Let v, v0, v1, v2 etc. be Vectors

```
v.distanceToLine(v1, v2)
```

This returns the perpendicular distance to the line passing though v1 in the direction v2

```
v.distanceToLineSegment(v1, v2)
```

 The name and tooltip are misleading here.  This function returns the vector to the closest point on the line segment between v1 and v2.  It is along the perpendicular if that meets the line segment, otherwise it is to the nearest endpoint.

```
v.distanceToPlane(v1, v2)
```

The plane is defined by v1, any point on it, and v2, the direction of the normal to the plane. The method returns the shortest distance to the plane - positive if v is on the side of the plane pointed to by its normal v2, negative otherwise.

### References

https://wiki.freecadweb.org/Placement

https://github.com/FreeCAD/FreeCAD/blob/5d49bf78de785a536f941f1a6d06d432582a95d3/src/Base/Rotation.cpp

