
```
Latest Version: 0.26                                Released: September 15, 2014
```

## LICENSE CHANGED FROM LGPL to Apache v2.0 ##

![http://efficient-java-matrix-library.googlecode.com/svn/ejml_logo_medium.gif](http://efficient-java-matrix-library.googlecode.com/svn/ejml_logo_medium.gif)

## Introduction ##

Efficient Java Matrix Library (EJML) is a [linear algebra](http://en.wikipedia.org/wiki/Linear_algebra) library for manipulating dense matrices. Its design goals are; 1) to be as computationally and memory efficient as possible for both small and large matrices, and 2) to be accessible to both novices and experts.  These goals are accomplished by dynamically selecting the best algorithms to use at runtime and by designing a clean API.  EJML is free, written in 100% Java and has been released under an Apache v2.0 license.

EJML has three distinct ways to interact with it.  This allows a programmer to choose between simplicity and efficiency.  1) A simplified interface that allows a more object oriented way of programming.  2) Procedural interface that provides complete control over memory, speed, and specific algorithms.  3) Equations which provide a way to integrate symbolic equations similar to Matlab and other Compute Algebra Systems (CAS).  In general EJML is one the fastest _single_ threaded pure Java library and has additional optimizations for small matrices.  See [Java Matrix Benchmark](http://code.google.com/p/java-matrix-benchmark/) for a detailed comparison of different libraries.

The following functionality is provided:
  * Basic Operators (addition, multiplication, ... )
  * Matrix Manipulation (extract, insert, combine, ... )
  * Linear Solvers (linear, least squares, incremental, ... )
  * Decompositions (LU, QR, Cholesky, SVD, Eigenvalue, ...)
  * Matrix Features (rank, symmetric, definitiveness, ... )
  * Random Matrices (covariance, orthogonal, symmetric, ... )
  * Different Internal Formats (row-major, block)
  * Unit Testing

[A list of projects which use EJML is available by clicking HERE!](ProjectsThatUseEJML.md)



## [Source Code on GitHub](https://github.com/lessthanoptimal/ejml) ##
## [Download from SourceForge](https://sourceforge.net/projects/ejml/files/v0.26/) ##

Maven
```
<groupId>com.googlecode.efficient-java-matrix-library</groupId>
<artifactId>ARTIFACT</artifactId>
<version>0.26</version>
ARTIFACT is core or equation
```
Gradle
```
compile group: 'com.googlecode.efficient-java-matrix-library', name: 'core', version: '0.26'
compile group: 'com.googlecode.efficient-java-matrix-library', name: 'equation', version: '0.26'
```

## Documentation ##

A complete set of (mostly) up to date documentation is provided for EJML in the form of an online manual, several examples, and code comments.  Documentation can be accessed online by clicking on the links below.
  * <font size='4'> <a href='EjmlManual.md'>Manual with tutorials.</a> </font>
  * <font size='4'> <a href='http://ejml.org/javadoc/'>Online JavaDoc.</a> </font>
  * <font size='4'> <a href='FAQ.md'>Frequently Asked Questions</a> </font>

## Reporting Bugs ##

Bugs can be reported using the "Issues" tab above, by clicking [HERE](http://code.google.com/p/efficient-java-matrix-library/issues/list), or writing to the [efficient-java-matrix-library-discuss](http://groups.google.com/group/efficient-java-matrix-library-discuss) mailing list.

## Comments and Suggestions ##

For additional **help** or to make a **comment/suggestion** please use [efficient-java-matrix-library-discuss](http://groups.google.com/group/efficient-java-matrix-library-discuss).  Constructive feedback is always welcomed.  _If you find this library to be useful feel free to post a quick comment too._

## External Libraries Used ##
EJML is entirely self contained and is only dependent on JUnit for tests.
  * http://www.junit.org/