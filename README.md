# Anime-Style Character Rendering in Unity URP

基于 Unity URP 与 HLSL 实现的二次元角色渲染练习，重点研究 Ramp 阴影、SDF 面部阴影、Lightmap 通道控制、金属 MatCap、高光、边缘光与描边。

> A study project for anime-style character rendering in Unity URP, covering ramp-based shading, SDF face shadows, packed lightmaps, stylized specular, MatCap reflections, rim lighting, and outlines.

<p align="center">
  <img src="docs/images/hero.jpg" alt="Anime-style character shader final result" width="100%">
</p>

## 效果对比

| 原始材质 | 最终效果 |
| --- | --- |
| <img src="docs/images/before.jpg" alt="Original material" width="100%"> | <img src="docs/images/hero.jpg" alt="Final toon shader result" width="100%"> |

## 主要功能

- 基于 Ramp Texture 的日间/夜间分层阴影
- 基于头部方向与 SDF 贴图的面部阴影
- Lightmap RGBA 通道驱动的 AO、材质分区和高光控制
- 卡通化 Blinn–Phong 普通高光与金属高光
- 基于观察空间法线的金属 MatCap 反射
- 主光源、附加灯与实时阴影支持
- 方向性 Rim Light
- 基于背面扩张的角色描边
- 面部阴影遮罩与可调腮红
- 独立 ShadowCaster Pass

## 渲染流程

```text
Base Color
    ↓
Lightmap + Main/Additional Lights
    ↓
Ramp Shadow / SDF Face Shadow
    ↓
Regular Specular / Metal MatCap
    ↓
Rim Light + Outline
    ↓
Final Character Color
```

### Body Lightmap 通道

Body Shader 使用一张 RGBA Lightmap 打包多种材质控制数据：

| 通道 | 当前用途 |
| --- | --- |
| R | 普通高光强度数据，并通过高值区域识别金属/MatCap 区域 |
| G | AO 与阴影区域控制 |
| B | 普通高光出现范围与阈值控制 |
| A | 材质 ID，用于选择不同的 Ramp 行 |

### MatCap 与实时高光

MatCap 不直接替代全部实时高光。当前实现中，MatCap 使用观察空间法线采样，用于表现稳定的大范围金属反射；Blinn–Phong 高光继续响应实时灯光，用于提供局部亮点。Lightmap R 的高值区域控制两者之间的材质分区。

### SDF 面部阴影

`SetFaceMaterialVector.cs` 将角色头部的 Forward、Right 和 Up 方向传入面部材质。Face Shader 根据主光方向选择并翻转 SDF 采样，从而生成随光源方向变化、但保持美术可控的面部阴影。

## 技术环境

| 项目 | 版本/说明 |
| --- | --- |
| Unity | 2022.3.62f3 LTS |
| Render Pipeline | Universal Render Pipeline 14.0.12 |
| Shader | ShaderLab / HLSL |
| Rendering Path | URP Forward Rendering |
| Asset Management | Git LFS |

## 项目结构

```text
Assets/Cel-Shaded/
├─ Shaders/
│  ├─ ToonShader_Body.shader
│  └─ ToonShader_Face.shader
├─ Scripts/
│  └─ SetFaceMaterialVector.cs
├─ Scenes/
│  └─ Scene_Cel-Shading.unity
└─ Lumine/
   ├─ materials/
   ├─ textures/
   └─ Lumine FBX.fbx
```

- [Body Shader](Assets/Cel-Shaded/Shaders/ToonShader_Body.shader)
- [Face Shader](Assets/Cel-Shaded/Shaders/ToonShader_Face.shader)
- [面部方向控制脚本](Assets/Cel-Shaded/Scripts/SetFaceMaterialVector.cs)
- [演示场景](Assets/Cel-Shaded/Scenes/Scene_Cel-Shading.unity)

## 运行项目

1. 安装 [Git LFS](https://git-lfs.com/) 并初始化：

   ```bash
   git lfs install
   ```

2. 克隆仓库：

   ```bash
   git clone https://github.com/MaiXiang0929/Cel_Shader.git
   ```

3. 使用 Unity Hub 通过 **Unity 2022.3.62f3 LTS** 打开项目。
4. 打开 `Assets/Cel-Shaded/Scenes/Scene_Cel-Shading.unity`。
5. 在 Body 与 Face 材质面板中调节阴影、高光、MatCap、Rim Light 和 Outline 参数。

首次打开时，Unity 需要导入模型、贴图并编译 Shader，耗时取决于本机配置。

## 常用参数

| 参数 | 作用 |
| --- | --- |
| Shadow Position / Softness | 控制明暗分界位置与过渡宽度 |
| Shadow Ramp Width | 控制 Ramp 采样范围 |
| Shininess / Specular Threshold | 控制普通高光集中度与大小 |
| LightMap B Specular Area | 控制贴图中 B 通道对普通高光范围的影响 |
| MatCap Blend / Intensity | 控制金属反射混合比例与亮度 |
| Metal Mask Threshold | 从 Lightmap R 中划分金属区域 |
| Rim Power / Intensity | 控制边缘光宽度与强度 |
| Outline Width | 控制背面扩张描边宽度 |

## 已知限制

- 当前实现主要面向桌面端 URP，尚未针对移动平台完成性能验证。
- SDF 面部阴影依赖正确配置的头部方向辅助节点。
- MatCap 当前仅在指定金属材质上启用，其他材质需要按实际 Lightmap 数据单独配置。
- 项目用于渲染技术研究，并非对游戏原始 Shader 的完整复刻。

## 版权说明

本仓库中由作者编写的代码按照 [MIT License](LICENSE) 发布。

角色、模型、贴图及相关美术资源的版权归其原始权利人所有。本项目仅用于个人学习、技术研究和作品集展示，与 HoYoverse 官方无关；第三方美术资源不包含在本仓库的 MIT 授权范围内。

更多作品：[MaiX Portfolio](https://maix-portfolio.vercel.app/)
