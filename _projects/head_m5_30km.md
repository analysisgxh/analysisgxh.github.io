---
layout: page
title: "Head M5: 火箭头部 30 km / Ma 5 精细仿真"
description: v5 头部局部计算域 0.5 cm 超高分辨率仿真、贴体弓形激波与激波层剪切涡精细解析
img: assets/img/head_m5_30km/cover.png
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

火箭气动数值模拟项目 **head_m5_30km**（v5 头部局部计算域、30 km 平流层高空、Mach 5.0 高超声速、最细网格 0.5 cm，对应 v4 Case 3 头部高分辨率复现）的几何构型与飞行工况分栏分析、`inputs.used` 核心参数配置分栏详解及稳态波系演化结果。

---

## 一、头部局部构型与飞行工况分栏详解（30 km / Mach 5.0）

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 30 km 平流层高超声速自由来流状态</h4>
      <ul>
        <li><b>飞行高度与马赫数</b>：<code>30 km</code>，<code>prob.mach = 5.0</code>（对应 v4 全箭 Case 3 工况的头部超高分辨率算例），来流沿 <code>+y</code> 轴方向（<code>0. 1. 0.</code>）</li>
        <li><b>环境静压与静温</b>：<code>prob.p_inf = 1.2e4</code> dyn/cm²（<code>1.2 kPa</code>），<code>prob.T_inf = 227.0 K</code></li>
        <li><b>来流速度与气体属性</b>：<code>U_inf = 1510 m/s</code>，理想气体比热比 <code>γ = 1.4</code>，摩尔质量 <code>μ = 28.97 g/mol</code>，无粘欧拉模型 <code>do_visc = 0</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>2. v5 火箭头部局部截取构型（Head-Only）</h4>
      <ul>
        <li><b>几何截取策略</b>：加载 <code>0°</code> 收拢构型 <code>rocket.stl</code>（<code>model_v1_0.stl</code>，<code>stl_scale = 0.1</code> mm→cm，<code>stl_center = 0. 0. 0.</code>），下游出口截断于 <code>y = +64 cm</code>（肩后保留约 <code>0.77 m</code> 箭体），尾部舵面（<code>y &gt; 360 cm</code>）置于域外</li>
        <li><b>前体精细结构</b>：包含头部激波针（针尖 <code>y = -68.6 cm</code>，<code>r: 0 → 12 cm</code>，针根 <code>y = -48.5 cm</code>）、鼻帽过渡段（<code>y = -47 .. -13 cm</code>，<code>r → 100 cm</code>）及肩部凸缘（<code>y = -13 .. -10 cm</code>，<code>r = 112 cm</code>）</li>
        <li><b>前体压差阻力定义</b>：圆柱箭体侧壁平行于来流并穿过超声速出口边界，所测 <i>C<sub>d</sub></i> 纯粹表征<b>前体（头部）高超声速压差阻力系数</b></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-warn">
      <h4>3. Mach 5 贴体弓形激波与激波层剪切涡特征</h4>
      <ul>
        <li><b>11.5° 小马赫角贴体效应</b>：<code>Ma = 5.0</code> 马赫角仅 <code>arcsin(1/5) = 11.5°</code>，脱体弓形激波显著后移贴体，驻留于激波针尖上游约 <code>5–8 cm</code>（<code>y ≈ -73 .. -76 cm</code>，距入口保留约 <code>115 cm</code> 未扰动自由来流）</li>
        <li><b>横向穿出与紧凑盒子</b>：弓形激波在 <code>y = +64 cm</code> 出口面处半径收窄至 <code>r ≈ 232 cm</code>，横向计算域收紧为 <code>x, z ∈ [-320, 320] cm</code>（距侧壁留有约 <code>90 cm</code> 裕度，无侧壁激波反射）</li>
        <li><b>针—帽激波干扰涡结构</b>：<code>0.5 cm</code> 细网格清晰解析出激波针前缘压缩波与钝鼻帽强弓形激波相交诱发的 Kelvin–Helmholtz 不稳定剪切涡列</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>4. 高波速时间尺度与稳态气动收敛特性</h4>
      <ul>
        <li><b>极短流经时间</b>：轴向域长 <code>2.56 m</code> 在 <code>1510 m/s</code> 来流速度下特征流经时间仅 <code>t_flow ≈ 1.7 ms</code></li>
        <li><b>稳态推进时长</b>：推进至 <code>stop_time = 0.006 s</code>（<code>6.00 ms</code>，<code>plt00533</code>），覆盖约 <code>3.5</code> 倍全域流经时间</li>
        <li><b>气动力收敛精度</b>：<code>t &gt; 1.5 ms</code> 后阻力曲线完全平直，后 <code>25 %</code> 时段前体平均阻力系数稳定于 <b><i>C<sub>d</sub></i> = 1.445</b>，脉动幅值低至 <b>&Delta;<i>C<sub>d</sub></i> = 0.001</b></li>
      </ul>
    </div>
  </div>
