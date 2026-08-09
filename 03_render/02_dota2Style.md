## 一、Dota 2 Shader 基础构成分析

 ###### 官方网站可供下载的模型贴图，大量冗余堆砌且方向混乱，某些部分正常不需要甚至不该提供的贴图却带了一个像素，有一定误导性。于是，使用贴图前，需要根据固有色贴图调整方向，再将相近范畴灰度图合并通道以减少采样，步骤繁琐，工作量较大。


### 1. Diffuse

#### 1.1 color - BaseColor & metalnessMask - Metallic

$$
C_{diffuse} \leftarrow (1-M_{metal}) \times F_{lambert} \times C_{base}
$$

* ###### 注：金属弱漫反射，强反射环境色，非金属强漫反射；(1-M)C 即 lerp(C,0,M)

#### 1.2 diffuseWarp - Diffuse Ramp

$$
F_{lambert}=LUT(max(0,N \cdot L))
$$

* ###### 注：Ramp WrapMode 为 Clamp；LookUp-Table对变量再映射，采样风格化调整数值

#### 1.3 normal - NormalMap

$$
N=M_{tbn}n_{t}
$$

* ###### 注：需设置贴图类型

### 2. Specular 

#### 2.1 specularMask - SpecularIntensity & specularExponent - Smoothness

$$
P=lerp(P_{min},P_{max},F_{smooth})
$$

$$
F_{phong}=max(0,R_{light} \cdot V)^{P}
$$

$$
C_{specular} \leftarrow M_{spec} \times F_{phong} \times I_{spec} \times C_{spec}
$$

* ###### 注：遮罩区分同一材质下不同区域特性，如是否光滑；数值插值界定最值区间（[0,1] to [min,max]），高光重点控制强度

#### 2.2 tintByBaseMask - Specular Color Mask

$$
C_{spec}=(1-M_{tint}) \times I_{tint} \times C_{base}
$$

* ###### 注：高光染色，混合固有色，插值方式多样


### 3. Ambient

#### 3.1 cubeMap - Reflection Cubemap

$$
R_{view}=2(N \cdot V)N-V
$$

$$
C_{env}=T_{cube}(R_{view})
$$

$$
C_{envSpec} \leftarrow (M_{metal}+I_{fresSpec})\times C_{env} \times C_{tint}
$$

* ###### 注：菲涅尔高光混合固有色，金属强菲涅尔高光

#### 3.2 rimMask - Rim Mask & fresnel
$$
F_{schlick}=F_0+(1-F_0)(1-N \cdot V)^5
$$

$$
I_{rim}=M_{rim} \times (1-N \cdot V)^P
$$

* ###### 注：菲涅尔效应描述光在介质界面反射角与反射强度正相关的反射特性，由Schlick近似

#### 3.3 fresnelWarp - Fresnel LUT

$$
F_{fresnel}=LUT(max(0,N \cdot V)) \times M_{metal}
$$

$$
C_{rim} \leftarrow M_{rim} \times F_{fresRim} \times N_{y}
$$

* ###### 注：Rim 轮廓光<br>1. 关于 ndotv 映射，反向非线性变化<br>2. 金属度越高，菲涅尔效应，即轮廓光越弱，故关于金属度反向插值<br>3. 限制上方轮廓光，需乘以世界法线竖直分量

### 4. Alpha

#### 4.1 translucency - Opacity Mask

$$
clip(M_{opacity}^{-})
$$

* ###### 注：0. 改自背光投射<br>1. 不透明度遮罩（默认为1），将灰度阈值外片元丢弃（AlphaClip），包括着色与投影（支持AC的ShadowCaster）；<br>   2. 要观察到透贴双面（树叶、头发、布料等）需要关闭背面剔除

### 5. Emission

#### 5.1 selflllumMask - Emission Mask

$$
C_{emission} \leftarrow M_{emis} \times C_{emis} \times I_{emis}
$$

## 二、总结

$C_{final}=C_{diffuse}+C_{specular}+C_{envDiff}+C_{envSpec}+C_{rim}+C_{emission}=F(RT)$

* ###### 注：分量大多遵循 灰度遮罩（M）- 强度（F）- 颜色（C）