## 4、基础知识

### 4.1、 `` cvx_begin   `` 和 `` cvx_end `` 

所有的CVX模型都必须先于命令 `` cvx_begin `` ，并在命令 `` cvx_end `` 处截止。所有变量声明，目标函数和约束都必须放在两个命令中间。 `` cvx_begin `` 可以包括一种或多种修改模式：

-  `` cvx_begin qiute `` ：在解决模型时阻止任何屏幕输出。
-  `` cvx_begin sdp `` ：调用半正定规划模型（详情参考 *semidefinite programming mode*）。
-  `` cvx_begin gp `` ：调用几何规划模型（详情参考 *geometric programming mode*）。

有需要的时候，这些修改模式可以组合使用。举个例子， `` cvx_begin sdp quite `` 会调用SDP模型并使求解器在输出时静默。

### 4.2、变量

所有变量在被用在约束和目标函数之前都必须通过 `` variable `` 命令进行声明（或者是 `` variables `` 命令，查阅以下内容）。一个 `` variable `` 命令包括变量的名称，一个可选的维度列表和一个或多个关键字用于提供该变量的内容或结构的额外信息。

变量可以是实数或者是复数变量，向量，矩阵或者 $ n $ 维数组。举个例子，

```matlab
variable X
variable Y(20,10)
variable Z(5,5,5)
```

以上三个命令声明了326个（标量）变量：一个标量 `` X `` ，一个 $ 20\times10 $ 的矩阵 `` Y `` （包含了200​个标量变量）和一个 $ 5\times5\times5 $ 的数组 `` Z `` （包含了125个标量变量）。

变量声明可以包括一个或多个关键字去说明该变量的结构或是条件。举个例子，想要去声明一个复数变量，可以用 `` complex `` 关键字：

```matlab
variable w(50) comnplex
```

非负变量和对称/赫米特正半定（positive semidefinite，PSD）矩阵可以分别用 `` nonnegetive `` 和 `` semidefinite `` 关键字来声明：

```matlab
variable X(10) nonnegetive
variable Z(5,5) semidefinite
variable Q(5,5) complex semidefinite
```

在这个例子中， `` X `` 是一个非负的向量， `` Z `` 是一个实数对称的PSD矩阵，而 `` Q `` 是一个复数赫米特PSD矩阵。 正如我们下面看到的，赫米特半定可以看做是第三个命令的等效写法。

对于MIDCPs，关键字 `` integer `` 和 `` binary `` 被用于声明整数和二元变量，分别写做：

```matlab
variable p(10) integer
variable q binary
```

各种各样的关键字可以帮助构建带有矩阵结构的变量，例如对称性或者条带化。举例说明，代码段为：

```matlab
variable Y(50,50) symmetric
variable Z(100,100) hermitian toeplitz
```

声明 `` Y `` 为一个实数 $ 50\times50 $ 对称矩阵变量，而且 `` Z `` 为一个 $ 100\times100 $ 的埃尔米特·托普利兹矩阵变量（关键字 `` hermitian `` 也指明这个矩阵为复数）。当前支持的结构关键字有：

```matl
banded(lb,ub)		diagonal		 hankel			 hermitian
skew_symmetric		symmetric		 toeplitz		 tridiagonal
lower_bidiagonal	lower_hessenberg lower_triangular	
upper_bidiagonal	upper_hankel	 upper_hessenberg upper_triangular
```

下划线可以被省略。举个例子， `` lower triangular `` 也可以被被接受。这些关键字都是不言自明的，除了以下几个例外：

** `` banded(lb,ub) `` **：下边界为lb上边界为ub的矩阵。如果上下边界都为零，则该矩阵为对角矩阵，其中上边界可以被忽略，因为上下边界相同。例如， `` banded(1,1) `` （或者 `` banded(1) `` ）为一个三对角线矩阵。

** `` upper_hankel `` **：该矩阵为汉克尔（Hankel）矩阵（每一条逆对角线上的元素相等），且逆对角线下的元素为零，即 $ i+j>n+1 $ 。

当多个关键字被使用的时候，生成的矩阵结构由关键字的交集构成。举个例子， `` symmetric tridiagonal `` 是有效的组合形式。也就是说，在本例中，如果存在更为合理的替代方式—— `` diagonal `` ，则CVX将会拒绝例如 `` symmetric lower_triangular `` 的组合关键字。此外，如果关键字完全冲突，以致于**没有**非零矩阵满足所有的关键字，那么将会得到一个错误。