</div>

---

## 二、算例参数配置分栏详解（`head_m5_30km/inputs.used`）

`head_m5_30km/inputs.used` 由 `tools/run_case.py` 组合 v5 `common.inputs` 与 `head_M5_30km/inputs` 生成，采用**均匀来流冲击启动 + `0.5 ms` 渐进过渡保护 + 五层 AMR 紧凑双分区加密（最细 `0.5 cm`）**，核心配置分栏如下：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 均匀来流冲击启动（Impulsive Start）</h4>
      <ul>
        <li><b>初始化模式</b>：<code>prob.init_mode = 0</code>（<code>t = 0</code> 时刻计算域全域赋值为均匀 Mach 5.0 自由来流）</li>
        <li><b>方案优势</b>：无需等待入口启动激波扫过计算域，直接由前体壁面反射快速建立贴体脱体弓形激波，缩短非定常过渡时间</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>2. v5 启动过渡段渐进保护（Startup Ramp）</h4>
      <ul>
        <li><b>0.5 ms 快速启动窗口</b>：匹配 <code>1510 m/s</code> 高波速（气流约 4 次扫过 <code>60 cm</code> 头部区），设定 <code>cns.startup_time = 5.0e-4 s</code>（<code>0.5 ms</code>），期间强制采用一阶迎风重构（<code>cns.startup_plm_iorder = 1</code>）并将 CFL 由 <code>cns.cfl_start = 0.1</code> 线性提升至 <code>cns.cfl = 0.3</code></li>
        <li><b>稳态二阶高精度重构</b>：<code>t ≥ 0.5 ms</code> 自动切换至二阶 PLM 重构（<code>cns.plm_iorder = 2</code>，<code>plm_theta = 1.0</code> minmod 限制器），开启粗细界面通量重分配 <code>do_reflux = 1</code></li>
        <li><b>小切单元保正修复</b>：<code>tiny_cell_vfrac = 0.2</code>，<code>T_floor = 10.0 K</code>，<code>repair_T_min = 50.0 K</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. STL 嵌入边界高分辨率构建（STL &amp; EB2）</h4>
      <ul>
        <li><b>几何文件与坐标</b>：<code>eb2.geom_type = stl</code>，<code>eb2.stl_file = rocket.stl</code>，<code>eb2.stl_scale = 0.1</code>，<code>eb2.stl_center = 0. 0. 0.</code></li>
        <li><b>0.5 cm 级 EB 参数</b>：<code>eb2.cover_multiple_cuts = 1</code>，<code>eb2.coarse_by_coarsening = 0</code> 各层直接由 STL 精确构建几何，<code>eb2.maxiter = 200</code>，<code>eb2.max_grid_size = 16</code> 降低细层 EB 内存占用</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. Mach 5 紧凑头部计算域与边界（Domain &amp; BCs）</h4>
      <ul>
        <li><b>计算域范围</b>：<code>prob_lo = -320. -192. -320.</code>，<code>prob_hi = 320. 64. 320.</code>（cm），物理尺寸 <code>6.40 × 2.56 × 6.40 m</code>（相比 <code>Ma = 2</code> 横向收窄 <code>23 %</code>），<code>is_periodic = 0 0 0</code></li>
        <li><b>基础网格划分</b>：<code>amr.n_cell = 80 32 80</code>，对应粗网格单元尺寸 <b>L0 = 8 cm</b></li>
        <li><b>边界条件配置</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y = -192 cm</code> 自由来流入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧壁与 <code>y = +64 cm</code> 超声速零梯度出口 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>5. 五层 AMR 网格与紧凑 <code>tip</code> / <code>head</code> 双分区加密</h4>
      <ul>
        <li><b>五层网格分辨率</b>：<code>amr.max_level = 4</code>（<code>ref_ratio = 2 2 2 2</code>），对应 <b>L0–L4 = 8 / 4 / 2 / 1 / 0.5 cm</b>（<code>blocking_factor = 8</code>，<code>max_grid_size = 32</code>，<code>regrid_int = 4</code>，<code>n_error_buf = 2</code>，开启 <code>loadbalance_with_workestimates = 1</code>）</li>
        <li><b><code>tip</code> 针尖核心区（0.5 cm）</b>：<code>[-24, 24] × [-80, -40] × [-24, 24]</code> cm 包裹激波针尖与针根，<code>fill_lev = cut_lev = shock_lev = 4</code> 锁定最细 <b>0.5 cm</b></li>
        <li><b><code>head</code> 紧凑激波层区（1–2 cm）</b>：随激波贴体收紧至 <code>[-136, 136] × [-112, 0] × [-136, 136]</code> cm，背景 <code>fill_lev = 2</code>（<b>2 cm</b>），鼻帽/肩部表面与主激波 <code>cut_lev = shock_lev = 3</code>（<b>1 cm</b>）</li>
        <li><b>全局判据与显存约束</b>：区外箭体表面 <code>refine_cutcells_max_lev = 3</code>（<b>1 cm</b>）；外翼弓形激波 <code>refine_shock_max_lev = 2</code>（<b>2 cm</b>，<code>pjump = 0.2</code>；测试表明若升至 L3 在 <code>t = 0.4 ms</code> 即超 900 万网格并突破 15 GB 显存）；<code>refine_max_dengrad_lev = 0</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 高频采样输出与前体阻力积分（Output &amp; Forces）</h4>
      <ul>
        <li><b>推进终止控制</b>：<code>stop_time = 0.006 s</code>（<code>max_step = 10000000</code>）</li>
        <li><b>流场与检查点频率</b>：<code>amr.plot_per = 1.0e-4 s</code>（每 <code>0.1 ms</code> 输出一帧流场），<code>amr.check_per = 0.001 s</code>（每 <code>1 ms</code> 保存检查点）</li>
        <li><b>前体阻力实时监测</b>：<code>forces.int = 2</code> 持续输出 <code>forces.csv</code>，参考面积 <code>forces.ref_area = 31415.93 cm²</code>（<code>πR²</code>，<code>R = 100 cm</code>）</li>
      </ul>
    </div>
  </div>
