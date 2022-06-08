# Shader Basics

## 1. Anatomy Of Mesh

- Cube is a mesh made of six sides & each side is constructed from two triangles.
- The most effiecient way to store a mesh is by using triangles. (so most models are constructed by triangles)
- `triangles` are also called polygons.
- `normals` are really important. They represent the side of the polygon that the texture should be applied to. Also used in calculating how an object is shaded by determining how lights hits the surface.

![](https://i.imgur.com/tqvhktt.png)

- surface normal
- each vertex also has its own normal.
---
- A mesh is stored as a series of arrays that stores all the info about the vertices & normals.
- They include a vertex array holding eight corner's 3D coordinates, a normal array which holds each of the vertice's normal, a uv array that specifies how a texture is mapped to a model, an array of triangles (polygon array)
- Cube - For the vertex array there are 24 entries. (only 8 vertices but each vertex is connected to 3 sides.).each vertex has a normal. so normal array length is 24.
- The UV array indicate how diffrent parts of a 2D texture mapped to each 3D vertex.
- triangle array list vertices in group of 3 where each tuple represent a single triangle.

### 1.1 UV : 

- UV represent a point on a texture that is mapped to a point on a polygon.
- UV values fall between 0-1. (no matter how big the texture is)
- Meshes can have more than one set of UVs. This can be the case when more than one image is mapped over its surface.

## 2. Render Pipeline

https://docs.unity3d.com/Manual/render-pipelines.html
https://www.youtube.com/watch?v=qHpKfrkpt4c&list=PL09X4HXJpa8kfw8cZjyYZel8WlOT5B1_k&index=21

- Rendering is the process of drawing a scene on the computer screen.
- Rendering Pipeline (graphics plotline) - the flow of process that takes a virtual environment drawn on to the computer screen.


## 3. Shader : 

- It's code that will run on GPU. Normal C# scripts, they run on CPU. GPU is one that handles graphics related computations.

https://docs.unity3d.com/Manual/SL-DataTypesAndPrecision.html

### 3.1  Shader Properties

- Properties are the way we get values from the inspector into the shader.

```
_Color("Color",Color) = (1,1,1,1) //Color Picker -> fixed4 _Color;
_Range("Range",Range(0,5)) = 1 //Range -> half _Range;
_Texture("Texture",2D) = "white" {} //Image -> sampler2D _Texture; 
_Cube("Cube",CUBE) = ""{} //Cube Map -> samplerCUBE _Cube;
_Float("float",Float) = 0.5 //multiplier -> float _Float;
_Vector("vector",Vector) = (0.5,1,1,1) // vector4 -> float4 _vector;
```

### 3.2 Variables : 

- Shader code is exected per vertex or per pixel basis.(we write the code for one pixel, for other pixels GPU will handle the looping.)
- Variables are bit different. they are designed to be efficient.


#### Basic Data types : 
1. float : like a regular c# 32 bit float. used for world pos, texture coordinates etc.
2. half : half sized float (16 bits). used for short vectors, directions & dynamic color ranges.
3. fixed : lowest precision (11 bits). used for regular colors & simple color operations.
4. int : used for counters & array indices.

#### Texture Data types : 
1. sampler2D : regular images
2. samplerCUBE : cube maps

```
//low precision

sampler2D_half
smaplerCUBE_half

//high precision

sampler2D_float
samplerCUBE_float
```

#### Packed Arrays : 
* any of these data types can be made into special array(in shaders) called packed arrays.
* To create a packed array, only requires a number representing the length of the array to be placed on the end of the data type.
* ex : int3, float3, fixed4

```
fixed4 col1 = fixed4(1,0,1,0);//r,g,b,a/x,y,z,w
col1.a = col1.w = 1

fixed3 col2 = 1; (smearing)
fixed3 col2 = fixed3(1,1,1);

cannot mix diffrent color channels (rgz)

fixed3 col2 = col1.rgb;
fixed3 col2 = col1.grb; //(swizzling/changing the order)
```

#### Packed Matrix : 
```
float4x4 mat;//4 rows,4 cols

float val = mat._m00;
fixed4 col1 = mat._m00_m01_m02_m03;//returns an array of 4 values

fixed4 col2 = mat[0];//entire row

```

## 4.  Lighting

### Lambert Lighting (diffuse lighting)

https://medium.com/shader-coding-in-unity-from-a-to-z/light-in-computer-graphics-be438e13522f

- It's a nice & simple model that creates adequate effect & quite fast rendering given it's lack of complexity.
- only uses surface normal & source vector for calculations.
- Not very good at genarating reflections & highlights from light source.

## 4. Vertex & Fragment Shader

* The vertex func gives us access to each vertex in the model. (can set its color or pos)
* The frag func provide per pixel coloring. Also can access pixels relative to world positions.

![](https://i.imgur.com/5ZhDZCK.png)

* Material - It can set properties of the shader.
* frag func is expensive as it will run for each vertex. Use vertex func as much as possible.
* In VF shaders, we have to handle light by ourselves. In surface shaders it's easy.

### 4.1 Structure

```
#pragma vertex vert //vertex func name
#pragma fragment frag //frag func name

#include "UnityCG.cginc" // contains all the prewritten methods and variables

//will be passed to the vertex func
struct appdata
{
    float4 vertex : POSITION;
    float2 uv : TEXCOORD0;
    float3 normal : NORMAL;
    fixed4 color : COLOR:
    float4 tangent : TANGENT;
};

//will be passed to the frag func
//interpolators
struct v2f
{
    float4 vertex : SV_POSITION;
    float2 uv : TEXCOORD0;
    float3 normal : NORMAL;
    fixed4 color : COLOR;
    float4 tangent : TANGENT;
};

//will run for each vertex
v2f vert (appdata v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}

//will run for each frag(pixel)
//needs to return a color
fixed4 frag (v2f i) : SV_Target
{
    fixed4 col = tex2D(_MainTex, i.uv);
    return col;
}
```
### 4.2 Interpolators : 

* In `v2f` struct, everything is called interpolators. 
* Actually we have mesh data only per each vertex. (pos, normal, uv, color etc)

![](https://i.imgur.com/FUdpD5I.png)
* But for the `frag` func, it needs data for evety pixel. So that means it needs data of the red dots. But they don't come with the model. What happens these values will be caluculated by interpolation of exisiting values. (barycentric interpolation)

![](https://i.imgur.com/vOCaGNP.png)

* When you return the vertex color, you will get above images. So what happen here is middle pixel color values is blending between vertex color values.
* This interpolation has a side effects. 

![](https://i.imgur.com/h7Mg1sf.png)

* here vertex normal are normalized vectors. But interpolated normal will not be normalized vector. (vector of length 1). So you will have to manually do that.


## 5. CG Functions

### 5.1 UnityObjectToClipPos : 

* Model data is in object space(local space). This method transform a point from object space to camera's cilp space. (convert from 3D space to 2D space)
* Clip space - It's a space relative to the camera

### 5.2 tex2D :

* use texture & UV value to get the color of a pixel.

### 5.4 normalize : 

### 5.3 dot : 

### 5.4 saturate : 
* returns a value between 0-1.

```
float x = 0.5;
saturate(0.5)
```

### 5.5 max : 

### 5.5 clip :

### 5.5 step : 