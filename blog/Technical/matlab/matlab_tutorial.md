#### help

```
help name2check
```

#### comment

```
% this is comment

%% used as code section, can be run in editor by clicking 'run section'
```



#### size

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

m = size(A, 1)
n = size(A, 2)

v = [1, 2, 3]
l = length(v)
```

#### arith

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

#### Identity matrix

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

#### Transpose and inverse

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

#### pi

```
a = pi;
```

#### logical

```
1 == 2
1 ~= 2
```

#### binary

```
1 && 2
1 || 0
xor(1, 0)
```

#### display

```
disp(a) % print a
disp(sprintf('%0.2f',a))


```

#### create vector

```matlib
v = 1:1:9
v = [1;2]
v = ones(2,3) | zeros | rand | randn

a(2, :) the second row

v = [v; 3]

v = [A B] --> A on the left and B on the right
v = [A;B] --> A on the top

v = a(:) ---> all elements

v = linspace(0,3,8) % create from 0 to 3 by 8 pieces ,called Uniformly Spaced Vectors 
```

Useful function: rand, ones, eye, randi, randn, zeros, toeplitz, vander, diag, magic, hilb  

#### display as graph

```matlib
w = -6 + randn(1, 10000);
hist(w)
hist(w, 50)

 t = (0 : 0.01 : 0.98);
 y1 = sin(2*pi*4*t);
 y2 = cos(2*pi*4*t);
 
 % ---- in a single figure
%  plot(t, y1);
%  hold on;
%  plot(t, y2, 'r');
%  xlabel('time');
%  ylabel('value');
%  legend('sin', 'cos');
%  title('my plot');

 %---- in two figures
%  figure(1);plot(t, y1);
%  figure(2);plot(t, y2);

% --- in single figure as sub-graph
% subplot(1, 2, 1);
% plot(t, y1);
% subplot(1, 2, 2);
% plot(t, y2);
% axis([0.5 1 0 1]);

% ---- draw matrix with color
A = magic(5);
imagesc(A), colorbar, colormap gray;

plot(x, y, Param) % param = m:s | g--* | r-
```

#### Who

```
who ---> show the variables in memory
whos
```

#### load/save

```
d = load("file")
v = d(1:10)

save tem.mat v;
load tem.mat -> there is v

save tem.txt v -ascii;
```

#### Computation

```matlib
A = [1 2; 3 4; 5 6];
B = [11 12; 13 14; 15 16];
C = [1 -1; 2 2];
D = A*C;
E = A .* B;
F = A .^ 2;
G = abs(C);

H = [1 2 3 4 5];
a = max(H);
[val, index] = max(H);

I = [1 2; 2 3; 2 1];
b = max(I);

K = I < 3;

M = magic(3); % row = col = cross

[r, c] = find(M>=7);

v = [1 0.5 0.2 5 0.75];
sum(v)
prod(v)
floor(v) % int smaller
ceil(v) % int larger

N = rand(4); % generate 4*4

max(A,[],1) % max of col
max(A, [], 2) % max of row

O = eye(5);
P = flipud(O); % up down the O
O
P

v = [1; 4; 3; 2; 5]
I = v < 4 % I = [1;0;1;1;0]
r = v(I) % r = [1;3;2]
```

#### Control Statements

```matlib
v = zeros(10,1);
for i=1:10
    v(i) = 2^i;
end
disp(v)

i = 1;
while i <= 5
    v(i) = 2*i;
    i = i+1;
end
disp(v)

i = 1;
while true
    v(i) = i;
    i = i+1;
    if i > 6
        break;
    end
end
disp(v)
```

#### Function

```matlab
same name of the file and the function name

% in the file named squareAndCube.m
function [x2, x3] = squareAndCube(x)
x2 = x^2;
x3 = x^3;

% use in command window
>> [a, b] = squareAndCube(5)

a =

    25


b =

   125
   

% cost function
function J = costFuncJ(X, y, theta)
m = size(X, 1);
predicates = X*theta;
sqrErrors = (predicates - y).^2;
J = 1/(2*m)*sum(sqrErrors);

>> X = [1 1; 1 2; 1 3];
y = [1; 2; 3];
theta = [0;1];
Result = costFuncJ(X, y, theta);
disp(Result);
     0
     
>> theta = [0;0];
>> Result = costFuncJ(X, y, theta);
>> disp(Result);
    2.3333
```

#### Build-in

* mean | std 



