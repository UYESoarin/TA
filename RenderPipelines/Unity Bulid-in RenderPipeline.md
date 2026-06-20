# 渲染管线


### CPU Application —— 确定渲染对象与顺序
---
* **剔除 `Culling`**
视锥剔除 `Frustum Culiing`：包围盒判断与视锥碰撞
遮挡剔除 `Occlusion Culling`
层级剔除 `Layer Culling`

* **排序 `Sort`**
渲染队列 `RenderQueue`
    1. 不透明队列**自前而后**（RenderQueue < 2500） 
    2. 透明队列**自后而前**（RenderQueue > 2500）

* **打包数据 - 3D vertices input**
模型：顶点坐标、法线、UV、切线、顶点色、索引列表
变换矩阵、灯光、材质参数
* **提交 Draw call**


### Vertex Shader Processing —— 顶点变换到屏幕空间
---
* **MVP变换**
    1. 模型空间`Object Space`
    ↓ **Model** Matrix
    2. 世界空间`World Space`
    ↓ **Veiw** Matrix
    3. 相机空间`View Space`
    ↓ **Projection** Matrix
    4. 裁剪空间`Clip Space`
* **自定义数据**


### Triangle Processing & Rasterization —— 点连成图元，图元光栅化得片元
---
* 裁剪剔除：剔除裁剪空间外的面
* 标准化设备坐标 NDC：透视除法，视口归一化便于适配分辨率
* 背面剔除：根据法线方向
* 屏幕坐标：视口转换
* 图元装配：顶点连线
* 光栅化：分配三角面与内部片元


### Fragment Shader Processing —— 上色
---
* 纹理着色
    * 纹理采样：UV 映射与采样
    * 纹理过滤`Filter Mode`：插值M
    * Mip Map：多级平均
    * 纹理寻址`Wrap Mode`：放空、拉伸、重复、镜像重复
    * 纹理压缩：根据平台选择
* 光照着色
    * 光照组成：
        直接光、间接光
        漫反射`Diffuse`、镜面反射`Specular`、环境光`Ambient`
    * 光照模型：
        * Phong
        $$
        max(n·l,0) + pow(max(r·v,0),smoothness) + ambient$$


### Merge & Output image(pixels) —— 颜色遮挡与透明混合
---
* 帧缓冲区`Frame Buffer Operations`：颜色、深度、模板缓冲
* Alpha测试
* 深度测试`Depth Test`：`Early-Z`为顶点着色提前深度测试
* 深度写入`Depth Write`
* 颜色混合`Blending`：透明重叠插值

### 后处理
---

### Computer Graphic Math Principle
---
#### Vector multiplication in Matrix form
* Dot product
$$
\begin{pmatrix} x_1&y_1&z_1\end{pmatrix}\cdot\begin{pmatrix}x_2\\ y_2\\ z_2\end{pmatrix}=x_1x_2+y_1y_2+z_1z_2
$$
---
* Cross product
$$
\begin{pmatrix} x_1&y_1&z_1\end{pmatrix}\times\begin{pmatrix}x_2\\ y_2\\ z_2\end{pmatrix}=\begin{pmatrix}0&-z_1&y_1\\z_1&0&-x_1\\-y_1&x_1&0\end{pmatrix}\begin{pmatrix}x_2\\ y_2\\ z_2\end{pmatrix}
$$
---
* Rotation around an axis Matrix
$$
R_x(\alpha)=\begin{pmatrix}1&0&0&0\\0&\cos{\alpha}&-\sin{\alpha}&0\\0&\sin{\alpha}&\cos{\alpha}&0\\0&0&0&1\end{pmatrix}
$$
$$
R_y(\alpha)=\begin{pmatrix}\cos{\alpha}&0&\sin{\alpha}&0\\0&1&0&0\\-\sin{\alpha}&0&\cos{\alpha}&0\\0&0&0&1\end{pmatrix}
$$
$$
R_x(\alpha)=\begin{pmatrix}\cos{\alpha}&-\sin{\alpha}&0&0\\\sin{\alpha}&\cos{\alpha}&0&0\\0&0&1&0\\0&0&0&1\end{pmatrix}
$$
---
* View/Camera Transformation(Position to Zero, up to y, forward to -z)
$$
ViewMatrix=RT=\begin{bmatrix}x_{right}&y_{right}&z_{right}&0\\x_{up}&y_{up}&z_{up}&0\\x_{back}&y_{back}&z_{back}&0\\0&0&0&1\end{bmatrix}\begin{bmatrix}1&0&0&-x_c\\0&1&0&-y_c\\0&0&1&-z_c\\0&0&0&1\end{bmatrix}
$$
$注意，R由正交性，逆向转置求解$
---
* Orthographic/Perspective Projection 略
---
* **Lambertian(Diffuse)**
$$
L_d=k_d(I/r^2)max(n\cdot l,0)
$$
$其中，n为面单位法向量、l为入射光单位方向向量$
---
* **Specular Term(Blinn-Phong)**
$$
L_s=k_s(I/r^2)max(n\cdot h,0)^p
$$
$其中，h为入射光方向向量 l，与视线观察向量 v 的单位半角向量，能一定程度反映观察方向与镜面反射方向夹角$
---
* **Ambient Term**
$$L_a=k_aI_a$$
---
* **Blinn-Phong Reflection Model**
$$
L=L_a+L_d+L_s
$$
* Barycentric coordinates`重心坐标`
$$
(x,y)=\alpha A+\beta B+\gamma C\\
\alpha+\beta+\gamma=1\\
其中，A，B，C为三角形的顶点坐标，若系数皆非负，则坐标在三角形内\\
推广：V=V_A+V_B+V_C\\
对内部插值参数颜色、坐标、法线、深度、材质
$$
---
* Bilinear interpolation 略
* Mip Map Level Computing
* Displacement mapping
* Constuctive Solid Geometry
* Catmull-Clark Subdivision
* Spatial Patitioning 
    * KD-Tree
    * Oct-Tree
    * BSP-Tree
---
* The Plenoptic Function
$$
P(\theta,\phi,\lambda,t,V_x,V_y,V_z)
$$