矩阵专用的关键字也可以用在 $ n $ 维数组上：每一个数组的 $ 2 $ 维切片都具有指定的结构。举例说明，下面的声明

```matl
variable R(10,10,8) hermitian semidefinite
```

构建了8个 $10\times10$ 的复数赫米特PSD矩阵，储存在 `` R `` 的 $2$ 维切片中。

尽管 `` variable `` 的声明很灵活，但是其只能声明单个变量，如果您有很多变量需要去声明，这个命令是很不方便的。出于这个原因，CVX还有提供了 `` variables `` 允许您声明多个变量，即

```matlab
variables x1 x2 x3 y1(10) y2(10,10,10);
```

 `` variables `` 命令有一个限制就是它不能声明复数，整数或者是结构化的变量。这些变量必须使用 `` variable `` 命令单独依次声明。

### 4.3、目标函数

声明目标函数需要使用 `` minimize `` 或 `` maximize `` 函数（ `` minimise `` 和 `` maximise `` 同样适用）。最小化调用中的目标函数必须是凸的，最大化调用中的目标函数必须是凹的。举例说明，

```matlab
minimize(norm(x,1))
maximize(geo_mean(x))
```

在CVX标准中，最多只有一个目标函数可以被声明，而且必须返回一个标量值。

如果没有目标函数被指定，则问题会被解释为一个可行性问题，该问题与执行一个目标函数设置为零的最小化问题一样。在本例中，如果可行解存在，则cvx_optval（CVX的最优值）为0，如果不存在则cvx_optval为+Inf（正无穷）。

### 4.4、约束

CVX支持一下的约束类型：

- 等式约束，用 `` == `` 构建，两边都是放射的。
- 小于等于约束，用 `` <= `` 构建，其中左边是凸函数，右边是凹函数。
- 大于等于约束，用 `` >= `` 构建，其中左边是凹函数，右边是凸函数。

不等号操作符 `` ~= `` 几乎不用在约束中，在任意的例子中，这样的约束几乎是不凸的。最新版本的CVX支持不等式约束的链式写法，举例说明， `` l<=x<=u `` （之前的版本是不支持这样的链式写法的，模目前最新的版本为Release 2.2）。

需要注意区分单等号 `` = `` 和双等号 `` == `` 的区别，前者是赋值，后者表示相等，详情请参考下面的 *Assignment and expression holders*。

严格不等式约束 `` < `` 和 `` > `` 也可以使用，但它们的解释与非严格对应的解释相同。强烈建议不要使用严格约束，之后版本的CVX可能会将其一起移出，其背后的原因请查阅更详细的讨论 *Srtict inequalities*。

不等式和等式约束以元素方式使用，与MATLAB本身的行为匹配。举例说明，如果如果 `` A `` 和 `` B `` 都是 $ m\times n $ 的数组，则 `` A<=B `` 可以被解释为 $ mn $ 个（标量）不等式 `` A(i,j)<=B(i,j) `` 。当其中一边为标量时，该标量值要复制的，举个例子， `` A>0 `` 被解释为 `` A(i,j)>=0 `` 。

这种元素形式的处理方式可以用 *semidefinite programming mode* 来替换，详情查阅对应章节。

CVX也支持集合成员关系约束，详情查阅下面的 *Set membership*。

### 4.5、函数

基本的CVX函数库包含了一系列的可接受CVX变量或者是表达式作为参数的凸函数，凹函数和仿射函数。其中许多都是MATLAB函数，例如： `` sum `` ， `` trace `` ， `` diag `` ， `` sqrt `` ， `` max `` 和 `` min `` ，可根据需要重新实现用来支持CVX。还有一些MATLAB没有支持的新函数。基本库中完整的函数列表可以查阅 *Reference guide*。您也可以将您的新函数添加到其中，具体查阅 *Adding new functions to the atom library*。

