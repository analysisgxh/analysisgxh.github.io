---
layout: page
title: "Case 2: 火箭 60 km / Ma 16 高空仿真"
description: 60° 舵面高超声速强激波仿真、近真空数值保护配置与流场演化展示
img: assets/img/case2/cover.png
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
</style>

火箭气动数值模拟项目 **Case 2**（60 km 高空、Mach 16 极高马赫数、60° 气动减速舵面）的物理工况、强激波与近真空数值保护分栏配置及多物理场演化结果。

---

## 一、飞行工况与物理特性

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/case5_stl_views.png" title="火箭不同舵面展开角 STL 几何构型（0° / 30° / 60° / 90°）" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  火箭减速舵面四种展开角 STL 模型对比。Case 2 采用 60° 展开角构型 <code>model_v1_60.stl</code>，来流沿 <code>+y</code> 方向以 Mach 16 入射。
</div>

Case 2 对应再入返回高空段 **60 km、Mach 16** 工况。此时努森数 $$ \text{Kn} \approx 10^{-4} < 10^{-3} $$，连续介质假设依然成立；在理想气体（$$ \gamma = 1.4,\ \mu = 28.97 $$）模型下，强激波压比超过 300，激波层无离解滞止温度超过 $$ 1.2 \times 10^4\ \text{K} $$，对求解器的强间断鲁棒性与背风近真空保正性提出了极高要求：

| 工况参数 | 设定 / 导出数值 | 备注说明 |
| :--- | :---: | :--- |
| **飞行高度** | `60 km` | 高空稀薄过渡前缘连续流区（$$ \text{Kn} \approx 10^{-4} $$） |
| **自由来流马赫数** $$ \text{Ma}_\infty $$ | **`16.0`** | 极高超声速强激波工况 |
| **来流静压** $$ p_\infty $$ | **`220.0 dyn/cm²`**（`0.022 kPa`） | 标准大气 60 km 静压（`22 Pa`） |
| **来流静温** $$ T_\infty $$ | **`246.9 K`** | 对应声速 $$ c_\infty \approx 315\ \text{m/s} $$ |
| **来流速度** $$ U_\infty $$ | **`5040 m/s`** | 动压 $$ q_\infty \approx 3.9\ \text{kPa} $$，特征雷诺数 $$ \text{Re} \approx 6.4 \times 10^5 $$ |
| **来流密度** $$ \rho_\infty $$ | **`3.10 × 10⁻⁴ kg/m³`** | CGS 单位 `3.10 × 10⁻⁷ g/cm³` |

---

## 二、Case 2 算例配置分栏（`case2/inputs.used`）

