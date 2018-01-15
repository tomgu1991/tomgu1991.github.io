```matlab
% The ; denotes we are going back to a new row.
A = [1, 2, 3; 4, 5, 6; 7, 8, 9; 10, 11, 12]
% Initialize a vector 
v = [1;2;3] 
% Get the dimension of the matrix A where m = rows and n = columns
[m,n] = size(A)
% You could also store it this way
dim_A = size(A)
% Get the dimension of the vector v 
dim_v = size(v)
% Now let's index into the 2nd row 3rd column of matrix A
A_23 = A(2,3)
```

```matlib
% Initialize matrix A and B 
A = [1, 2, 4; 5, 3, 2]
B = [1, 3, 4; 1, 1, 1]
% Initialize constant s 
s = 2
% See how element-wise addition works
add_AB = A + B 
% See how element-wise subtraction works
sub_AB = A - B
% See how scalar multiplication works
mult_As = A * s
% Divide A by s
div_As = A / s
% What happens if we have a Matrix + scalar?
add_As = A + s
```

```matlab
% Initialize matrix A 
A = [1, 2, 3; 4, 5, 6;7, 8, 9] 
% Initialize vector v 
v = [1; 1; 1] 
% Multiply A * v
Av = A * v
```
```matlab
A = [1, 2; 3, 4;5, 6]
% Initialize a 2 by 1 matrix 
B = [1; 2] 
% We expect a resulting matrix of (3 by 2)*(2 by 1) = (3 by 1) 
mult_AB = A*B
```

```matlib
A =
     1     2
     4     5
B =
     1     1
     0     2
% Initialize a 2 by 2 identity matrix
I = eye(2)
I =
     1     0
     0     1
```

```matlib
A =
     1     2     0
     0     5     6
     7     0     9
% Transpose A 
A_trans = A' 
A_trans =
     1     0     7
     2     5     0
     0     6     9

% Take the inverse of A 
A_inv = inv(A)
A_inv =
    0.3488   -0.1395    0.0930
    0.3256    0.0698   -0.0465
   -0.2713    0.1085    0.0388

% What is A^(-1)*A? 
A_invA = inv(A)*A
A_invA =
    1.0000   -0.0000    0.0000
    0.0000    1.0000   -0.0000
   -0.0000    0.0000    1.0000
```

