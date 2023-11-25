# Lecture2 卷积与边缘提取(上)

## 1.卷积的定义

卷积核(滤波核),可以理解为加权平均,例如:
$$
\frac{1}{9}\begin{bmatrix}
{1}&{1}&{1}\\
{1}&{1}&{1}\\
{1}&{1}&{1}\\
\end{bmatrix}
$$
上述卷积核做了一个平滑的操作,具体上就是将一个像素点的值与周围的8个像素点做了平均。注意在具体的操作的时候实际上是将这个卷积核进行了翻转,再与周围的每个值进行加权求和操作如下操作:
$$
\begin{bmatrix}
{a}&{b}&{c}\\
{d}&{e}&{f}\\
{g}&{h}&{i}\\
\end{bmatrix} \rightarrow 
\begin{bmatrix}
{g}&{h}&{i}\\
{d}&{e}&{f}\\
{a}&{b}&{c}\\
\end{bmatrix}
$$
原因是为了满足卷积的计算公式:
$$
(f*g)[m,n] = \sum_{k,l}f[m-k,n-l]g[k,l]
$$
二维图像卷积要翻转可以理解为是对一维卷积的延伸

(**注意在深度学习中卷积核不需要翻转**)

## 2.卷积的特性

卷积具有以下的特性:

- 线性: $filter(f_1+f_2) = filter(f_1) + filter(f_2)$,两张图像求和再去卷积和分别去卷积得到的结果一致
- 平移不变性,$filter(shift(f_1)) = shift(filter(f_1))$,先对图像进行平移在进行卷积,和对图像进行卷积再进行平移得到的结果一样(任何平移不变的操作都可以用卷积核表示)
- 结合律,交换律,分配律
- Scalars factor out:$ka*b = a*kb = k*(a,b)$
- Identity :对脉冲信号进行卷积,会得到信号的本身

## 3.卷积存在的问题

- 尺度缩小问题

  由于顶点附近没有像素值,因此做卷积会使得尺寸变小。为了解决这个问题,需要对图像进行padding(深度学习常用)

- 其他填充方式

  - clip fiter(black)
  - wrap around
  - copy edge
  - reflect across edge

## 4.平滑去噪与锐化

对于噪声问题,我们理所当然就会想到用周围的像素值的平均去去噪，例如使用如下的卷积核:
$$
\frac{1}{9}\begin{bmatrix}
{1}&{1}&{1}\\
{1}&{1}&{1}\\
{1}&{1}&{1}\\
\end{bmatrix}
$$
这样的操作会去噪,但是也会损失掉敏感的边缘信息。使得图像变得模糊.(原因是平滑对这个点与周围值差异比较大的时候改变很大)

- 锐化

  考虑如下的卷积核
  $$
  \begin{bmatrix}
  {0}&{0}&{0}\\
  {0}&{2}&{0}\\
  {0}&{0}&{0}\\
  \end{bmatrix} -\frac{1}{9}\begin{bmatrix}
  {1}&{1}&{1}\\
  {1}&{1}&{1}\\
  {1}&{1}&{1}\\
  \end{bmatrix}
  $$
  以上卷积核会加强图像的边缘信息,原因如下:

  图像的边缘信息（图像的边缘信息在平滑中变化最大,当原图减去平滑图,就能够得到变化最大的边缘信息）:
  $$
  \begin{bmatrix}
  {0}&{0}&{0}\\
  {0}&{1}&{0}\\
  {0}&{0}&{0}\\
  \end{bmatrix} -\frac{1}{9}\begin{bmatrix}
  {1}&{1}&{1}\\
  {1}&{1}&{1}\\
  {1}&{1}&{1}\\
  \end{bmatrix}
  $$
  图像本身加上它的边缘信息就得到了锐化。

## 5.高斯滤波器(低通滤波器)

上述的平滑操作存在问题,平滑操作会使得图像出现振铃效果,原因是卷积过程中引入了图像没有的信息，其根源是模板的每一个值都是一样的。为了取出这个效果,提出了高斯滤波器:

- 高斯滤波器的权重由高斯函数决定,高斯函数中间大,两边小,使得该像素本身的权值大,周围的权值小,具体可以通过调整高斯函数的方差$\sigma$,进行调整
- 为了防止图像衰减或增强,权值加起来应该等于1,所以对高斯核进行了归一化操作。

高斯函数为:$G_\sigma = \frac{1}{2\pi\sigma^2}e^{-\frac{(x^2+y^2)}{2\sigma^2}}$,其存在可调参数$\sigma$,而对于高斯滤波器来说,其还可调滤波器的大小即窗宽。

- $\sigma$:越大平滑越厉害,越小滤波效果越弱
- 窗宽:越大平滑越厉害,越小滤波效果越弱
- $\sigma$与窗宽:经验上一般窗宽为$3\sigma$

注意不是越大越好,滤波效果是以损失边缘信息为代价的。

高斯核性质:

- 滤掉高频信号(low -pass)
- 一个大高斯核可以用两个小高斯核连续操作来得到
- 高斯核可以分解(分解成x方向和y方向的高斯核)

第二,三条性质可以用于加速计算,例如原计算复杂度为$O(n^2m)$,分解后的复杂度可能为$O(n^2m)$,(神经网络卷积核不保证得出的计算结果一样,但是高斯核是保证一样的,深度学习很多改进的思想,例如用小卷积核代替大卷积核)

## 6.噪声

- 椒盐噪声
- 脉冲噪声
- 高斯噪声

当高斯噪声的方差增大时,高斯滤波器的方差也要逐渐增大(会损失边缘,使信号衰减),但是其对于椒盐噪声效果不好。因此引入中值滤波,

中值滤波就是在像素模板上选中值代替原像素值,其是非线性的。同样也是模板越大,更模糊和平滑.

## 7.边缘检测

边缘主要有以下几种:

- 面上的不连续
- 深度上的不连续
- 字母边缘
- 阴影边缘

这几种边缘都各有用处,我们需要将其提取。提前边缘的最简单的方法就是求导,导数的极值点就是边缘信息。我们将求导进行了简化:
$$
\frac{\partial f(x,y)}{\partial{x}} \thickapprox \frac{f(x+1,y)-f(x,y)}{1}
$$
这个可以使用卷积核表示

同时还有其梯度:
$$
\lVert \nabla f\lVert = \sqrt{(\frac{\partial f(x)}{\partial{x}})^2+(\frac{\partial f(y)}{\partial{x}})^2}
$$
梯度的方向是垂直于边缘方向的
