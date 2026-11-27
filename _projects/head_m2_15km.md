---
layout: page
title: "Head M2: 火箭头部 15 km / Ma 2 精细仿真"
description: v5 头部局部计算域 0.5 cm 超高分辨率仿真、冲击启动渐进控制与脱体激波演化展示
img: assets/img/head_m2_15km/cover.png
importance: 0
category: work
pretty_table: true
---

<style>
  .cfg-grid {
    margin-top: 0.75rem;
  }
  .cfg-col {
    margin-bottom: 1rem;
  }
  .cfg-card {
    height: 100%;
    padding: 1rem 1.15rem;
    border-radius: 10px;
    background: rgba(127, 127, 127, 0.07);
    border: 1px solid rgba(127, 127, 127, 0.22);
  }
  .cfg-card h4 {
    font-size: 1rem;
    font-weight: 600;
    margin-top: 0;
    margin-bottom: 0.55rem;
    padding-bottom: 0.4rem;
    border-bottom: 1px solid rgba(127, 127, 127, 0.2);
  }
  .cfg-card ul {
    padding-left: 1.15rem;
    margin-bottom: 0;
    font-size: 0.88rem;
    line-height: 1.58;
  }
  .cfg-card li {
    margin-bottom: 0.28rem;
  }
  .cfg-card p {
    font-size: 0.88rem;
    line-height: 1.58;
    margin-bottom: 0.45rem;
  }
  .cfg-card pre {
    margin-top: 0.45rem;
    margin-bottom: 0;
    font-size: 0.8rem;
  }
  .cfg-card-accent {
    border-left: 4px solid #388bfd;
  }
  .cfg-card-warn {
    border-left: 4px solid #d29922;
  }
</style>

火箭气动数值模拟项目 **head_m2_15km**（v5 头部局部计算域、15 km 平流层高度、Mach 2.0 超声速、最细网格 0.5 cm）的几何构型与飞行工况分栏分析、`inputs.used` 核心参数配置分栏详解及稳态波系演化结果。

---

## 一、头部局部构型与飞行工况分栏详解（15 km / Mach 2.0）

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 15 km 平流层超声速自由来流状态</h4>
      <ul>
        <li><b>飞行高度与马赫数</b>：<code>15 km</code>，<code>prob.mach = 2.0</code>，来流沿 <code>+y</code> 轴方向（<code>0. 1. 0.</code>）</li>
        <li><b>环境静压与静温</b>：<code>prob.p_inf = 1.211e5</code> dyn/cm²（<code>12.11 kPa</code>），<code>prob.T_inf = 216.65 K</code></li>
        <li><b>来流速度与气体属性</b>：<code>U_inf = 590 m/s</code>，理想气体比热比 <code>γ = 1.4</code>，摩尔质量 <code>μ = 28.97 g/mol</code>，无粘欧拉模型 <code>do_visc = 0</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>2. v5 火箭头部局部截取构型（Head-Only）</h4>
      <ul>
        <li><b>几何截取策略</b>：加载 <code>model_v1_0.stl</code>（<code>stl_scale = 0.1</code> mm→cm，<code>stl_center = 0. 0. 0.</code>），出口面截断于 <code>y = +64 cm</code>，尾部舵面（<code>y &gt; 360 cm</code>）置于计算域外以集中算力解析前体流场</li>
        <li><b>前体精细结构</b>：包含激波针（针尖 <code>y = -68.6 cm</code>，<code>r: 0 → 12 cm</code>，针根 <code>y = -48.5 cm</code>）、鼻帽过渡段（<code>y = -47 .. -13 cm</code>，<code>r → 100 cm</code>）、肩部凸缘（<code>y = -13 .. -10 cm</code>，<code>r = 112 cm</code>）及肩后约 <code>0.75 m</code> 圆柱箭体（<code>r = 100 cm</code>）</li>
        <li><b>前体阻力定义</b>：圆柱段直接穿过超声速出口边界，无底部低压尾迹干扰，所测 <i>C<sub>d</sub></i> 纯粹表征<b>火箭头部前体压差阻力系数</b></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-warn">
      <h4>3. Mach 2 宽张角脱体弓形激波特征</h4>
      <ul>
        <li><b>30° 大马赫角效应</b>：<code>Ma = 2.0</code> 对应马赫角 <code>arcsin(1/2) = 30°</code>，相比高马赫数工况，头部弓形激波脱体距离大幅前移且横向扩张角最宽</li>
        <li><b>激波驻留与穿出尺度</b>：主脱体弓形激波前推至 <code>y ≈ -100 cm</code>（激波针尖上游约 <code>32 cm</code>），在 <code>y = +64 cm</code> 出口截面处激波半径扩展至 <code>r ≈ 331 cm</code></li>
        <li><b>加宽横向计算域</b>：采用 v5 头部系列最宽横向盒子 <code>x, z ∈ [-416, 416] cm</code>、<code>y ∈ [-192, 64] cm</code>（<code>8.32 × 2.56 × 8.32 m</code>），确保弓形激波由下游超声速出口无反射穿出</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>4. 时间尺度与稳态气动收敛特性</h4>
      <ul>
        <li><b>全域流经时间</b>：轴向域长 <code>2.56 m</code> 对应自由来流特征流经时间 <code>t_flow = 2.56 / 590 ≈ 4.3 ms</code></li>
        <li><b>稳态推进时长</b>：推进至 <code>stop_time = 0.02 s</code>（<code>20.00 ms</code>，<code>plt00963</code>），覆盖约 <code>4.6</code> 倍全域流经时间</li>
        <li><b>气动力收敛精度</b>：后 <code>25 %</code> 时段内前体阻力系数稳定于 <b><i>C<sub>d</sub></i> = 1.322</b>，稳态脉动幅值仅 <b>&Delta;<i>C<sub>d</sub></i> = 0.005</b></li>
      </ul>
    </div>
  </div>
