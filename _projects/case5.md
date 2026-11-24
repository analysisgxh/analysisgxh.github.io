---
layout: page
title: "Case 5: 火箭 1 km / Ma 0.3 GPU 气动仿真"
description: 60° 气动舵面（model_v1_60.stl）AMReX-CNS GPU 算例运行方案、AMR 网格加密策略与初始流场模式整理
img: assets/img/case5/summary/plt07323.png
importance: 0
category: work
pretty_table: true
---

本页整理自火箭气动数值模拟项目 **Case 5**（`case5_1km_M03_f60_gpu_success60_full`）的算例配置与运行记录，涵盖 GPU 完整算例启动监控、自适应网格加密（AMR）策略、v4 `common.inputs` 分区组合加密机制以及初始流场速度模式（`prob.init_mode`）说明，用于组会交流与演示。

---

## 一、算例概况与几何构型

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/case5_stl_views.png" title="火箭不同舵面展开角 STL 几何构型（0° / 30° / 60° / 90°）" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  火箭气动减速舵面四种展开角 STL 模型对比（点击可放大查看）。本次 Case 5 GPU 算例采用 60° 展开角构型 <code>model_v1_60.stl</code>。
</div>

| 项目 | 参数 / 数值 | 说明 |
| :--- | :--- | :--- |
| **飞行工况** | 高度 $$ 1\ \text{km} $$，$$ \text{Ma} = 0.3 $$ | 低空亚声速着陆段 |
| **来流参数（CGS）** | $$ p_\infty = 8.98 \times 10^5\ \text{dyn/cm}^2 $$，$$ T_\infty = 281.0\ \text{K} $$ | 约 $$ 89.8\ \text{kPa} $$，$$ U_\infty \approx 101\ \text{m/s} $$，来流方向 `+y` |
| **几何模型** | `geometry/model_v1_60.stl`（缩放 `0.1`，中心 `(0, 40.91, 0)` cm） | 60° 舵面 STL，AMReX 嵌入式边界（EB）建模 |
| **计算域范围** | $$ x, z \in [-1024,\, 1024]\ \text{cm} $$，$$ y \in [-1024,\, 2048]\ \text{cm} $$ | 粗网格 `64 × 96 × 64`（L0 = 32 cm） |
| **求解器与格式** | 无粘可压缩欧拉（$$ \gamma = 1.4 $$，$$ \mu = 28.97 $$），CFL = `0.3` | 2 阶 PLM（`plm_theta = 1.0` minmod）+ 小切割单元修复 |
| **参考面积与时长** | $$ A_{\text{ref}} = 31415.93\ \text{cm}^2 $$，目标时长 `STOP_TIME = 1.5 s` | 对应 $$ \pi \times (100\ \text{cm})^2 $$，输出 `forces.csv` 与 `plt` 流场 |

---

## 二、GPU 完整算例运行与日志监控

使用 `tmux` 保持后台会话，在单卡 GPU 上运行完整 1.5 s 仿真：

```bash
# 1. 新建 tmux 会话
tmux new -s case5_full

# 2. 进入项目目录、检查 GPU 状态并启动完整算例
cd "$HOME/XiaohuiGao/rocket_cns_v4_gpu/rocket_cns_v4"
nvidia-smi
TAG=gpu_success60_full STOP_TIME=1.5 CUDA_VISIBLE_DEVICES=0 bash scripts/run_case5_success_gpu.sh
```

终端会实时打印计算进度，同时将完整输出保存至：

```text
results/case5_1km_M03_f60_gpu_success60_full/run.log
```

**常用重连与监控命令**：

```bash
# 重新进入后台计算会话
tmux attach -t case5_full

# 在另一个终端实时跟踪最新 30 行日志
tail -n 30 -F "$HOME/XiaohuiGao/rocket_cns_v4_gpu/rocket_cns_v4/results/case5_1km_M03_f60_gpu_success60_full/run.log"
```

---

## 三、Case 5 GPU 算例的网格加密与分块配置

