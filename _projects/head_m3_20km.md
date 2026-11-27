---
layout: page
title: "Head M3: 火箭头部 20 km / Ma 3 精细仿真"
description: v5 头部局部计算域 0.5 cm 超高分辨率仿真、中等张角脱体激波与激波层干涉结构展示
img: assets/img/head_m3_20km/cover.png
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

火箭气动数值模拟项目 **head_m3_20km**（v5 头部局部计算域、20 km 平流层高度、Mach 3.0 超声速、最细网格 0.5 cm）的几何构型与飞行工况分栏分析、`inputs.used` 核心参数配置分栏详解及稳态波系演化结果。

---

## 一、头部局部构型与飞行工况分栏详解（20 km / Mach 3.0）

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 20 km 平流层低等温层超声速来流状态</h4>
      <ul>
        <li><b>飞行高度与马赫数</b>：<code>20 km</code>，<code>prob.mach = 3.0</code>（处于 v5 头部系列 <code>Ma 2 → Ma 3 → Ma 5</code> 过渡工况），来流沿 <code>+y</code> 方向（<code>0. 1. 0.</code>）</li>
        <li><b>环境静压与静温</b>：<code>prob.p_inf = 5.529e4</code> dyn/cm²（<code>5.529 kPa</code>），<code>prob.T_inf = 216.65 K</code></li>
        <li><b>来流速度与气体属性</b>：<code>U_inf = 885 m/s</code>，理想气体比热比 <code>γ = 1.4</code>，摩尔质量 <code>μ = 28.97 g/mol</code>，无粘欧拉模型 <code>do_visc = 0</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>2. v5 火箭头部局部截取构型（Head-Only）</h4>
      <ul>
        <li><b>几何截取策略</b>：加载 <code>0°</code> 收拢构型 <code>rocket.stl</code>（<code>model_v1_0.stl</code>，<code>stl_scale = 0.1</code> mm→cm，<code>stl_center = 0. 0. 0.</code>），下游出口截断于 <code>y = +64 cm</code>（肩后保留约 <code>0.77 m</code> 箭体），尾部舵面（<code>y &gt; 360 cm</code>）位于域外</li>
        <li><b>前体精细结构</b>：包含头部激波针（针尖 <code>y = -68.6 cm</code>，<code>r: 0 → 12 cm</code>，针根 <code>y = -48.5 cm</code>）、鼻帽过渡段（<code>y = -47 .. -13 cm</code>，<code>r → 100 cm</code>）及肩部凸缘（<code>y = -13 .. -10 cm</code>，<code>r = 112 cm</code>）</li>
        <li><b>前体压差阻力定义</b>：圆柱段侧壁平行于来流并直接穿过超声速出口面，所测 <i>C<sub>d</sub></i> 纯粹表征<b>火箭头部前体压差阻力系数</b></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-warn">
      <h4>3. Mach 3 中等张角脱体弓形激波与波系干涉</h4>
      <ul>
        <li><b>19.5° 马赫角过渡特征</b>：<code>Ma = 3.0</code> 对应马赫角 <code>arcsin(1/3) = 19.5°</code>，介于 <code>Ma 2</code>（<code>30°</code>）与 <code>Ma 5</code>（<code>11.5°</code>）之间，弓形激波脱体距离与张角呈典型过渡形态</li>
        <li><b>激波脱体与穿出尺度</b>：主脱体弓形激波驻留于 <code>y ≈ -84 cm</code>（激波针尖上游约 <code>16 cm</code>，恰为 <code>Ma 5</code> 的 2 倍、<code>Ma 2</code> 的一半），在 <code>y = +64 cm</code> 出口面处半径扩展至 <code>r ≈ 264 cm</code></li>
        <li><b>适度加宽计算域</b>：横向计算域由 <code>±320 cm</code> 扩展至 <code>x, z ∈ [-352, 352] cm</code>（距侧壁保留约 <code>88 cm</code> 净空），确保激波完全由下游出口面无反射穿出</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>4. 时间尺度与稳态气动收敛特性</h4>
      <ul>
        <li><b>全域流经时间</b>：轴向域长 <code>2.56 m</code> 在 <code>885 m/s</code> 来流下特征流经时间约 <code>t_flow = 2.56 / 885 ≈ 2.9 ms</code></li>
        <li><b>稳态推进时长</b>：算例推进至 <code>plt00597</code>（<code>t = 0.009802 s</code> / <code>9.80 ms</code>），覆盖约 <code>3.4</code> 倍全域流经时间</li>
        <li><b>气动力单调演化规律</b>：后 <code>25 %</code> 时段前体平均阻力系数稳定于 <b><i>C<sub>d</sub></i> = 1.418</b>（末值 <code>1.415</code>，脉动幅值 <b>&Delta;<i>C<sub>d</sub></i> = 0.007</b>），在 <code>Ma 2</code>（<code>1.322</code>）与 <code>Ma 5</code>（<code>1.445</code>）之间呈清晰单调递增趋势</li>
      </ul>
    </div>
  </div>
