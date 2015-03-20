# Introduction #

EJML is a linear algebra library for Java.  There are three different interfaces that can be used to program in EJML; SimpleMatrix, operator, and equations. These interfaces have different design goals and range from optimizing easy of development to optimizing computational performance.  A combination of example code and tutorials are provided below that introduces a user to working with EJML.

Runtime performance and control over memory management are two key factors in the design of EJML.  Constant benchmarking and unit tests are part of its development process.  EJML is one of the faster single threaded java libraries.  Its performance relative to other libraries can be found on the [Java Matrix Benchmark](http://code.google.com/p/java-matrix-benchmark/) web page. In the future multi-threaded algorithms might be added.  Rigorous testing for numerical stability is done using internal benchmarks as well as Java Matrix Benchmark.

Detailed Javadoc documentation is provided in almost all the classes.  See the [home page](http://code.google.com/p/efficient-java-matrix-library/) for a link to the latest Javadoc.

  * For its history and acknowledgements click [HERE.](AcknowledgementsHistory.md)
  * Don't forget to check out the [Frequently Asked Questions](FAQ.md) page!

## Matrix Data Structures ##

Several different matrix data structures are provided in EJML, but only two are of interest to most users.  [DenseMatrix64F](DenseMatrix64F.md) is used by the operator programming interface and is intended for use by users who require tight control over memory and are concerned about writing optimized code.  SimpleMatrix is an easy to use wrapper around DenseMatrix64F that has more limited functionality but removes much of the tedium of working with matrices.  There is no need to choose one over the other since it is easy to switch between the two classes.

  * [DenseMatrix64F](DenseMatrix64F.md)
  * [SimpleMatrix](SimpleMatrix.md)
  * [FixedMatrix](FixedMatrix.md)

A comparison of runtime performance for different operations between SimpleMatrix and using the operations interface is provided here:

  * [SimpleMatrix versus Operator Interface Speed Comparison](SpeedSimpleMatrix.md)

Internally EJML will switch to BlockMatrix64F for very large matrices.  Many of the same functions that are provided for DenseMatrix64F are also provided for this matrix type.  While there is some benefit in terms of reduced memory foot print to using BlockMatrix64F directly, because of the added complexity it's use is not recommended.

## Example Code ##

Each example code, provided below, is fully operational and included with EJML's source code in the example directory.  They are designed to show off different concepts and provide insight into how EJML can be used to its full potential.  Full source code to all of these examples can be found in the "trunk/examples/" directory.

  1. [Kalman Filter](KalmanFilterExamples.md).
    * Example of using simple, operator, and equations interfaces.
  1. [Levenberg-Marquardt](LevenbergMarquardtExample.md)
    * Effective matrix reshaping.
    * General example.
  1. [Polynomial Fitting](PolynomialFitExample.md)
    * Creating a linear solver.
    * Using an adjustable linear solver.
  1. [QR Decomposition Implementations](QRDecompositionExample.md)
    * Demonstrates how to extract and insert using different APIs.
  1. [Principal Component Analysis](PrincipalComponentAnalysisExample.md)
    * Simple example of a popular application of linear algebra.
  1. [Finding roots of Polynomials](PolynomialRootExample.md)
    * General Eigenvalue decomposition
    * Complex eigenvalues
  1. [Extending Simple Matrix](ExtendingSimpleMatrix.md)
    * Shows how SimpleMatrix can be extended to increase its functionality in different domains.
  1. [Fixed Sized Matrices](FixedSizedMatrixExample.md)
    * Demonstrates how to create and manipulate a fixed sized matrix and vector

## Tutorials ##

When working with EJML it is often best to use factories instead of directly calling algorithms.  Factories are provided for solving linear systems and for common matrix decompositions.  This way as new algorithms are added your code will automatically use the best one.  Advanced users might wish to call a specific implementation, but most people should not do this.  Calling specific implementations is more difficult and more prone to errors if you do not understand all the implementation details.

  1. [Matlab to EJML](MatlabFunctions.md)
  1. [Introduction to Equations API](Equation.md)
  1. [Solving Linear Systems](SolvingLinearSystems.md)
  1. [Matrix Decompositions](MatrixDecomposition.md)
  1. [Random matrices, Matrix Features, and Matrix Norms](random_matrices.md)
  1. Extracting and Inserting submatrices and vectors
  1. [Matrix Input/Output](MatrixInputOutput.md)
  1. [Unit Testing](UnitTests.md)

## Linear Algebra Books ##

Want to learn more about how EJML works to write more effective code and employ more advanced techniques?  Understand where EJML's logo comes from?  The following books are recommended reading and made EJML's early development possible.
  * Best introduction to the subject that balances clarity without sacrificing important implementation details:
    * <a href='http://www.amazon.com/gp/product/0470528338/ref=as_li_ss_tl?ie=UTF8&tag=ejml-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0470528338'>Fundamentals of Matrix Computations by David S. Watkins</a><img src='http://www.assoc-amazon.com/e/ir?t=boofcv-20&l=as2&o=1&a=0470528338' alt='' border='0' width='1' height='1' />
  * Classic reference book that tersely covers hundreds of algorithms
    * <a href='http://www.amazon.com/gp/product/0801854148/ref=as_li_ss_tl?ie=UTF8&tag=ejml-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0801854148'>Matrix Computations by G. Golub and C. Van Loan</a><img src='http://www.assoc-amazon.com/e/ir?t=boofcv-20&l=as2&o=1&a=0801854148' alt='' border='0' width='1' height='1' />
  * Popular book on linear algebra
    * <a href='http://www.amazon.com/gp/product/0030105676/ref=as_li_ss_tl?ie=UTF8&tag=ejml-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0030105676'>Linear Algebra and Its Applications by Gilbert Strang</a><img src='http://www.assoc-amazon.com/e/ir?t=ejml-20&l=as2&o=1&a=0030105676' alt='' border='0' width='1' height='1' />


Purchasing through these links will help EJML's developer buy high end ramen noodles.