</div>

---

## 二、算例参数配置分栏详解（`head_M2_15KM/inputs.used`）

`head_M2_15KM/inputs.used` 基于 v5 头部局部超高分辨率架构设计，采用**均匀来流冲击启动 + `1.5 ms` 渐进过渡保护 + 五层 AMR 分区加密（最细 `0.5 cm`）**，核心配置分栏如下：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 均匀来流冲击启动（Impulsive Start）</h4>
      <ul>
        <li><b>初始化模式</b>：<code>prob.init_mode = 0</code>（<code>t = 0</code> 时刻计算域全域赋值为均匀 Mach 2.0 自由来流）</li>
        <li><b>选型物理依据</b>：在 v4 全箭测试中，Mach 2 静止气体启动（<code>init_mode = 1</code>）易在钝头前产生大尺度亚声速滞止包并诱发数值发散；改用均匀来流冲击启动可直接由壁面反射建立脱体弓形激波</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>2. v5 启动过渡段渐进保护（Startup Ramp）</h4>
      <ul>
        <li><b>1.5 ms 渐进启动窗口</b>：<code>t &lt; cns.startup_time = 1.5e-3 s</code> 期间强制使用一阶迎风重构（<code>cns.startup_plm_iorder = 1</code>），同时将 CFL 数由 <code>cns.cfl_start = 0.1</code> 线性爬升至 <code>cns.cfl = 0.3</code>，消除激波针尖微小切割单元初始剥离负压</li>
        <li><b>稳态二阶高精度格式</b>：<code>t ≥ 1.5 ms</code> 自动切换至二阶 PLM 重构（<code>cns.plm_iorder = 2</code>，<code>plm_theta = 1.0</code> minmod 限制器），开启粗细网格通量修正 <code>do_reflux = 1</code></li>
        <li><b>小切单元热力学保护</b>：<code>tiny_cell_vfrac = 0.2</code>，<code>T_floor = 10.0 K</code>，<code>repair_T_min = 50.0 K</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. STL 嵌入边界高分辨率构建（STL &amp; EB2）</h4>
      <ul>
        <li><b>几何文件与坐标</b>：<code>eb2.geom_type = stl</code>，<code>eb2.stl_file = model_v1_0.stl</code>，<code>eb2.stl_scale = 0.1</code>，<code>eb2.stl_center = 0. 0. 0.</code></li>
        <li><b>0.5 cm 级 EB 构建参数</b>：<code>eb2.cover_multiple_cuts = 1</code> 覆盖多重切割单元，<code>eb2.coarse_by_coarsening = 0</code> 在各层独立精确重建几何，<code>eb2.maxiter = 200</code>，<code>eb2.max_grid_size = 16</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. 加宽头部计算域与边界条件（Domain &amp; BCs）</h4>
      <ul>
        <li><b>计算域范围</b>：<code>prob_lo = -416. -192. -416.</code>，<code>prob_hi = 416. 64. 416.</code>（cm），物理尺寸 <code>8.32 × 2.56 × 8.32 m</code>，<code>is_periodic = 0 0 0</code></li>
        <li><b>基础网格划分</b>：<code>amr.n_cell = 104 32 104</code>，对应粗网格单元尺寸 <b>L0 = 8 cm</b></li>
        <li><b>边界条件配置</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y = -192 cm</code> 自由来流入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧壁与 <code>y = +64 cm</code> 超声速零梯度出口 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>5. 五层 AMR 网格与 <code>tip</code> / <code>head</code> 双分区加密</h4>
      <ul>
        <li><b>五层网格分辨率</b>：<code>amr.max_level = 4</code>（<code>ref_ratio = 2 2 2 2</code>），对应 <b>L0–L4 = 8 / 4 / 2 / 1 / 0.5 cm</b>（<code>blocking_factor = 8</code>，<code>max_grid_size = 32</code>，<code>regrid_int = 4</code>，<code>n_error_buf = 2</code>，开启 <code>loadbalance_with_workestimates = 1</code>）</li>
        <li><b><code>tip</code> 针尖核心区（0.5 cm）</b>：<code>[-24, 24] × [-80, -40] × [-24, 24]</code> cm 包裹激波针尖与针根，<code>fill_lev = cut_lev = shock_lev = 4</code> 锁定最细 <b>0.5 cm</b></li>
        <li><b><code>head</code> 头部激波层区（1–2 cm）</b>：<code>[-176, 176] × [-160, 0] × [-176, 176]</code> cm 覆盖鼻帽激波层与肩部凸缘，背景 <code>fill_lev = 2</code>（<b>2 cm</b>），表面与激波 <code>cut_lev = shock_lev = 3</code>（<b>1 cm</b>）</li>
        <li><b>全局判据与显存约束</b>：区外箭体表面 <code>refine_cutcells_max_lev = 3</code>（<b>1 cm</b>）；外翼激波 <code>refine_shock_max_lev = 2</code>（<b>2 cm</b>，<code>pjump = 0.2</code>；测试表明若升至 L3 总网格将超 900 万并突破 15 GB 显存）；<code>refine_max_dengrad_lev = 0</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 采样输出与前体气动力积分（Output &amp; Forces）</h4>
      <ul>
        <li><b>推进终止控制</b>：<code>stop_time = 0.02 s</code>（<code>max_step = 50000</code>）</li>
        <li><b>流场与检查点输出</b>：<code>amr.plot_per = 4.0e-4 s</code>（每 <code>0.4 ms</code> 输出一帧流场），<code>amr.check_per = 0.004 s</code>（每 <code>4 ms</code> 保存检查点）</li>
        <li><b>前体阻力实时监测</b>：<code>forces.int = 2</code> 持续输出 <code>forces.csv</code>，参考面积 <code>forces.ref_area = 31415.93 cm²</code>（<code>πR²</code>，<code>R = 100 cm</code>）</li>
      </ul>
    </div>
  </div>