基本函数中的一个例子就是二次线性函数 `` quad_over_lin `` ：
$$
f: \mathbf{R}^{n} \times \mathbf{R} \rightarrow \mathbf{R}, \quad f(x, y)=\left\\{\\begin{array}{ll}
x^{T} x / y & y>0 \\\
+\infty & y \leq 0
\\end{array}\right.
$$
（该函数也接受复数 $ x $ ，但是为了简单这里只考虑实数 $ x $ ）。二次线性函数在 $ x $ 和 $ y $ 上都是都是凸函数，而且可以被当做目标函数，或者用在合适的约束中，或者用在更复杂的表达式中。举例说明，我们可以最小化二次线性函数 $ (Ax-b,c^Tx+d) $ 中，

```matlab
minimize(quad_over_lin(A*x-b,c'*x+d))；
```

在CVX标准中，假设 $ x $ 为向量最优化变量， $ A $ 为一个矩阵， $ b $ 和 $ c $ 为向量，而 $ d $ 为一个标量。 CVX可以将目标表达式识别为凸函数，因为其是有带有一个仿射函数的凸函数（二次线性函数）组成的。

您也可以在CVX标准外使用函数 `` quad_over_lin `` 。在本例中，该函数只是计算了在给定（数值类型）参数的情况下的（数值类型）值。如果 `` c'*x+d `` 是正值，那么该结果数值上等效于

```matlab
((A*x-b)'-(A*x-b))/(c'*x+d)
```

然而， `` quad_over_lin `` 命令会对定义域进行检查，如果 `` c'*x+d `` 为零或者是负值，那么会返回一个 `` Inf `` 。

### 4.6、集合成员关系

CVX支持凸集的定义和使用。基本库中包含了正半定 $ n\times n $ 矩阵的锥，二阶或洛伦兹（Lorentz）锥以及各种范式球（norm ball）。基本库支持的集合的完成列表请在 *Sets* 中查阅。

不幸的是，MATLAB并不支持关系例如 `` x in S `` 表示 $ x \in S $ 。因此在CVX中，我们使用略有不同的语法来要求表达式必须在集合中。我们用一个返回单个要求在集合中的未命名变量的函数来表示集合。举例说明，考虑 $ \mathbf{S}^n_+ $ ，即对称正半定 $ n\times n $ 矩阵。在CVX中，我们通过函数 `` semidefinite(x) `` 来表示，该函数会返回一个新的被约束为正半定的未命名变量。为了要求矩阵表达式 `` X `` 为对称正半定，我们使用下列语法。

```matlab
X==semidefinite(n)
```

字面上的意思就是 `` X `` 被约束为一些未命名的变量，而这些变量被要求是 $ n\times n $ 的对称正半定矩阵。当然，这就相当于说 `` X `` 本身必须为对称半正定矩阵。

举个例子，考虑这样的一个约束，（矩阵）变量 `` X `` 为一个相关矩阵，换句换说，就是该变量是对称的，具有单位对角元素和是正半定的。在CVX中我们可以通过以下的命令来声明这样的变量并加以约束条件。

```matlab
variable X(n,n) symmetric;
X==semidefinite(X);
diag(X)==1;
```

上述命令的第二行将 `` X `` 加上约束使其成为正半定的（您可以将这里 `` == `` 读作“是”或者是“在...内”，因此第二行可以被解释为 `` X `` 是正半定的）。第三行左边是包含 `` X `` 对角元素的一个向量，该向量的每个元素值都被要求等于1。

如果这种用来表示集合成员关系的等式约束的用法依旧不理解或者只是在美学上令人不快，我们也创造了一个“伪操作符” `` <In> `` 来代替等号。举个例子，上述的半定约束可以被替换为

```matlab
X <In> semidefinite(n);
```

这与使用等式约束操作符是一样的，但是您如果觉得这种用法更赏心悦目，请随意使用。使用这样的操作符需要一些MATLAB的操作技巧，因此不要在CVX模型外使用。

集合可以在仿射表达式中合并，而且我们将一个仿射表达式约束为一个凸集。举个例子，我们可以施加下列形式的约束

```matlab
A*X*A'-X <In> B*semidefinite(n)*B';
```

其中 `` X `` 是一个 $ n\times n $ 的对称变量矩阵，而且 `` A `` 和 `` B `` 为 $ n\times n $ 的常量矩阵。这个约束要求的是对于一些 $ Y \in \mathbf{S}^n_+ $ ，使得 $ AXA^T-X=BYB^T $ 。

CVX也支持元素为数量为有序列表的集合。举例说明，考虑二阶锥或者洛伦兹锥，
$$
\mathbf{Q}^m=\{(x,y)\in\mathbf{R}^m\times\mathbf{R}|\ ||x||_2\leq y\}=\mathbf{epi}||\cdot||_2
$$
其中 $ \mathbf{epi} $ 为表示函数的题词（epigraph）。 $ \mathbf{Q}^m $ 中的一个元素为一个有两个元素的有序列表：第一个元素为一个 $ m $ 维的向量，而第二个元素为一个标量。我们使用这个锥来表示 *Least squares* 章节中简单的最小二乘问题（以相当复杂的方式）：
$$
\begin{array}{ll}
\text{minimize}& y\\
\text{subject to}& (Ax-b,y)\in \mathbf{Q}^m
\end{array}
$$
CVX使用MATLAB的元胞数组方式来模拟这种表示：

```matlab
cvx_begin
	variables x(n) y;
	minimize(y);
	subject to
		{A*x-b,y} <In> lorentz(m);
cvx_end
```

该函数调用了 `` lorentz(m) `` 并反悔了一个未命名的变量（换句话说，一对包含一个向量和一个标量），该变量被约束在长度为 $ m $ 的洛伦兹锥中。因此在这个标准中的约束要求变量对 `` {A*x-b,y} `` 在适当尺寸的洛伦兹锥中。

### 4.7、对偶变量

当一个标准的凸规划被解决的时候，其相应的对偶问题也会被解决。（在本文中，初始问题（original problem）被叫做原问题（primal problem））。每一个最优的对偶变量都与原问题的一个约束相关。而且这些最优的对偶变量可以给出有关原问题的变量信息，例如关于干扰约束的敏感性（参考 *Convex Optimization* 第五章）。为了在CVX中获得最优的对偶变量，您可以简单地声明它们，并将它们与约束联系起来。举个例子，考虑线性规划问题：
$$
\begin{array}{ll}
\text{minimize}&c^Tx\\
\text{subject to}&Ax\preceq b
\end{array}
$$
其中，变量 $x\in\mathbf{R}^n$ ，以及 $m$ 个不等式约束。为了将对偶变量 `` y `` 与线性规划中的不等式约束 $ Ax\preceq b $ 联系起来，我们使用一下的语法

```matlab
n=size(A,2);
cvx_begin
	variable x(n);
	dual variable y;
	minimize(c'*x);
	subject to
		y:A*x<=b;
cvx_end
```

 `` dual variable y `` 会告诉CVX， `` y `` 表示对偶变量，而 `` y:A*x<=b; `` 则将其与不等式约束联系在一起。注意分号 `` : `` 操作符与在标准的MATLAB用法不同，在标准的Matlab语法中是用于构建数值序列，例如 `` 1:10 `` 。这种新式的操作符只有在有对偶变量的时候才会有效，所以不用担心出现混乱。 `` y `` 的维度并没有给出，这是因为可以根据与其相关联的约束自动确定维度。举个例子，如果 $ m=20 $ ，在 `` cvx_end `` 命令之前在MATLAB命令窗口输入 `` y `` ，则会立即得到。

```matlab
y=
	cvx dual variable (20x1 vector)
```

此外，也并不需要将对偶变量放在约束的左边。举例说明，上述的对偶变量命令也可以写作

```matlab
A*x<=b:y;
```

同时，对偶变量对应的不等式约束总是非负的，这就意味着不需要改变对偶变量的值，将上述不等式约束翻转一些也是可以的，即

```matlab
b>=A*x:y;
```

两种写法的结果一致。另一方面，对于等式约束，将其左右两边互换将会使对偶值的最优值无效。

在 `` cvx_end `` 命令被处理之后，且假设成功地得到了最优值，CVX会将数值结果分别分配给 `` x `` 和 `` y `` 即最优原变量值和最优对偶变量值。对于线性规划的最优的原变量和对偶变量必须满足补充松弛条件
$$
y_i(b-Ax)_i=0,\quad i=1,\dots,m.
$$
您可以在MATLAB中用以下命令进行检查

```matlab
y.*(b-A*x)
```

然后会打印出 `` y `` 和 `` b-A*x `` 两项的乘积，该乘积应该接近于0。该行命令必须在 `` cvx_end `` 命令后被执行（这是为了将数值结果分配给 `` x `` 和 `` y `` ）。如果在CVX语段内执行该命令则会抛出一个错误，其中 `` y `` 和 `` b-A*x `` 依旧是抽象的表达式。

如果没有成功地得到最优值，因为这么问题是不可行或者是无限制的，所以 `` x `` 和 `` y `` 具有不同的值。在无限制的例子中， `` x `` 将会包含一个无限制的方向，换句话说，一个点 $ x $ 满足
$$
c^Tx=-1,\quad Ax\preceq0
$$
同时 `` y `` 将会得到一个 `` NaN `` 值，这就反映了对偶变量不可行的事实。在不可行的例子中，``x`` 充斥着 ``NaN`` 值，而 ``y`` 包含了一个无边界的对偶方向，换句话说就是一个点 ``y`` 满足
$$
b^Ty=-1,\ A^Ty=0,\quad y\succeq 0
$$
当然，原始点和对偶点和（或）方向取决于问题的结构。更多对偶信息的解释请查阅 *Convex Optimization* 等参考资料。

### 4.8、赋值和表达式持有符

任何有过C语言或者MATLAB经验的人都会明白单个等号 ``=`` 的赋值操作符和两个等号 ``==`` 操作符的区别。在CVX中这种区别也至关重要，而且CVX也会采取行动来保证赋值操作不会被错误地使用。举例说明，考虑下列代码片段：

```matlab
variable X(n,n) symmetric;
X=semidefinite(n);
```

乍一看，这个声明 ``X=semidefinite(n);`` 可能看起来像约束 ``X`` 为一个半正定的。但是由于赋值操作符的使用，``X`` 实际上是被匿名正定变量重写了。幸运的是，CVX禁止以重写的形式声明变量。当执行到 ``cvx_end`` 时，这个模型就会报出以下的错误

```matlab
??? Error using ==> cvx_end
The following cvx variable(s) have been overwritten:
	X
This is often an indication that an equality constraint was
written with one equals '=' instead of two
must be rewritten before cvx can proceed.
```

我们希望这个检查至少可以阻止一些编程语句上的错误，从而避免对您的模型造成令人沮丧的后果。

尽管存在这个错误，赋值操作还是十分有用的。因此我们鼓励您正确地使用这些操作符。举例说明，考虑以下的代码

```matlab
variables x y
z=2*x-y;
square(z)<=3;
quad_over_lin(x,z)<=1;
```

约束 ``z=2*x-y;`` 并不是一个等式约束，它是赋值操作。它将一个仿射表达式 ``2*x-y`` 的立即计算结果值存储下来，从而可以在后面的两个约束中使用。我们将 ``z`` 称之为表达式持有符去和正式声明的CVX变量区分开来。

通常这对于将计算一组表达式的值放在一个MATLAB变量中是十分有用的。不幸的是，在这种情况下，Matlab对象模型的某些技术细节可能会引起问题。考虑以下结构

```matlab
variable u(9);
x(1)=1;
for k=1:9
x(k+1)=sqrt(x(k)+u(k));
end
```

这看起来足够合理：``x`` 应该是一个向量，其中第一个值为 $1$，而后面的值是一个凹的CVX表达式。但是如果您在CVX模型中运行这个，MATLAB将会一个相当神秘的错误：

```matlab
??? The following error occurred converting from cvx to double:
Error using ==> double
Conversion to double from cvx is not possible.
```

发生这种错误的原因是因为当执行赋值命令 ``x(1)=1`` 时，MATLAB变量 ``x`` 被初始化为一个数值数组，而且MATLAB不会允许CVX对象插入到数值数组中。

解决的方法就是在赋值之前，准确地声明 ``x`` 为一个表达式持有符。出于这个目的，我们提供了关键字表达式和表达式用于声明单个或者多个表达式持有符方便之后地赋值。一旦一个表达式持有符被声明，您就可以自由地将其插入在数值和CVX表达式中。举个例子，之前的示例可以被改正为以下的形式：

```matlab
variable u(9);
expression x(10);
x(1)=1;
for k=1:9
x(k+1)=sqrt(x(k)+u(k));
end
```

CVX将会接受这个表示且不会出现错误。您可以以一种合适的方式来使用凹表达式 ``x(1),...,x(10)`` 。举例说明，您可以最大化 ``x(10)``。

一个变量对象和一个表示式对象的区别是很大的。一个变量对象包含优化变量，而且不能被重写或者在CVX标准中赋值（然而在解决问题之后，CVX将会用最优值重写优化变量）。另一方面，一个表达式对象被初始化为零，而且应该被考虑为一个临时的占位符用于存贮CVX表达式，它可以被赋值，自由地被重分配，或者在CVX标准中重写。

当然，正如我们第一个例子展示的，并不是必须要在表达式被创造或者使用之前去声明一个表达式占位符。但是这样做可以为模型提供额外的清晰度，因此我们强烈建议您这样做。