本次 GPU 独立模板（`cases/case5_1km_M03_f60_gpu_template/inputs`）采用 **4 层自适应网格（L0–L3：32 / 16 / 8 / 4 cm）**，通过 **表面切割单元加密 + 密度差判据 + 固定加密盒** 三者联合控制局部加密。

### 1. 区域加密核心参数

| 参数 | 当前配置 | 作用说明 |
| :--- | :--- | :--- |
| `cns.refine_cutcells` | `1` | 开启火箭表面切割单元（cut-cell）加密 |
| `cns.refine_cutcells_max_lev` | `3` | 火箭表面最高可加密到 **L3（4 cm）** |
| `cns.refine_max_dengrad_lev` | `3` | 密度变化判据可触发加密至 **L3（4 cm）** |
| `cns.refine_dengrad` | `1.1e-5` | 密度差阈值，越小越容易触发加密 |
| `cns.refine_box_lo_0` / `hi_0` | `(-256, -192, -256)` → `(256, 1280, 256)` cm | 指定固定加密区域，覆盖箭体与尾流核心区 |

### 2. 固定加密区域与网格分块控制

```ini
# 固定加密区（单位：cm）
cns.refine_box_lo_0 = -256. -192. -256.
cns.refine_box_hi_0 =  256. 1280.  256.

# 网格分块与缓冲层控制
amr.max_grid_size = 32
amr.n_error_buf   = 2 2 2 2
amr.grid_eff      = 0.7
```

---

## 四、v4 通用框架（`common.inputs`）分区组合加密策略对照

在项目的 v4 通用配置文件 `cases/common.inputs`（L0 = 16 cm，最高 L4 = 1 cm / L5 = 0.5 cm）中，**关闭了密度差加密**（`cns.refine_max_dengrad_lev = 0`），改用 **表面加密 + 固定区域加密 + 激波判据加密** 三种方式组合：

### ① 火箭表面加密

```ini
cns.refine_cutcells = 1
cns.refine_cutcells_max_lev = 2
```

识别被 STL 表面切割的单元并在附近加密：全局表面加密到 **L2（4 cm）**，鼻尖与鼻部区域另有更细要求。

### ② 固定区域加密

不需要流场出现变化，指定盒子内部即强制加密（盒子范围由各自的 `.lo`、`.hi` 指定，单位为 cm）：

```ini
cns.zone.tip.fill_lev  = 4
cns.zone.nose.fill_lev = 3
```

* **`tip`（鼻尖盒）**：盒内加密到 **L4（1 cm）**。
* **`nose`（鼻部盒）**：盒内加密到 **L3（2 cm）**。

### ③ 激波判据加密

```ini
cns.refine_shock_max_lev = 1
cns.refine_shock_pjump   = 0.2
cns.refine_shock_start   = 2.5e-3
```

从 `0.0025 s` 开始，对**同时满足**以下两个条件的单元进行加密：

1. 相邻单元相对压力差达到阈值 `0.2`；
2. 局部流动处于压缩状态。

区域外满足激波判据的位置最高加密到 **L1（8 cm）**；鼻尖与鼻部区域允许更细加密。

### 三种判据组合汇总表

| 区域 | 整个盒内 `fill_lev` | 表面 `cut_lev` | 满足激波判据 `shock_lev` |
| :--- | :--- | :--- | :--- |
| **鼻尖 `tip`** | L4（1 cm） | 配置 L5，当前限于 L4（1 cm） | 配置 L5，当前限于 L4（1 cm） |
| **鼻部 `nose`** | L3（2 cm） | L4（1 cm） | L4（1 cm） |
| **两个区域之外** | 不强制整片加密 | L2（4 cm） | L1（8 cm） |

> **补充说明**：
> * 密度差加密当前关闭：`cns.refine_max_dengrad_lev = 0`。
> * `amr.regrid_int = 4 4 4 4 4` 仅控制每隔多少时间步重新执行加密判定并更新网格，不改变加密判据本身。

---

## 五、初始流场速度模式（`prob.init_mode`）

