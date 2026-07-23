## CG (Build-in) & URP HLSL

---

### 1. API

#### 1.1 Spatial Transformation

|Func|CG|HLSL|Note|
|:-:|:-:|:-:|:-:|
|O2C|`UnityObjectClipPos(posOS)`|`TransformObjectHClip(posOS.xyz)`|MVP|
|O2W|`mul(unity_ObjectToWorld, posOS)`|`TransformObjectToWorld(posOS.xyz)`|Interact in World(Light)|
|W2C|`mul(UNITY_MATRIX_VP, posWS)`|`TransformWorldToHClip(posWS.xyz)`||
|nDirWS|`UnityObjectToworldNormal(nDirOS)`|`TransformObjectToWorldNormal(nDirOS)`|Non-uniform Scaling (auto Inverse Transpose Matrix)|
|vDirWS|`_WorldSpaceCameraPos - posWS`|`GetWorldSpaceViewDir(posWS)`|ToNormalized<br>`GetWorldSpaceNormalizedViewDir(posWS)`|
|cameraPosWS|`_WorldSpaceCameraPos`|`GetCameraPositionWS()`||
|C2S(NDC)|`posCS.xy/posCS.w * 0.5 + 0.5`|`GetNormalizedScreenSpace(posCS)`||
|ScreenResolution|`_ScreenParams.xy`|`GetScaledScreenParams().xy`||

#### 1.2 Light

|Func|CG|HLSL|Note|
|:-:|:-:|:-:|:-:|
|**Data**|
|lDir/Col|`_WoldSpaceLightPos0.xyz`/`_LightColor0.rgb`|`GetMainLight()`|HLSL: `struct Light{half3 direction, color, distanceAttenuation, shadowAttenuation};`|
|extraLights|`#pragma multi_compile_fwdadd`|`GetAdditionalLight(uint idx, float3 posWS)`|HLSL: Return `Light`<br> Cycle with `GetAdditionalLightsCount()`|
|**Model**|
|Lambert|`max(0.0, dot(n, l)) * lCol`|`LightingLambert(lCol, lDir, nDir)`||
|Blinn-Phong|`pos(max(0.0, dot(n, h)), gloss)`|`LightingSpecular(lCol, lDir, nDir, vDir, specular, smoothness)`||
|Ambient|`UNITY_LIGHTMODEL_AMBIENT`|`SampleSH(nDir)`||

#### 1.3 AO

|Func|CG|HLSL|Note|
|:-:|:-:|:-:|:-:|
|SSAO||`SampleAmbientOcclusion(screenUV)`|(half)0~1|
|(in)directAO||`GetScreenSpaceAmbientOcclusion(screenUV)`|`struct AmbientOcclusionFactor{half indirect,direct};`|

#### 1.4 DepthTexture

|Func|CG|HLSL|Note|
|:-:|:-:|:-:|:-:|
|SampleDepth|`tex2D(_CameraDepthTexture, uv)`|`SampleSceneDepth(uv)`||
|DirectLoad|`tex2Dlod`|`LoadSceneDepth(pixelCoord)`||
|EyeDepth|`LinearEyeDepth(depth)`|`LinearEyeDepth(depth, _ZBufferParams)`||
|01Depth|`Linear01Depth(depth)`|`Linear01Depth(depth, _ZBufferParams)`||

#### 1.5 Sampler & Texture (Macros)

|Func|CG|HLSL|Note|
|:-:|:-:|:-:|:-:|
|**Declare**|
|2D|`sampler2D _MainTex;`|`TEXTURE2D(_MainTex);`<br>`SAMPLER(sampler_MainTex);`||
|CUBE|`samplerCUBE _MainTex`|`TEXTURECUBE(_Env);`<br>`SAMPLER(sampler_Env);`||
|**Sample**|
|2D|`tex2D(_MainTex, uv)`|`SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv)`||
|CUBE|`texCUBE(_Env, dir)`|`SAMPLE_TEXTURECUBE(_Env, sampler_Env, dir)`||
|LOD|`tex2Dlod(_MainTex, float4(uv, 0, lod))`|`SAMPLE_TEXTURE2D_LOD(_Tex, sampler_Tex, uv, lod)`||
|Tilling&offset|`TRANSFORM_TEX(uv, _MainTex)`|`TRANSFORM_TEX(uv, _MainTex)`|`float4 _MainTex_ST;`<br>`o.uv = i.uv * _MainTex.xy + _MainTex.zw`|