</div>

---

## 二、算例参数配置分栏详解（`head_m3_20km/inputs.used`）

`head_m3_20km/inputs.used` 由 `tools/run_case.py` 组合 v5 `common.inputs` 与 `head_M3_20km/inputs` 生成，采用**均匀来流冲击启动 + `1.0 ms` 渐进过渡保护 + 适配 `19.5°` 激波张角的五层 AMR 双分区加密（最细 `0.5 cm`）**，核心配置分栏如下：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 均匀来流冲击启动（Impulsive Start）</h4>
      <ul>
        <li><b>初始化模式</b>：<code>prob.init_mode = 0</code>（<code>t = 0</code> 时刻全域初始化为均匀 Mach 3.0 自由来流）</li>
        <li><b>方案优势</b>：避免静止气体启动在钝头前产生慢速亚声速滞止包，直接由前体壁面反射快速建立 <code>y ≈ -84 cm</code> 处的脱体弓形激波</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>2. v5 启动过渡段渐进保护（Startup Ramp）</h4>
      <ul>
        <li><b>1.0 ms 渐进启动窗口</b>：匹配 <code>885 m/s</code> 来流波速设定 <code>cns.startup_time = 1.0e-3 s</code>（<code>1.0 ms</code>，介于 <code>Ma 5</code> 的 <code>0.5 ms</code> 与 <code>Ma 2</code> 的 <code>1.5 ms</code> 之间），期间采用一阶重构（<code>cns.startup_plm_iorder = 1</code>）并将 CFL 由 <code>cns.cfl_start = 0.1</code> 线性爬升至 <code>cns.cfl = 0.3</code></li>
        <li><b>稳态二阶高精度格式</b>：<code>t ≥ 1.0 ms</code> 自动切换至二阶 PLM 重构（<code>cns.plm_iorder = 2</code>，<code>plm_theta = 1.0</code> minmod 限制器），开启层间通量修正 <code>do_reflux = 1</code></li>
        <li><b>小切单元保正保护</b>：<code>tiny_cell_vfrac = 0.2</code>，<code>T_floor = 10.0 K</code>，<code>repair_T_min = 50.0 K</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. STL 嵌入边界高分辨率构建（STL &amp; EB2）</h4>
      <ul>
        <li><b>几何文件与坐标</b>：<code>eb2.geom_type = stl</code>，<code>eb2.stl_file = rocket.stl</code>，<code>eb2.stl_scale = 0.1</code>，<code>eb2.stl_center = 0. 0. 0.</code></li>
        <li><b>0.5 cm 级 EB 参数</b>：<code>eb2.cover_multiple_cuts = 1</code>，<code>eb2.coarse_by_coarsening = 0</code> 各层直接由 STL 独立构建几何，<code>eb2.maxiter = 200</code>，<code>eb2.max_grid_size = 16</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. Mach 3 适度加宽头部计算域与边界（Domain &amp; BCs）</h4>
      <ul>
        <li><b>计算域范围</b>：<code>prob_lo = -352. -192. -352.</code>，<code>prob_hi = 352. 64. 352.</code>（cm），物理尺寸 <code>7.04 × 2.56 × 7.04 m</code>，<code>is_periodic = 0 0 0</code></li>
        <li><b>基础网格划分</b>：<code>amr.n_cell = 88 32 88</code>，保持粗网格单元尺寸 <b>L0 = 8 cm</b></li>
        <li><b>边界条件配置</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y = -192 cm</code> 自由来流入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧壁与 <code>y = +64 cm</code> 超声速零梯度出口 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>5. 五层 AMR 网格与扩容 <code>tip</code> / <code>head</code> 双分区加密</h4>
      <ul>
        <li><b>五层网格分辨率</b>：<code>amr.max_level = 4</code>（<code>ref_ratio = 2 2 2 2</code>），对应 <b>L0–L4 = 8 / 4 / 2 / 1 / 0.5 cm</b>（<code>blocking_factor = 8</code>，<code>max_grid_size = 32</code>，<code>regrid_int = 4</code>，<code>n_error_buf = 2</code>，开启 <code>loadbalance_with_workestimates = 1</code>）</li>
        <li><b><code>tip</code> 针尖核心区（0.5 cm）</b>：<code>[-24, 24] × [-80, -40] × [-24, 24]</code> cm 包裹激波针尖与针根，<code>fill_lev = cut_lev = shock_lev = 4</code> 锁定最细 <b>0.5 cm</b></li>
        <li><b><code>head</code> 激波层扩容区（1–2 cm）</b>：适配前移至 <code>y ≈ -84 cm</code> 的弓形激波，由 <code>[-136,136]×[-112,0]</code> 扩展为 <code>[-160, 160] × [-128, 0] × [-160, 160]</code> cm，背景 <code>fill_lev = 2</code>（<b>2 cm</b>），壁面与主激波 <code>cut_lev = shock_lev = 3</code>（<b>1 cm</b>）</li>
        <li><b>全局判据与显存控制</b>：区外箭体表面 <code>refine_cutcells_max_lev = 3</code>（<b>1 cm</b>）；外翼激波 <code>refine_shock_max_lev = 2</code>（<b>2 cm</b>，<code>pjump = 0.2</code>）；<code>refine_max_dengrad_lev = 0</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 采样输出与前体气动力积分（Output &amp; Forces）</h4>
      <ul>
        <li><b>推进终止控制</b>：配置目标 <code>stop_time = 0.012 s</code>（实际稳态采样至 <code>plt00597</code> / <code>9.80 ms</code>）</li>
        <li><b>流场与检查点频率</b>：<code>amr.plot_per = 2.0e-4 s</code>（每 <code>0.2 ms</code> 输出一帧流场），<code>amr.check_per = 0.002 s</code>（每 <code>2 ms</code> 保存检查点）</li>
        <li><b>前体阻力实时监测</b>：<code>forces.int = 2</code> 持续输出 <code>forces.csv</code>，参考面积 <code>forces.ref_area = 31415.93 cm²</code>（<code>πR²</code>，<code>R = 100 cm</code>）</li>
      </ul>
    </div>
  </div>