`inputs.used` 在复用 Case 5 的 60° 舵面 4 cm 自适应网格布局基础上，专门针对 Mach 16 强激波与底部近真空膨胀区引入了**降阶重构、低 CFL 数及密度/速度双限幅保护**：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 高超声速来流与静止启动（Free Stream &amp; Init）</h4>
      <ul>
        <li><b>来流条件（CGS）</b>：<code>prob.mach = 16.0</code>，<code>prob.p_inf = 220.0</code> dyn/cm²，<code>prob.T_inf = 246.9</code> K，流向 <code>+y</code>（<code>0. 1. 0.</code>）</li>
        <li><b>静止气体启动</b>：<code>prob.init_mode = 1</code>（初始全域静止，由 <code>y = -1024 cm</code> 入口注入 Mach 16 高速来流）</li>
        <li><b>快速收敛特性</b>：在 <code>5040 m/s</code> 高流速下，入射波约 <code>3.2 ms</code> 抵达箭体，约 <code>15 ms</code> 后波系与阻力系数即进入平稳状态</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>2. 强激波与近真空数值保护（Numerical Safeguards）</h4>
      <ul>
        <li><b>强激波鲁棒推进</b>：采用 1 阶空间重构 <code>cns.plm_iorder = 1</code> 配合保守 <code>cns.cfl = 0.1</code>，消除极强弓形激波跨跃处的非物理过冲</li>
        <li><b>背风近真空双限幅</b>：开启密度下限 <code>cns.rho_floor = 3.1e-10</code> g/cm³（约 <code>10⁻³ ρ_inf</code>）与速度上限 <code>cns.vel_max = 8.1e5</code> cm/s（约 <code>1.6 U_inf</code>），防止底部与 60° 舵面背风膨胀区出现真空负密度发散</li>
        <li><b>切割小单元保护</b>：<code>tiny_cell_vfrac = 0.2</code>，<code>T_floor = 10.0 K</code>，<code>repair_T_min = 20.0 K</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. 60° 舵面 STL 与嵌入式边界（STL &amp; EB2）</h4>
      <ul>
        <li><b>几何模型</b>：<code>eb2.geom_type = stl</code>，运行目录加载 <code>rocket.stl</code>（<code>model_v1_60.stl</code>）</li>
        <li><b>缩放与中心偏移</b>：<code>eb2.stl_scale = 0.1</code>（mm → cm），<code>eb2.stl_center = 0. 40.91 0.</code>，<code>stl_reverse_normal = 0</code></li>
        <li><b>多切割面解析</b>：<code>eb2.cover_multiple_cuts = 1</code>，<code>eb2.coarse_by_coarsening = 0</code>（各 AMR 层直接由 STL 构建）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. 计算域与边界条件（Domain &amp; BCs）</h4>
      <ul>
        <li><b>计算域尺度</b>：<code>x, z ∈ [-1024, 1024]</code> cm，<code>y ∈ [-1024, 2048]</code> cm（<code>20.48 × 30.72 × 20.48 m</code>）</li>
        <li><b>基础网格</b>：非周期笛卡尔坐标系，<code>amr.n_cell = 64 96 64</code>，L0 基础单元尺寸 <b>32 cm</b></li>
        <li><b>边界条件</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y-low</code> 超声速来流入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧壁与出口零梯度外流 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>5. AMR 四层网格与负载均衡（AMR Hierarchy）</h4>
      <ul>
        <li><b>四层加密分辨率</b>：<code>amr.max_level = 3</code>，<code>ref_ratio = 2 2 2</code>，对应 <b>L0–L3 = 32 / 16 / 8 / 4 cm</b></li>
        <li><b>网格分块参数</b>：<code>max_grid_size = 32</code>，<code>blocking_factor = 8</code>，<code>regrid_int = 4 4 4</code>，<code>n_error_buf = 2 2 2 2</code>，<code>grid_eff = 0.7</code></li>
        <li><b>守恒与负载均衡</b>：开启层间通量重分配 <code>do_reflux = 1</code> 与工作量估计负载均衡 <code>loadbalance_with_workestimates = 1</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 三重网格加密与采样控制（Refinement &amp; I/O）</h4>
      <ul>
        <li><b>表面与固定盒加密</b>：<code>refine_cutcells = 1</code>（最高 <b>L3：4 cm</b>）；固定加密盒 <code>[-256, 256] × [-192, 1280] × [-256, 256] cm</code></li>
        <li><b>密度梯度判据</b>：<code>refine_max_dengrad_lev = 3</code>，<code>refine_dengrad = 1.1e-5</code>；<code>refine_shock_max_lev = 0</code></li>
        <li><b>步数与输出间隔</b>：<code>max_step = 200000</code>，<code>stop_time = 0.3</code>（因高波速与 <code>cfl = 0.1</code> 步长极小，推进至 <code>plt18001</code> / <code>43.00 ms</code> 已充分稳态），<code>plot_per = 0.001 s</code>，<code>check_per = 0.1 s</code></li>
      </ul>
    </div>
  </div>
</div>

---

## 三、算例结果展示（Mach 16 强弓形激波与气动特性）

