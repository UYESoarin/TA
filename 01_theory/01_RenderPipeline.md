# 渲染管线


---

### 1. CPU
#### 1.1 Application

确定渲染对象与顺序，准备数据与渲染

* **剔除 `Culling`**
    1. 视锥剔除 `Frustum Culiing`：AABB包围盒`Axis-Aligned Bounding Box`判断与视锥碰撞
    2. 遮挡剔除 `Occlusion Culling`
    3. 层级剔除 `Layer Culling`

* **排序 `Sort`**
渲染队列 `RenderQueue`
    1. 不透明队列**自前而后**（RenderQueue < 2500） 
    2. 透明队列**自后而前**（RenderQueue > 2500）
* **动态合批**

* **打包数据 - 3D vertices input**
模型代码：顶点坐标、法线、UV、切线、顶点色、索引列表
变换矩阵、灯光、材质参数、磁盘图片
* **提交 SetPassCall & DrawCall**


---

### 2. GPU
#### 2.1 Vertex Shader Processing

处理CPU数据，顶点变换到裁剪空间

* **MVP变换**
    1. 模型空间`Object Space`
    **Model** Matrix
    2. 世界空间`World Space`
    **Veiw** Matrix
    3. 相机空间`View Space`
    **Projection** Matrix
    4. 裁剪空间`Clip Space`
    (x,y,z,w)

* **自定义数据**
位置、法线、uv

* 注：暂无几何着色器（Geometry：三角形外部加顶点）、曲面细分着色器（Tensselation：三角形内部加顶点）


#### 2.2 Triangle Processing & Rasterization

映射屏幕空间，点连成图元，图元光栅化得片元与插值数据

* 裁剪剔除：剔除裁剪空间外的面
* 标准化设备坐标 NDC：透视除法，视口归一化便于适配分辨率(视锥体坍缩为立方体)
* 背面剔除：根据法线方向
* 屏幕坐标：视口转换
* 图元装配：顶点连线
* 光栅化：分配三角面与内部片元


#### 2.3. Fragment Shader Processing

根据已有数据对片元颜色处理输出

* 纹理着色
    * 纹理采样：贴图uv映射与采样器
    * 纹理过滤 `Filter Mode` ：Point(nearest), Bilinear, Trilinear(mipmap)
    * Mip Map：多级平均，额外1/3内存
    * 各向异性过滤`AnisoLevel`；解决表面与相机大角度下屏幕渲染片元比失衡导致细节错误
    * 纹理寻址 `Wrap Mode` ：Clamp, Repeat, Mirror, MirrorOnce
    * 纹理压缩：选择平台（PC：DXT，安卓/苹果：ASTC、ETC）

* 光照着色
    * 光照组成：
        直接光、间接光
        漫反射 `Diffuse` 、镜面反射 `Specular` 、环境 `Ambient`
    * 光照模型 `Lighting Model`


---

### 3. Merge & Output Image(pixels)

颜色遮挡与透明混合

* 帧缓冲区`Frame Buffer Operations`：颜色、深度、模板缓冲
* 透明度测试 `Alpha Test`
* 深度测试`Depth Test`
`EarlyZ`片元着色前优化，`LateZ`片元着色后判断
* 深度写入`Depth Write`
实心写入（由近及远），半透明不写入（由远及近）
* 颜色混合`Blend`：透明重叠插值 Src * Alpha + Dst * OneMinusAlpha

---

### 4. 后处理
