# Introduction #

Writing succinct and readable linear algebra code in Java, using any library, is problematic.  Originally EJML just offered two API's for performing linear algebra.  A procedural API which provided complete control over memory and which algorithms were used, but was verbose and has a sharper learning curve.  Alternatively you could use an object oriented API (SimpleMatrix) but lose control over memory and it has a limited set of operators.  Neither of these API's produces code which is all that similar to how equations are written mathematically.

Languages, such as Matlab, are specifically designed for processing matrices and are much closer to mathematical notation.  C++ offers the ability to overload operators allowing for more natural code, see [Eigen](http://eigen.tuxfamily.org).  To overcome this problem EJML now provides the _Equation_ API, which allows a Matlab/Octave like notation to be used.

This is achieved by parsing text strings with equations and converting it into a set of executable instructions.  Below is an example comparing the different API's offered in EJML.

Equations
```
eq.process("K = P*H'*inv( H*P*H' + R )");
```

SimpleMatrix
```
SimpleMatrix S = H.mult(P).mult(H.transpose()).plus(R);
SimpleMatrix K = P.mult(H.transpose().mult(S.invert()));
```

Procedural
```
mult(H,P,c);
multTransB(c,H,S);
addEquals(S,R);
if( !invert(S,S_inv) ) throw new RuntimeException("Invert failed");
multTransA(H,S_inv,d);
mult(P,d,K);
```

It is easy to see that the equations code is the most compact and readable.  While the syntax is heavily inspired by Matlab and its kin, it does not attempt to replicate their functionality.  It is also not a replacement for SimpleMatrix or the procedural API.  There are situations where those other interfaces are easier to use and most programs would need to use a mix.

Equations is designed to have minimal overhead.  It runs almost as fast as the procedural API and can be used such that all memory is predeclared.


---


# Table of Contents #
  * [Quick Start](Equation#Quick_Start.md)
  * [Compiler](Equation#The_Compiler.md)
  * [Alias](Equation#Alias.md)
  * [Submatrices](Equation#Submatrices.md)
  * [Inline Matrix](Equation#Inline_Matrix.md)
  * [Built in Functions and Variables](Equation#Built_in_Functions_and_Variables.md)

# Quick Start #

The syntax used in Equation is very similar to Matlab and other computer algebra systems (CAS).  It is assumed the reader is already familiar with these systems and can quickly pick up the syntax through these examples.

Let's start with a complete simple example then explain what's going on line by line.

```
01: public void updateP( DenseMatrix64F P , DenseMatrix64F F , DenseMatrix64F Q ) {
02:     Equation eq = new Equation();
03:     eq.alias(P,"P",F,"F",Q,"Q");
04:     eq.process("S = F*P*F'");
05:     eq.process("P = S + Q");
06: }
```

Line 02: Declare the Equation class.<br>
Line 03: Create aliases for each variable.  This allowed Equation to reference and manipulate those classes.<br>
Line 04: Process() is called and passed in a text string with an equation in it.  The variable 'S' is lazily created and set to the result of F*P*F'.<br>
Line 05: Process() is called again and P is set to the result of adding S and Q together.  Because P is aliased to the input matrix P that matrix is changed.<br>
<br>
Three types of variables are supported, matrix, double, and integer.  Results can be stored in each type and all can be aliased.  The example below uses all 3 data types and to compute the likelihood of "x" from a multivariable normal distribution defined by matrices 'mu' and 'P'.<br>
<pre><code>eq.alias(x.numRows,"k",P,"P",x,"x",mu,"mu");<br>
eq.process("p = (2*pi)^(-k/2)/sqrt(det(P))*exp(-0.5*(x-mu)'*inv(P)*(x-mu))");<br>
</code></pre>
The end result 'p' will be a double.  There was no need to alias 'pi' since it's a built in constant.  Since 'p' is lazily defined how do you access the result?<br>
<pre><code>double p = eq.lookupDouble("p");<br>
</code></pre>
For a matrix you could use eq.lookupMatrix() and eq.lookupInteger() for integers.  If you don't know the variable's type then eq.lookupVariable() is what you need.<br>
<br>
It is also possible to define a matrix inline:<br>
<pre><code>eq.process("P = [10 0 0;0 10 0;0 0 10]");<br>
</code></pre>
Will assign P to a 3x3 matrix with 10's all along it's diagonal.  Other matrices can also be included inside:<br>
<pre><code>eq.process("P = [A ; B]");<br>
</code></pre>
will concatenate A and B horizontally.<br>
<br>
Submatrices are also supported for assignment and reference.<br>
<pre><code>eq.process("P(2:5,0:3) = 10*A(1:4,10:13)");<br>
</code></pre>
P(2:5,0:3) references the sub-matrix inside of P from rows 2 to 5 (inclusive) and columns 0 to 3 (inclusive).<br>
<br>
This concludes the quick start tutorial.  The remaining sections will go into more detail on each of the subjects touched upon above.<br>
<br>
<h1>The Compiler</h1>

The current compiler is very basic and performs very literal translations of equations into code.  For example, "A = 2.5*B*C'" could be executed with a single call to CommonOps.multTransB().  Instead it will transpose C, save the result, then scale B by 2.5, save the result, multiply the results together, save that, and finally copy the result into A.  In the future the compiler will become smart enough to recognize such patterns.<br>
<br>
Compiling the text string contains requires a bit of overhead but once compiled it can be run with very fast.  When dealing with larger matrices the overhead involved is insignificant, but for smaller ones it can have a noticeable impact.  This is why the ability to precompile an equation is provided.<br>
<br>
Original:<br>
<pre><code>eq.process("K = P*H'*inv( H*P*H' + R )");<br>
</code></pre>

Precompiled:<br>
<pre><code>// precompile the equation<br>
Sequence s = eq.compile("K = P*H'*inv( H*P*H' + R )");<br>
// execute the results with out needing to recompile<br>
s.perform();<br>
</code></pre>

Both are equivalent, but if an equation is invoked multiple times the precompiled version can have a noticable improvement in performance.  Using precompiled sequences also means that means that internal arrays are only declared once and allows the user to control when memory is created/destroyed.<br>
<br>
To make it clear, precompiling is only recommended when dealing with smaller matrices or when tighter control over memory is required.<br>
<br>
When an equation is precompiled you can still change the alias for a variable.<br>
<pre><code>eq.alias(0,"sum",0,"i");<br>
Sequence s = eq.compile("sum = sum + i");<br>
for( int i = 0; i &lt; 10; i++ ) {<br>
    eq.alias(i,"i");<br>
    s.perform();<br>
}<br>
</code></pre>
This will sum up the numbers from 0 to 9.<br>
<br>
<h2>Debugging</h2>

There will be times when you pass in an equation and it throws some weird exception or just doesn't do what you expected.  To see the tokens and sequence of operations set the second parameter in compile() or peform() to true.<br>
<br>
For example:<br>
<pre><code>eq.process("y = z - H*x",true);<br>
</code></pre>
When application is run it will print out<br>
<pre><code>Parsed tokens:<br>
------------<br>
VarMATRIX<br>
ASSIGN<br>
VarMATRIX<br>
MINUS<br>
VarMATRIX<br>
TIMES<br>
VarMATRIX<br>
<br>
Operations:<br>
------------<br>
multiply-mm<br>
subtract-mm<br>
copy-mm<br>
</code></pre>

<h1>Alias</h1>

To manipulate matrices in equations they need to be aliased.  Both DenseMatrix64F and SimpleMatrix can be aliased.  A copy of scalar numbers can also be aliased.  When a variable is aliased a reference to the data is saved and a name associated with it.<br>
<pre><code>DenseMatrix64F x = new DenseMatrix64F(6,1);<br>
eq.alias(x,"x");<br>
</code></pre>
Multiple variables can be aliased at the same time too<br>
<pre><code>eq.alias(x,"x",P,"P",h,"Happy");<br>
</code></pre>
As is shown above the string name for a variable does not have to be the same as Java name of the variable.  Here is an example where an integer and double is aliased.<br>
<pre><code>int a = 6;<br>
eq.alias(2.3,"distance",a,"a");<br>
</code></pre>

After a variable has been aliased you can alias the same name again to change it.  Here is an example of just that:<br>
<pre><code>for( int i = 0; i &lt; 10; i++ ) {<br>
    eq.alias(i,"i");<br>
    // do stuff with i<br>
}<br>
</code></pre>

If after benchmarking your code and discover that the alias operation is slowing it down (a hashmap lookup is done internally) then you should consider the following faster, but uglier, alternative.<br>
<pre><code>VariableInteger i =  eq.lookupVariable("i");<br>
for( i.value = 0; i.value &lt; 10; i.value++ ) {<br>
    // do stuff with i<br>
}<br>
</code></pre>

<h1>Submatrices</h1>

Sub-matrices can be read from and written to.  It's easy to reference a sub-matrix inside of any matrix. A few examples are below.<br>
<pre><code>A(1:4,0:5)<br>
</code></pre>
Here rows 1 to 4 (inclusive) and columns 0 to 5 (inclusive) compose the sub-matrix of A.  The notation "a:b" indicates an integer set from 'a' to 'b', where 'a' and 'b' must be integers themselves.  To specify every row or column do ":" or all rows and columns past a certain 'a' can be referenced with "a:".  Finally, you can reference just a number by typeing it, e.g. "a".<br>
<pre><code>A(3:,3)  &lt;-- Rows from 3 to the last row and just column 3<br>
A(:,:)   &lt;-- Every element in A<br>
A(1,2)   &lt;-- The element in A at row=1,col=2<br>
</code></pre>
The last example is a special case in that A(1,2) will return a double and not 1x1 matrix.  Consider the following:<br>
<pre><code>A(0:2,0:2) = C/B(1,2)<br>
</code></pre>
The results of dividing the elements of matrix C by the value of B(1,2) is assigned to the submatrix in A.<br>
<br>
A named variable can also be used to reference elements as long as it's an integer.<br>
<pre><code>a = A(i,j)<br>
</code></pre>

<h1>Inline Matrix</h1>

Matrices can be created inline and are defined inside of brackets. The matrix is specified in a row-major format, where a space separates elements in a row and a semi-colon indicates the end of a row.<br>
<pre><code>[5 0 0;0 4.0 0.0 ; 0 0 1]<br>
</code></pre>
Defines a 3x3 matrix with 5,4,1 for it's diagonal elements.  Visually this looks like:<br>
<pre><code>[ 5 0 0 ]<br>
[ 0 4 0 ]<br>
[ 0 0 1 ]<br>
</code></pre>
An inline matrix can be used to concatenate other matrices together.<br>
<pre><code>[ A ; B ; C ]<br>
</code></pre>
Will concatenate matrices A, B, and C along their rows.  They must have the same number of columns.  As you might guess, to concatenate along columns you would<br>
<pre><code>[ A B C ]<br>
</code></pre>
and each matrix must have the same number of rows.  Inner matrices are also allowed<br>
<pre><code>[ [1 2;2 3] [4;5] ; A ]<br>
</code></pre>
which will result in<br>
<pre><code>[ 1 2 4 ]<br>
[ 2 3 5 ]<br>
[   A   ]<br>
</code></pre>

<h1>Built in Functions and Variables</h1>
<b>Constants</b>
<pre><code>pi = Math.PI<br>
e  = Math.E<br>
</code></pre>

<b>Functions</b>
<pre><code>eye(N)       Create an identity matrix which is N by N.<br>
eye(A)       Create an identity matrix which is A.numRows by A.numCols<br>
normF(A)     Frobenius normal of the matrix.<br>
det(A)       Determinant of the matrix<br>
inv(A)       Inverse of a matrix<br>
pinv(A)      Pseudo-inverse of a matrix<br>
rref(A)      Reduced row echelon form of A<br>
trace(A)     Trace of the matrix<br>
zeros(r,c)   Matrix full of zeros with r rows and c columns.<br>
ones(r,c)    Matrix full of ones with r rows and c columns.<br>
diag(A)      If a vector then returns a square matrix with diagonal elements filled with vector<br>
diag(A)      If a matrix then it returns the diagonal elements as a column vector<br>
dot(A,B)     Returns the dot product of two vectors as a double.  Does not work on general matrices.<br>
solve(A,B)   Returns the solution X from A*X = B.<br>
kron(A,B)    Kronecker product<br>
abs(A)       Absolute value of A.<br>
max(A)       Element with the largest value in A.<br>
min(A)       Element with the smallest value in A.<br>
pow(a,b)     Scalar power of a to b.  Can also be invoked with "a^b".<br>
sin(a)       Math.sin(a) for scalars only<br>
cos(a)       Math.cos(a) for scalars only<br>
atan(a)      Math.atan(a) for scalars only<br>
atan2(a,b)   Math.atan2(a,b) for scalars only<br>
exp(a)       Math.exp(a) for scalars and element-wise matrices<br>
log(a)       Math.log(a) for scalars and element-wise matrices<br>
</code></pre>
<b>Symbols</b>
<pre><code>'*'        multiplication (Matrix-Matrix, Scalar-Matrix, Scalar-Scalar)<br>
'+'        addition (Matrix-Matrix, Scalar-Matrix, Scalar-Scalar)<br>
'-'        subtraction (Matrix-Matrix, Scalar-Matrix, Scalar-Scalar)<br>
'/'        divide (Matrix-Scalar, Scalar-Scalar)<br>
'/'        matrix solve "x=b/A" is equivalent to x=solve(A,b) (Matrix-Matrix)<br>
'^'        Scalar power.  a^b is a to the power of b.<br>
'\'        left-divide.  Same as divide but reversed.  e.g. x=A\b is x=solve(A,b)<br>
'.*'       element-wise multiplication (Matrix-Matrix)<br>
'./'       element-wise division (Matrix-Matrix)<br>
'.^'       element-wise power. (scalar-scalar) (matrix-matrix) (scalar-matrix) (matrix-scalar)<br>
'^'        Scalar power.  a^b is a to the power of b.<br>
'''        matrix transpose<br>
'='        assignment by value (Matrix-Matrix, Scalar-Scalar)<br>
</code></pre>

<h1>User Defined Functions</h1>

It's easy to add your own custom functions too.  A custom function implements ManagerFunctions.Input1 or ManagerFunctions.InputN, depending on the number of inputs it takes.  It is then added to the ManagerFunctions in Equation by call add().  The output matrix should also be resized.<br>
<br>
<a href='https://github.com/lessthanoptimal/ejml/blob/equation/examples/src/org/ejml/example/EquationCustomFunction.java'>Custom Function Example</a>