`prob.init_mode` 是本项目自定义的参数（非 AMReX 通用参数），用于控制仿真开始时计算域内气体的初始速度分布：

| 取值 | 初始状态 | 含义 |
| :---: | :--- | :--- |
| **`0`** | **全域自由来流** | 所有流体单元一开始即以 $$ U_\infty $$ 沿来流方向运动 |
| **`1`** | **全域静止** | 域内初始速度为 `0`，之后由入口来流推动流场发展 |
| **`2`** | **分区速度** | 分界面上游为 $$ U_\infty $$，下游为 `init_vel_frac` $$ \times\ U_\infty $$ |

> **Case 5 选型要点**：Case 5 属于低速亚声速工况（$$ \text{Ma} = 0.3 $$），采用 **`prob.init_mode = 0`（全域自由来流启动）**。前期测试表明，亚声速下若采用 `init_mode = 1`（静止启动），入口碰撞波无法被推向下游，会使计算域长期停留在错误来流状态（约 $$ \text{Ma} = 0.15,\ p \approx 1.23\,p_\infty $$），因此 Case 5 必须以均匀来流初始化。

---

## 六、后处理与可视化脚本速查

算例运行完成后，可使用项目自带的后处理脚本生成组会所需的流场总览图与动态演化视频：

| 脚本 | 主要用途 |
| :--- | :--- |
| `tools/plot_case.py` | 生成算例总览图：马赫数、压力 $$ \log_{10}(p/p_\infty) $$、温度及 $$ C_d $$ 历史曲线 |
| `tools/post/make_frames.py` | 按时间步批量输出普通流场帧图，可用于合成动画视频 |
| `tools/post/make_optical.py` | 生成光学（纹影等）风格的流场帧图 |

生成 Case 5 缩放总览图示例：

```bash
python3 tools/plot_case.py results/case5_1km_M03_f60_gpu_success60_full \
  --zoom 0.8 -o results/case5_1km_M03_f60_gpu_success60_full/summary_zoom.png
```

---

## 七、算例结果展示（1.5 s 完整仿真结果）

完整 1.5 s GPU 仿真（最细网格 `dx = 4 cm`）在约 `t = 0.4 s` 后进入准定常旋涡脱落阶段，末 25% 时段主要气动统计指标如下（下方所有流场图均支持**点击放大**，视频支持**拖动进度条与暂停**以便组会讲解）：

| 关键指标 | 数值结果（`t = 1.500 s`） |
| :--- | :--- |
| 稳态平均阻力系数 $$ C_d $$（末 25% 时段） | **1.457** |
| 阻力系数脉动幅值（`Cd swing`） | **0.190**（箭体与舵面尾流非定常旋涡脱落） |
| 流场最大马赫数 `max Mach` | **0.38** |
| 流场最高温度 `max T` | **290 K**（来流 281 K） |
| 流场最大压比 `max p / p_inf` | **1.1** |

### 7.1 综合总览（马赫数 / 压力 / 温度 + $$ C_d $$ 历史曲线）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/summary/plt07313.png" title="综合总览 plt07313 (t = 1.498 s)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/summary/plt07318.png" title="综合总览 plt07318 (t = 1.499 s)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/summary/plt07323.png" title="综合总览 plt07323 (t = 1.500 s)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  综合流场与阻力系数曲线关键帧对比（从左至右：<code>plt07313</code> / <code>t = 1.498 s</code>、<code>plt07318</code> / <code>t = 1.499 s</code>、<code>plt07323</code> / <code>t = 1.500 s</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="metadata" poster="{{ '/assets/img/case5/summary/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_summary_frames_zoom.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  Case 5 综合流场（马赫数、无量纲压力、温度与 <i>C<sub>d</sub></i> 时历曲线）0 → 1.5 s 全过程演化视频。
</div>

