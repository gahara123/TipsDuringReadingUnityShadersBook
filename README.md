# TipsDuringReadingUnityShadersBook
## 在学习《UnityShader入门精要》的一个笔记
### 一.13.2节中，缺乏对深度纹理采样后的深度值如何得到世界空间下的坐标表示的解释
对应代码如下

```shaderlab
//深度纹理采样，本质是返回纹理采样结果的r通道  
float d = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv_depth);
//获取投影空间下的坐标（数值范围在-1~1）  
float4 H = float4(i.uv.x * 2 - 1, i.uv.y * 2 - 1, d * 2 - 1, 1);  
// 通过视角*投影矩阵的逆矩阵，得到视角空间下的坐标  
float4 D = mul(_CurrentViewProjectionInverseMatrix, H);  
// 除以w分量，将其转为世界坐标  
float4 worldPos = D / D.w;
```

令世界坐标系中有一点P，齐次坐标记为$` v_{world}=(x,y,z,w)^T `$,其中$`v_{world}.w=1.0`$, 则有  
<br/>
$` v_{clip} = M_{proj}*M_{view}*v_{world}`$&emsp;&emsp;&emsp;&emsp;(1)  
<br/>
其中，$` v_{clip} `$表示P在裁剪空间中的坐标，$`M_{view} `$表示视角矩阵，$`M_{proj} `$表示投影矩阵，将上述两个矩阵相乘得到当前的观察投影矩阵$`M_{vp} `$表示观察矩阵，则有  
<br/>
$` v_{world} = M_{vp}^{-1}*v_{clip}`$&emsp;&emsp;&emsp;&emsp;(2)  
<br/>
P点的深度纹理采样结果为$`d`$，P点的NDC坐标记为$` v_{ndc} `$，根据Unity的深度纹理的存储规则，d是NDC映射而来，使用原映射的反函数即可，则有  
<br/>
$` v_{ndc} = d*2-1`$&emsp;&emsp;&emsp;&emsp;(3)  
<br/>
透视除法的过程是裁剪空间坐标的4个分量分别除以w分量，即
<br/>
$` v_{ndc} = v_{clip}/v_{clip}.w`$&emsp;&emsp;&emsp;&emsp;(4)  
<br/>
由公式(2)和公式(3)得  
<br/>
$` v_{world} = M_{vp}^{-1}*v_{ndc}*v_{clip}.w`$&emsp;&emsp;&emsp;&emsp;(5)  
<br/>
在公式(5)中，$`M_{vp}^{-1} `$由`_CurrentViewProjectionInverseMatrix`提供，$` v_{ndc} `$由$`d`$得到，$`v_{world}.w=1.0`$，则有  
<br/>
$` v_{world}.w = (M_{vp}^{-1}*v_{ndc}).w*v_{clip}.w = 1.0`$&emsp;&emsp;&emsp;&emsp;(6)  
<br/>
由公式(5)可得  
<br/>
$` v_{clip}.w = \frac{1,0}{(M_{vp}^{-1}*v_{ndc}).w} `$&emsp;&emsp;&emsp;&emsp;(7)  
<br/>
将公式(7)带入公式(5)，则有  
<br/>
$` v_{world} = \frac{M_{vp}^{-1}*v_{ndc}}{(M_{vp}^{-1}*v_{ndc}).w}`$  
<br/>
且由公式(3)，则可知上文H为$` v_{ndc} `$,D为$`{(M_{vp}^{-1}*v_{ndc})} `$,则世界坐标可由D / D.w得到



