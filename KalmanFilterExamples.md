# Introduction #

Here are three examples that demonstrate how a Kalman filter can be created using different API's in EJML.  Each API has different advantages and disadvantages.  High level interfaces tend to be easier to use, but sacrifice efficiency.  The intent of this article is to illustrate

API Overview:
  * _SimpleMatrix_ use an object oriented API and is fairly easy to write, but slower.
  * _Operations_ uses a mostly procedural API.  Harder to read and write, has the most control over algorithms and memory, and is the fastest.
  * _Equations_ compiles symbolic equations at runtime.  Easiest to read/write, less control than Operation, and has middle performance.  In the future it could produce code as fast as Operations.

Performance Summary:
| API | Execution Time (ms) |
|:----|:--------------------|
| SimpleMatrix | 1875 |
| Operations | 1280 |
| Equations | 1698 |

Direct Links:
  * [SimpleMatrix](#SimpleMatrix_Example.md)
  * [Operations](#Operations_Example.md)
  * [Equations](#Equations_Example.md)

All example code is included in EJML's source code directory.  You can also view them in GitHub:
[GitHub Example Code](https://github.com/lessthanoptimal/ejml/tree/master/examples)

**NOTE:** While the Kalman filter code below is fully functional and will work well in most applications, it might not be the best.  Other variants seek to improve stability and/or avoid the matrix inversion at the cost of added code complexity.

## SimpleMatrix Example ##

```
/**
 * A Kalman filter implemented using SimpleMatrix.  The code tends to be easier to
 * read and write, but the performance is degraded due to excessive creation/destruction of
 * memory and the use of more generic algorithms.  This also demonstrates how code can be
 * seamlessly implemented using both SimpleMatrix and DenseMatrix64F.  This allows code
 * to be quickly prototyped or to be written either by novices or experts.
 *
 * @author Peter Abeles
 */
public class KalmanFilterSimple implements KalmanFilter{

    // kinematics description
    private SimpleMatrix F;
    private SimpleMatrix Q;
    private SimpleMatrix H;

    // sytem state estimate
    private SimpleMatrix x;
    private SimpleMatrix P;


    @Override
    public void configure(DenseMatrix64F F, DenseMatrix64F Q, DenseMatrix64F H) {
        this.F = new SimpleMatrix(F);
        this.Q = new SimpleMatrix(Q);
        this.H = new SimpleMatrix(H);
    }

    @Override
    public void setState(DenseMatrix64F x, DenseMatrix64F P) {
        this.x = new SimpleMatrix(x);
        this.P = new SimpleMatrix(P);
    }

    @Override
    public void predict() {
        // x = F x
        x = F.mult(x);

        // P = F P F' + Q
        P = F.mult(P).mult(F.transpose()).plus(Q);
    }

    @Override
    public void update(DenseMatrix64F _z, DenseMatrix64F _R) {
        // a fast way to make the matrices usable by SimpleMatrix
        SimpleMatrix z = SimpleMatrix.wrap(_z);
        SimpleMatrix R = SimpleMatrix.wrap(_R);

        // y = z - H x
        SimpleMatrix y = z.minus(H.mult(x));

        // S = H P H' + R
        SimpleMatrix S = H.mult(P).mult(H.transpose()).plus(R);

        // K = PH'S^(-1)
        SimpleMatrix K = P.mult(H.transpose().mult(S.invert()));

        // x = x + Ky
        x = x.plus(K.mult(y));

        // P = (I-kH)P = P - KHP
        P = P.minus(K.mult(H).mult(P));
    }

    @Override
    public DenseMatrix64F getState() {
        return x.getMatrix();
    }

    @Override
    public DenseMatrix64F getCovariance() {
        return P.getMatrix();
    }
}
```

## Operations Example ##

```
/**
 * A Kalman filter that is implemented using the operations API, which is procedural.  Much of the excessive
 * memory creation/destruction has been reduced from the KalmanFilterSimple. A specialized solver is 
 * under to invert the SPD matrix.  
 * 
 * @author Peter Abeles
 */
public class KalmanFilterOperations implements KalmanFilter{

    // kinematics description
    private DenseMatrix64F F;
    private DenseMatrix64F Q;
    private DenseMatrix64F H;

    // system state estimate
    private DenseMatrix64F x;
    private DenseMatrix64F P;

    // these are predeclared for efficiency reasons
    private DenseMatrix64F a,b;
    private DenseMatrix64F y,S,S_inv,c,d;
    private DenseMatrix64F K;

    private LinearSolver<DenseMatrix64F> solver;

    @Override
    public void configure(DenseMatrix64F F, DenseMatrix64F Q, DenseMatrix64F H) {
        this.F = F;
        this.Q = Q;
        this.H = H;

        int dimenX = F.numCols;
        int dimenZ = H.numRows;

        a = new DenseMatrix64F(dimenX,1);
        b = new DenseMatrix64F(dimenX,dimenX);
        y = new DenseMatrix64F(dimenZ,1);
        S = new DenseMatrix64F(dimenZ,dimenZ);
        S_inv = new DenseMatrix64F(dimenZ,dimenZ);
        c = new DenseMatrix64F(dimenZ,dimenX);
        d = new DenseMatrix64F(dimenX,dimenZ);
        K = new DenseMatrix64F(dimenX,dimenZ);

        x = new DenseMatrix64F(dimenX,1);
        P = new DenseMatrix64F(dimenX,dimenX);

        // covariance matrices are symmetric positive semi-definite
        solver = LinearSolverFactory.symmPosDef(dimenX);
    }

    @Override
    public void setState(DenseMatrix64F x, DenseMatrix64F P) {
        this.x.set(x);
        this.P.set(P);
    }

    @Override
    public void predict() {

        // x = F x
        mult(F,x,a);
        x.set(a);

        // P = F P F' + Q
        mult(F,P,b);
        multTransB(b,F, P);
        addEquals(P,Q);
    }

    @Override
    public void update(DenseMatrix64F z, DenseMatrix64F R) {
        // y = z - H x
        mult(H,x,y);
        subtract(z, y, y);

        // S = H P H' + R
        mult(H,P,c);
        multTransB(c,H,S);
        addEquals(S,R);

        // K = PH'S^(-1)
        if( !solver.setA(S) ) throw new RuntimeException("Invert failed");
        solver.invert(S_inv);
        multTransA(H,S_inv,d);
        mult(P,d,K);

        // x = x + Ky
        mult(K,y,a);
        addEquals(x,a);

        // P = (I-kH)P = P - (KH)P = P-K(HP)
        mult(H,P,c);
        mult(K,c,b);
        subtractEquals(P, b);
    }

    @Override
    public DenseMatrix64F getState() {
        return x;
    }

    @Override
    public DenseMatrix64F getCovariance() {
        return P;
    }
}
```

## Equations Example ##

```
/**
 * Example of how the equation interface can greatly simplify code
 *
 * @author Peter Abeles
 */
public class KalmanFilterEquation implements KalmanFilter{

    // system state estimate
    private DenseMatrix64F x;
    private DenseMatrix64F P;

    private Equation eq;

    // Storage for precompiled code for predict and update
    Sequence predictX,predictP;
    Sequence updateY,updateK,updateX,updateP;

    @Override
    public void configure(DenseMatrix64F F, DenseMatrix64F Q, DenseMatrix64F H) {
        int dimenX = F.numCols;

        x = new DenseMatrix64F(dimenX,1);
        P = new DenseMatrix64F(dimenX,dimenX);

        eq = new Equation();

        // Provide aliases between the symbolic variables and matrices we normally interact with
        // The names do not have to be the same.
        eq.alias(x,"x",P,"P",Q,"Q",F,"F",H,"H");

        // Dummy matrix place holder to avoid compiler errors.  Will be replaced later on
        eq.alias(new DenseMatrix64F(1,1),"z");
        eq.alias(new DenseMatrix64F(1,1),"R");

        // Pre-compile so that it doesn't have to compile it each time it's invoked.  More cumbersome
        // but for small matrices the overhead is significant
        predictX = eq.compile("x = F*x");
        predictP = eq.compile("P = F*P*F' + Q");

        updateY = eq.compile("y = z - H*x");
        updateK = eq.compile("K = P*H'*inv( H*P*H' + R )");
        updateX = eq.compile("x = x + K*y");
        updateP = eq.compile("P = P-K*(H*P)");
    }

    @Override
    public void setState(DenseMatrix64F x, DenseMatrix64F P) {
        this.x.set(x);
        this.P.set(P);
    }

    @Override
    public void predict() {
        predictX.perform();
        predictP.perform();
    }

    @Override
    public void update(DenseMatrix64F z, DenseMatrix64F R) {

        // Alias will overwrite the reference to the previous matrices with the same name
        eq.alias(z,"z"); eq.alias(R,"R");

        updateY.perform();
        updateK.perform();
        updateX.perform();
        updateP.perform();
    }

    @Override
    public DenseMatrix64F getState() {
        return x;
    }

    @Override
    public DenseMatrix64F getCovariance() {
        return P;
    }
}
```