### 7.2 马赫数流场（Mach Number）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/mach/plt07313.png" title="马赫数流场 plt07313 (t = 1498.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/mach/plt07318.png" title="马赫数流场 plt07318 (t = 1499.06 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/mach/plt07323.png" title="马赫数流场 plt07323 (t = 1500.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  马赫数分布关键帧对比（从左至右：<code>t = 1498.03 ms</code>、<code>1499.06 ms</code>、<code>1500.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="metadata" poster="{{ '/assets/img/case5/mach/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_mach.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  马赫数流场与 <i>C<sub>d</sub></i> 同步演化视频（0 → 1.5 s）。
</div>

### 7.3 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/pressure/plt07313.png" title="压力流场 plt07313 (t = 1498.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/pressure/plt07318.png" title="压力流场 plt07318 (t = 1499.06 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/pressure/plt07323.png" title="压力流场 plt07323 (t = 1500.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 分布关键帧对比（从左至右：<code>t = 1498.03 ms</code>、<code>1499.06 ms</code>、<code>1500.00 ms</code>），头部滞止区与 60° 舵面迎风侧高压区清晰可见。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="metadata" poster="{{ '/assets/img/case5/pressure/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 1.5 s）。
</div>

### 7.4 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/temperature/plt07313.png" title="温度流场 plt07313 (t = 1498.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/temperature/plt07318.png" title="温度流场 plt07318 (t = 1499.06 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/temperature/plt07323.png" title="温度流场 plt07323 (t = 1500.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（从左至右：<code>t = 1498.03 ms</code>、<code>1499.06 ms</code>、<code>1500.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="metadata" poster="{{ '/assets/img/case5/temperature/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  温度场演化视频（0 → 1.5 s）。
</div>

### 7.5 数值纹影（Schlieren，垂直刀口）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/schlieren/plt07128.png" title="数值纹影 plt07128 (t = 1460.19 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/schlieren/plt07226.png" title="数值纹影 plt07226 (t = 1480.19 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/schlieren/plt07323.png" title="数值纹影 plt07323 (t = 1500.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影（沿 <i>z</i> 向视线积分，垂直刀口）关键帧对比（从左至右：<code>plt07128</code> / <code>t = 1460.19 ms</code>、<code>plt07226</code> / <code>1480.19 ms</code>、<code>plt07323</code> / <code>1500.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" autoplay loop muted playsinline controls poster="{{ '/assets/img/case5/schlieren/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影动态演化视频（自动循环播放，支持暂停与进度拖动）。
</div>

### 7.6 数值阴影（Shadowgraph）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/shadowgraph/plt07128.png" title="数值阴影 plt07128 (t = 1460.19 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/shadowgraph/plt07226.png" title="数值阴影 plt07226 (t = 1480.19 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/shadowgraph/plt07323.png" title="数值阴影 plt07323 (t = 1500.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值阴影（密度二阶导数视线积分）关键帧对比（从左至右：<code>plt07128</code> / <code>t = 1460.19 ms</code>、<code>plt07226</code> / <code>1480.19 ms</code>、<code>plt07323</code> / <code>1500.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" autoplay loop muted playsinline controls poster="{{ '/assets/img/case5/shadowgraph/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_shadowgraph.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值阴影动态演化视频（自动循环播放，支持暂停与进度拖动）。
</div>

### 7.7 数值干涉（Interferogram，无限条纹）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/interferogram/plt07128.png" title="数值干涉 plt07128 (t = 1460.19 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/interferogram/plt07226.png" title="数值干涉 plt07226 (t = 1480.19 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case5/interferogram/plt07323.png" title="数值干涉 plt07323 (t = 1500.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值干涉（无限条纹，&lambda; = 5820 nm 缩放）关键帧对比（从左至右：<code>plt07128</code> / <code>t = 1460.19 ms</code>、<code>plt07226</code> / <code>1480.19 ms</code>、<code>plt07323</code> / <code>1500.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" autoplay loop muted playsinline controls poster="{{ '/assets/img/case5/interferogram/plt07323.png' | relative_url }}">
      <source src="{{ '/assets/video/case5/movie_interferogram.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值干涉动态演化视频（自动循环播放，支持暂停与进度拖动）。
</div>