#### 1.6 Global Variables

|Variable|Value|
|:-:|:-:|
|`_Time`|`float4(t/20, t, 2t, 3t)`|
|`_SinTime`|`float4(sin(t/8), sin(t/4), sin(t/2), sin(t))`|
|`_CosTime`|`float4(cos(t/8), cos(t/4), cos(t/2), cos(t))`|
|`unity_DeltaTime`|`float4(dt, 1/dt, smootDt, 1/smoothDt)`|
|`_ProjectionParams`|`float4([FlipOrNot], [NearPlane], [FarPlane], 1/[FarPlane])`|
|`_ZBufferParams`|`float4`|
|`_ScreenParams`|`float4(width, height, 1/width, 1/height)`|

---

### 2. Math Function

|Type|Func|Note|
|:-:|:-:|:-:|
|三角|`sin`, `cos`, `tan`, `atan2`||
|指数/幂|`pow`, `exp`, `sqrt`, `rsqrt`||
|取整/小数|`floor`, `ceil`, `round`, `frac`||
|边界限制|`clamp`, `satuate`, `max`, `min`||
|插值/阶跃|`lerp`, `step`, `smoothstep`||
|向量|`dot`, `cross`, `normalize`, `length`, `distance`||
|导数|`ddx`, `ddy`, `fwidth`||
|丢弃|`clip`, `discard`||

---

### 3. Shader Language Features

#### 3.1 Semantics `语义`

|Semantic|Var|Note|
|:-:|:-:|:-:|
|**VertIn (appdata from CPU)**|
|`: POSITION`|`float4 posOS`||
|`: NORMAL`|`float3 nDirOS`|Lighting|
|`: TANGENT`|`float4 tDirOS`|normalTexture|
|**FragOut (VertIn)**|
|`: SV_POSITION`|`float4 posCS`||
|**FragOut**|
|`: SV_Target`|`float4 finalCol`||
|**VertIn/VertOut**||for Data Interpolation|
|`TEXCOORD0~7`|`float~float4`|`float2 uv[0~3] : TEXCOORD[0~3]` 4 for uv<br>`TEXCOORD0~TEXCOORD15` 16 for interpolation|
|`COLOR`|`float4 vertexCol`||

#### 3.2 Interpolation Modifiers `插值修饰符`

|Modifier|Note|
|:-:|:-:|
|`nointerpolation`||
|`centroid`|MSAA|
|`linear`|default|

#### 3.3 Preprocessor Directive - Pragmas`预编译指令`

|Directive|Note|
|:-:|:-:|
|**# pragma [shader] [shaderFuncName]**|
|`#pragma vertex vert`|vertexShaderFuncMain|
|`#pragma fragment frag`|fragmentShaderFuncMain|
|`#pragma target 3.0/4.0/5.0`|shaderVersion|
|**# pragma multi_compile**|
|`#pragma multi_compile _ _MAIN_LIGHT_SHADOWS`||
|`#pragma multi_compile _ _ADDITIONAL_LIGHTS`||

 
```cpp
c/c++
#include "MyFunc.hlsl" 
#define myGray(x) float4(x,x,x,1);
#ifdef #endif
#if #elif #else
#pragma
```

#### 3.4 SRP Batcher CBUFFER (Macros) `可编程渲染管线批处理器常量缓冲区宏`

```hlsl
//URP variables declare
CBUFFER_START(UnityPerMaterial)
    //Build-in RP uniform declare from Properties
    //[Type] [NameInProperties];
    float4 _MainTex_ST;
CBUFFER_END
```

#### 3.5 Data Type

* `float 32bit, half 16bit, fixed 8bit (onlyCG)` 
* `float3x3 matrix = float3x3(1,0,0, 0,1,0, 0,0,1);`
* `float4[.xyzw/.rgba/.xxx/.rg]`

---

### 4. Library Files

#### 4.1 CG

> `UnityCG.cginc`
> `UnityShaderVariables.cginc` **(autoIncluded)**
> `Lighting.cginc`
> `AutoLight.cginc`
> `UnityIndirect.cginc`
> `UnityInstancing.cginc`

#### 4.2 HLSL
>`#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/..."`

>`Core.hlsl`
>`Lighting.hlsl`
>`Shadows.hlsl`
>`DeclareDepthTexture.hlsl`
>`SurfaceInput.hlsl` **(PBR)**
>`Particles.hlsl`
> 
