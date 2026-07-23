## ShaderLab

---

### 1. Frame

```shaderlab
Shader "[Path/Name]"{
    Properties{
        [Property Definition]
    }
    SubShader{
        [Tags]
        [PublicRenderSetup]
        Pass{
            [Name]
            [Tags]
            [RenderSetup]
            [CG/HLSL]
        }
    }
}
```

---

### 2. Properties

#### 2.1 Definition `属性定义`

|Type|Example|Note|
|:-:|:-:|:-:|
||**`[Attribute] Name("InspectText", Type) = Default`**|
|**valueType**|
|Float|`_Name("Text", Float) = 0.0`|32bit float|
|Range|`_Name("Text", Range(0, 1)) = 0.0`|float slider|
|Int|`_Name("Text", Int) = 1`|float|
|Integer|`_Name("Text", Integer) = 1`|int|
|**struct**|
|Vector|`_Name("Text", Vector) = (1, 1, 1, 1)`|float4|
|Color|`_Name("Text", Color) = (1, 1, 1, 1)`|float4 Color Picker|
|**ReferrenceType**|
|2D|`_Name("Text", 2D) = "white"{}`|sampler2D (SAMPLER + TEXTRUE2D)<br>`"color"{texture}` color: white/black/bump/red|
|Cube|`_Name("Text", Cube) = ""{}`|samplerCUBE|
|3D|`_Name("Text", 3D) = ""{}`||
|2DArray|`_Name("Text", 2DArray) = ""{}`||

>* Serialization 
>* RenderSetup:
` Cull [_CullMode]`

#### 2.2 Property Drawer `属性绘制器`

|Attribute|Note|
|:-:|:-:|
|**`[Attribute]`**|
|`[Space()]`|Spacing|
|`[Header()]`|Title|
|`[Toggle]`|Float|
|`[Toggle(ENABLE)]`|`#ifdef ENABLE #else #endif`|
|`[PowerSlider(3.0)]`|Range|
|`[IntRange]`|Range|
|`[KeyWordEnum(StateOff, State0, etc)]`|`#ifd _Name_State0 #elif _Name_State1`|
|`[Enum(UnityEngine.Rendering.CullMode)]`||
|`[Enum(Off, 0, Front, 1, Back, 2)]`|`Cull [_Name]`|
|`[HideInInspector]`||
|`[PerRendererData]`||

### 2.3. Variant
|Pragma|Variant|Note|
|:-:|:-:|:-:|
|`#pragma shader_feature`|||
|`#pragma multi_compile`|`__ Var1 Var2 ...`|`_local`|

---

### 3. SubShader

#### 3.1 LOD

* ON **first** under `Shader.Find([Path/Name]).maximumLOD`
* `Fallback "Path/Name"/Off` when **none**

#### 3.2 SubShader Tags

|`Queue`|Index (RenderSequence)|Note|
|:-:|:-:|:-:|
|`Background`|1000|Skybox|
|`Geometry`|2000|Opaque(<=2500) near2far|
|`AlphaTest`|2450|Alpha|
|`Transparent`|3000|far2near|
|`Overlay`|4000|UI|

|`RenderType`|Note|
|:-:|:-:|
|`Background`||
|`Opaque`||
|`Transparent`||
|`TransparentCutout`||
|`Overlay`||


|OtherTagKey|Value|Note|
|:-:|:-:|:-:|
|`RenderPipeline`|`UniversalPipeline`/`HDRenderPipeline`||
|`DisableBatching`|`True`||
|`ForceNoShadowCasting`|`True`||
|`CanUseSpriteAtlas`|`True`||
|`PreivewType`|`Shpere`/`Plane`/`Skybox`||

---

### 4. Pass `drawcall`

#### 4.1 Pass Tags

`"LightMode" = "ForwardBase"/"ForwardAdd"/"UniversalForward"/"ShadowCaster"`

#### 4.2 RenderSetup

|Command|Type|Note|
|:-:|:-:|:-:|
|`Cull`|`Back`/`Front`/`Off`||
|`ZTest`|`Less`/`Greater`/`LEqual`/`GEqual`/`Equal`/`NotEqual`/`Always`||
|`ZWrite`|`On`/`Off`|Off when Alpha|
|`Offset`|offsetFactor, offsetUnits|Z-Fighting|
|`Blend`|`SrcAlpha OneMinusSrcAlpha`/`One One`||
|`BlendOp`|`Add`/`Sub`/`RevSub`/`Min`/`Max`||
|`ColorMask`|R G B A 0||
|`AlphaToMask`|`On`/`Off`|MSAA Alpha Test|