</div>

---

## 三、算例结果展示（0.5 cm 分辨率脱体激波与肩部干涉流场）

### 3.0 稳态流场定量汇总与计算域波系总览（`plt00597`, `t = 9.80 ms`）

| 指标名称 | 稳态数值 | 物理与数值特征说明 |
| :--- | :--- | :--- |
| **最细网格尺度** $$ \Delta x_{\min} $$ | **`0.5 cm`**（L4） | 激波针尖、针根及前缘滑移线达到亚厘米级解析度 |
| **前体平均阻力系数** $$ C_d $$（后 25 %） | **`1.418`** | 纯头部构型压差阻力（末值 `1.415`，介于 `Ma 2` 的 `1.322` 与 `Ma 5` 的 `1.445` 之间） |
| **稳态阻力脉动幅值** $$ \Delta C_d $$ | **`0.007`** | 启动波系排出出口后气动载荷平稳收敛 |
| **流场最高马赫数** $$ \mathrm{Ma}_{\max} $$ | **`3.00`** | 未扰动超声速自由来流马赫数 |
| **流场最高静温** $$ T_{\max} $$ | **`644 K`** | 鼻帽驻点与针根再附着压缩高温区（$$ T_\infty = 216.65\text{ K} $$，理论正激波总温 `606.6 K`） |
| **最高无量纲静压** $$ (p / p_\infty)_{\max} $$ | **`12.2`** | 鼻帽迎风驻点高压（与 `Ma = 3.0` 理论 Rayleigh Pitot 驻点压比 `12.06` 吻合） |