</div>

---

## 三、算例结果展示（0.5 cm 分辨率贴体激波与剪切涡流场）

### 3.0 稳态流场定量汇总与综合总览（`plt00533`, `t = 6.00 ms`）

| 指标名称 | 稳态数值 | 物理与数值特征说明 |
| :--- | :--- | :--- |
| **最细网格尺度** $$ \Delta x_{\min} $$ | **`0.5 cm`**（L4） | 激波针尖、针根与前缘剪切涡达到亚厘米级解析度 |
| **前体平均阻力系数** $$ C_d $$（后 25 %） | **`1.445`** | 纯头部构型（激波针 + 鼻帽 + 肩部凸缘）高超声速压差阻力 |
| **稳态阻力脉动幅值** $$ \Delta C_d $$ | **`0.001`** | 波系建立后前体气动载荷极度平稳 |
| **流场最高马赫数** $$ \mathrm{Ma}_{\max} $$ | **`5.00`** | 未扰动高超声速自由来流马赫数 |
| **流场最高静温** $$ T_{\max} $$ | **`1521 K`** | 鼻帽驻点与针根激波干扰高温区（$$ T_\infty = 227.0\text{ K} $$） |
| **最高无量纲静压** $$ (p / p_\infty)_{\max} $$ | **`32.6`** | 鼻帽迎风驻点高压（与 `Ma = 5.0` 理论 Rayleigh Pitot 驻点压比 `32.65` 吻合） |

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    {% include figure.liquid loading="eager" path="assets/img/head_m5_30km/summary.png" title="head_m5_30km 稳态马赫数、无量纲压力、静温切片与 Cd 收敛曲线总览 (t = 6.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  稳态时刻（<code>plt00533</code>，<code>t = 6.00 ms</code>）综合诊断总览：上方依次展示中心切片马赫数、对数无量纲静压 <code>log10(p / p_inf)</code>、静温分布与算例统计摘要，下方展示 <code>0 → 6.0 ms</code> 前体阻力系数 <i>C<sub>d</sub></i> 收敛历程。
</div>

### 3.1 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/pressure/plt00517.png" title="压力流场 plt00517 (t = 5.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/pressure/plt00525.png" title="压力流场 plt00525 (t = 5.90 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/pressure/plt00533.png" title="压力流场 plt00533 (t = 6.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 对数色标关键帧对比（从左至右：<code>plt00517</code> / <code>5.80 ms</code>、<code>plt00525</code> / <code>5.90 ms</code>、<code>plt00533</code> / <code>6.00 ms</code>）：Mach 5 弓形激波紧贴激波针尖，鼻帽激波层内压比达 <code>32.6</code>，肩部凸缘后方形成低压膨胀阴影区。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m5_30km/pressure/plt00533.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m5_30km/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 6.00 ms）。
</div>

### 3.2 密度流场（Density, $$ \rho / \rho_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/density/plt00517.png" title="密度流场 plt00517 (t = 5.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/density/plt00525.png" title="密度流场 plt00525 (t = 5.90 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/density/plt00533.png" title="密度流场 plt00533 (t = 6.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 对数色标关键帧对比（<code>5.80 ms</code> / <code>5.90 ms</code> / <code>6.00 ms</code>）：弓形激波后密度比超过 <code>5.0</code>（逼近 <code>γ = 1.4</code> 正激波理论值 <code>5.00</code>），肩部凸缘下游圆柱近壁区膨胀稀疏至 <code>0.5 &rho;<sub>&infin;</sub></code> 以下。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m5_30km/density/plt00533.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m5_30km/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 演化视频（0 → 6.00 ms）。
</div>

### 3.3 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/temperature/plt00517.png" title="温度流场 plt00517 (t = 5.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/temperature/plt00525.png" title="温度流场 plt00525 (t = 5.90 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/temperature/plt00533.png" title="温度流场 plt00533 (t = 6.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>5.80 ms</code> / <code>5.90 ms</code> / <code>6.00 ms</code>）：鼻帽前强压缩激波层温度升至 <code>1200–1521 K</code>，激波针两侧清晰可见非定常剪切涡卷起引起的热斑脉动，绕过肩部后降温至 <code>400–600 K</code>。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m5_30km/temperature/plt00533.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m5_30km/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  静温场演化视频（0 → 6.00 ms）。
</div>

### 3.4 数值纹影（Numerical Schlieren）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/schlieren/plt00517.png" title="数值纹影 plt00517 (t = 5.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/schlieren/plt00525.png" title="数值纹影 plt00525 (t = 5.90 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m5_30km/schlieren/plt00533.png" title="数值纹影 plt00533 (t = 6.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影关键帧对比：<code>head</code> 分区内 <code>1 cm</code> 与 <code>tip</code> 分区内 <code>0.5 cm</code> 细网格锐利刻画出贴体主弓形激波、激波针侧剪切涡卷起及肩部膨胀波系，外侧同时反映出向 <code>2 cm</code> 翼部激波网格的过渡边界。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m5_30km/schlieren/plt00533.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m5_30km/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影动态演化视频（0 → 6.00 ms）。
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