</div>

---

## 三、算例结果展示（0.5 cm 分辨率脱体激波与肩部膨胀流场）

### 3.0 稳态流场定量汇总与综合总览（`plt00963`, `t = 20.00 ms`）

| 指标名称 | 稳态数值 | 物理与数值特征说明 |
| :--- | :--- | :--- |
| **最细网格尺度** $$ \Delta x_{\min} $$ | **`0.5 cm`**（L4） | 激波针尖、针根与前缘波系达到亚厘米级解析度 |
| **前体平均阻力系数** $$ C_d $$（后 25 %） | **`1.322`** | 纯头部构型（激波针 + 鼻帽 + 肩部凸缘）无粘压差阻力 |
| **稳态阻力脉动幅值** $$ \Delta C_d $$ | **`0.005`** | 启动波系排出出口后气动力高度平稳 |
| **流场最高马赫数** $$ \mathrm{Ma}_{\max} $$ | **`2.81`** | 肩部凸缘（`r = 112 → 100 cm`）Prandtl–Meyer 膨胀扇超声速加速 |
| **流场最高静温** $$ T_{\max} $$ | **`413 K`** | 鼻帽驻点区与针根再附着激波气动压缩温升（$$ T_\infty = 216.65\text{ K} $$） |
| **最高无量纲静压** $$ (p / p_\infty)_{\max} $$ | **`5.7`** | 鼻帽迎风驻点高压区（与 `Ma = 2.0` 理论正激波后总压比 `5.64` 一致） |

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    {% include figure.liquid loading="eager" path="assets/img/head_m2_15km/summary.png" title="head_m2_15km 稳态多物理场、AMR 网格层级、三维头部流线与 Cd 收敛曲线总览 (t = 20.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  稳态时刻（<code>plt00963</code>，<code>t = 20.00 ms</code>）综合诊断总览：包含中心切片无量纲静压、静温、马赫数、叠加五层 AMR 网格框的数值纹影、三维头部几何绕流流线以及 <code>0 → 20 ms</code> 前体阻力系数 <i>C<sub>d</sub></i> 收敛历程。
</div>

### 3.1 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/pressure/plt00926.png" title="压力流场 plt00926 (t = 19.22 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/pressure/plt00944.png" title="压力流场 plt00944 (t = 19.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/pressure/plt00963.png" title="压力流场 plt00963 (t = 20.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 关键帧对比（从左至右：<code>plt00926</code> / <code>19.22 ms</code>、<code>plt00944</code> / <code>19.61 ms</code>、<code>plt00963</code> / <code>20.00 ms</code>）：主脱体弓形激波驻留于 <code>y ≈ -1.0 m</code>，鼻帽迎风侧形成高压滞止区（峰值 <code>5.7 p_inf</code>），气流绕过肩部凸缘后经膨胀扇迅速降压。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m2_15km/pressure/plt00963.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m2_15km/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 20.00 ms）。
</div>

### 3.2 密度流场（Density, $$ \rho / \rho_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/density/plt00926.png" title="密度流场 plt00926 (t = 19.22 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/density/plt00944.png" title="密度流场 plt00944 (t = 19.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/density/plt00963.png" title="密度流场 plt00963 (t = 20.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 关键帧对比（<code>19.22 ms</code> / <code>19.61 ms</code> / <code>20.00 ms</code>）：清晰呈现脱体激波层内的高密度压缩帽、激波针侧缘剪切层以及肩部凸缘下游环形低密度膨胀区。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m2_15km/density/plt00963.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m2_15km/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 演化视频（0 → 20.00 ms）。
</div>

### 3.3 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/temperature/plt00926.png" title="温度流场 plt00926 (t = 19.22 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/temperature/plt00944.png" title="温度流场 plt00944 (t = 19.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/temperature/plt00963.png" title="温度流场 plt00963 (t = 20.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>19.22 ms</code> / <code>19.61 ms</code> / <code>20.00 ms</code>）：鼻帽激波层与针根壁面附近静温由 <code>216.65 K</code> 升至峰值 <code>413 K</code>，肩部超声速膨胀加速区绝热冷却至约 <code>150 K</code>。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m2_15km/temperature/plt00963.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m2_15km/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  静温场演化视频（0 → 20.00 ms）。
</div>

### 3.4 数值纹影（Numerical Schlieren）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/schlieren/plt00926.png" title="数值纹影 plt00926 (t = 19.22 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/schlieren/plt00944.png" title="数值纹影 plt00944 (t = 19.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m2_15km/schlieren/plt00963.png" title="数值纹影 plt00963 (t = 20.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影关键帧对比：<code>0.5 cm</code>（L4）与 <code>1 cm</code>（L3）精细网格锐利捕捉了主脱体弓形激波、激波针前缘压缩波与剪切层、肩部 Prandtl–Meyer 膨胀扇及圆柱段再压缩波系。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m2_15km/schlieren/plt00963.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m2_15km/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影动态演化视频（0 → 20.00 ms）。
</div>

<script>
  document.addEventListener("DOMContentLoaded", () => {
    const videos = Array.from(document.querySelectorAll("video"));
    videos.forEach((v) => {
      v.addEventListener("play", () => {
        videos.forEach((other) => {
          if (other !== v && !other.paused) other.pause();
        });
      });
    });
    if ("IntersectionObserver" in window) {
      const io = new IntersectionObserver((entries) => {
        entries.forEach((e) => {
          if (!e.isIntersecting && !e.target.paused) e.target.pause();
        });
      }, { threshold: 0.1 });
      videos.forEach((v) => io.observe(v));
    }
  });
</script>