<div class="row">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/head_m3_20km/summary.png" title="head_m3_20km 稳态马赫数、无量纲压力、静温切片与 Cd 收敛曲线总览 (t = 9.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/head_m3_20km/head.png" title="head_m3_20km 计算域边界严格对齐切片（z = 0, t = 9.80 ms）" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  稳态时刻（<code>plt00597</code>，<code>t = 9.80 ms</code>）综合诊断对比：左图为马赫数、对数无量纲压力、静温切片与统计摘要总览；右图为严格对齐 <code>7.04 × 2.56 m</code> 计算域边界面（<code>z = 0</code>）的波系穿出验证图，可见脱体弓形激波在 <code>y = +64 cm</code> 出口面约 <code>r ≈ 260 cm</code> 处干净穿出，距 <code>±352 cm</code> 侧壁留有充裕无干扰裕度。
</div>

### 3.1 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/pressure/plt00576.png" title="压力流场 plt00576 (t = 9.41 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/pressure/plt00587.png" title="压力流场 plt00587 (t = 9.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/pressure/plt00597.png" title="压力流场 plt00597 (t = 9.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 对数色标关键帧对比（从左至右：<code>plt00576</code> / <code>9.41 ms</code>、<code>plt00587</code> / <code>9.61 ms</code>、<code>plt00597</code> / <code>9.80 ms</code>）：主弓形激波稳定驻留于 <code>y ≈ -0.84 m</code>，鼻帽迎风侧峰值压比达 <code>12.2</code>，肩部凸缘后方形成明显的膨胀低压扇与二次再压缩波。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m3_20km/pressure/plt00597.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m3_20km/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 9.80 ms）。
</div>

### 3.2 密度流场（Density, $$ \rho / \rho_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/density/plt00576.png" title="密度流场 plt00576 (t = 9.41 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/density/plt00587.png" title="密度流场 plt00587 (t = 9.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/density/plt00597.png" title="密度流场 plt00597 (t = 9.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 对数色标关键帧对比（<code>9.41 ms</code> / <code>9.61 ms</code> / <code>9.80 ms</code>）：激波层内压缩密度比约达 <code>3.8–4.0</code>（与 <code>Ma = 3.0</code> 正激波理论值 <code>3.86</code> 一致），并在外侧 <code>|x| ≈ 1.7 m</code> 处清晰显现嵌入压缩波与肩部低密度膨胀区（<code>&lt; 0.5 &rho;<sub>&infin;</sub></code>）的交汇结构。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m3_20km/density/plt00597.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m3_20km/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 演化视频（0 → 9.80 ms）。
</div>

### 3.3 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/temperature/plt00576.png" title="温度流场 plt00576 (t = 9.41 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/temperature/plt00587.png" title="温度流场 plt00587 (t = 9.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/temperature/plt00597.png" title="温度流场 plt00597 (t = 9.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>9.41 ms</code> / <code>9.61 ms</code> / <code>9.80 ms</code>）：鼻帽前激波层由 <code>216.65 K</code> 压缩升温至峰值 <code>644 K</code>，激波针前缘对称分布轴向滑移热羽流条纹，绕过肩部凸缘后绝热膨胀冷却至 <code>250–300 K</code>。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m3_20km/temperature/plt00597.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m3_20km/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  静温场演化视频（0 → 9.80 ms）。
</div>

### 3.4 数值纹影（Numerical Schlieren）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/schlieren/plt00576.png" title="数值纹影 plt00576 (t = 9.41 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/schlieren/plt00587.png" title="数值纹影 plt00587 (t = 9.61 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/head_m3_20km/schlieren/plt00597.png" title="数值纹影 plt00597 (t = 9.80 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影关键帧对比：<code>0.5 cm</code>（L4）与 <code>1 cm</code>（L3）高分辨率网格精细揭示了主脱体弓形激波、激波针前缘对称滑移剪切层、肩部 Prandtl–Meyer 膨胀扇及下游嵌入尾激波系。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/head_m3_20km/schlieren/plt00597.png' | relative_url }}">
      <source src="{{ '/assets/video/head_m3_20km/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影动态演化视频（0 → 9.80 ms）。
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