计算至 `plt18001`（`t = 43.00 ms`，对应超过 14 倍全计算域流经时间），贴体弓形激波、60° 舵面斜激波及气动阻力系数均已高度收敛稳定：

| 关键气动与流场指标 | 统计结果（`plt18001`，`t = 43.00 ms`，最细网格 `4 cm`） |
| :--- | :--- |
| 稳态平均阻力系数 $$ C_d $$（末 25% 时段） | **1.878** |
| 阻力系数脉动幅值（`Cd swing`） | **0.019**（流场与激波位形高度定常） |
| 流场最大马赫数 `max Mach` | **16.00** |
| 流场最高温度 `max T` | **15790 K**（理想气体无化学离解滞止高温） |
| 流场最大压比 `max p / p_inf` | **342.5**（Mach 16 强激波驻点压缩，理论正激波后压比约 298.5） |

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    {% include figure.liquid path="assets/img/case2/summary.png" title="Case 2 算例总览图 (plt18001, t = 0.043 s)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  Case 2 算例综合总览（<code>plt18001</code>，<code>t = 0.043 s</code>）：马赫数、对数压力比、温度分布与 <i>C<sub>d</sub></i> 收敛曲线。
</div>

### 3.1 马赫数流场（Mach Number）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/mach/plt16894.png" title="马赫数流场 plt16894 (t = 41.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/mach/plt17447.png" title="马赫数流场 plt17447 (t = 42.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/mach/plt18001.png" title="马赫数流场 plt18001 (t = 43.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  马赫数分布关键帧对比（从左至右：<code>plt16894</code> / <code>41.00 ms</code>、<code>plt17447</code> / <code>42.00 ms</code>、<code>plt18001</code> / <code>43.00 ms</code>）：Mach 16 下弓形激波紧贴鼻锥并呈大后掠角包络箭体。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case2/mach/plt18001.png' | relative_url }}">
      <source src="{{ '/assets/video/case2/movie_mach.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  马赫数流场与 <i>C<sub>d</sub></i> 同步演化视频（0 → 43 ms）。
</div>

### 3.2 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/pressure/plt16894.png" title="压力流场 plt16894 (t = 41.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/pressure/plt17447.png" title="压力流场 plt17447 (t = 42.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/pressure/plt18001.png" title="压力流场 plt18001 (t = 43.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 对数色标分布对比（<code>41.00 ms</code> / <code>42.00 ms</code> / <code>43.00 ms</code>），鼻锥激波层与 60° 舵面迎风侧二次激波高压区清晰对称。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case2/pressure/plt18001.png' | relative_url }}">
      <source src="{{ '/assets/video/case2/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 43 ms）。
</div>

### 3.3 密度流场（Density, $$ \rho / \rho_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/density/plt16894.png" title="密度流场 plt16894 (t = 41.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/density/plt17447.png" title="密度流场 plt17447 (t = 42.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/density/plt18001.png" title="密度流场 plt18001 (t = 43.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 对数色标分布对比：理想气体（&gamma; = 1.4）强激波密度比逼近理论极限 6，舵面与箭体背风侧呈现低于 <code>0.05 &rho;<sub>&infin;</sub></code> 的强膨胀稀薄尾迹。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case2/density/plt18001.png' | relative_url }}">
      <source src="{{ '/assets/video/case2/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 演化视频（0 → 43 ms）。
</div>

### 3.4 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/temperature/plt16894.png" title="温度流场 plt16894 (t = 41.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/temperature/plt17447.png" title="温度流场 plt17447 (t = 42.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case2/temperature/plt18001.png" title="温度流场 plt18001 (t = 43.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>41.00 ms</code> / <code>42.00 ms</code> / <code>43.00 ms</code>）：鼻锥贴体激波层与 60° 舵面迎风再压缩激波形成显著高温层。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case2/temperature/plt18001.png' | relative_url }}">
      <source src="{{ '/assets/video/case2/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  静温场演化视频（0 → 43 ms）。
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
