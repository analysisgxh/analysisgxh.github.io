---
layout: page
title: "Case 4: 火箭 10 km / Ma 1.0 跨声速仿真"
description: 60° 舵面跨声速气动仿真、AMR 显存分析、chk00672 断点续算与网格叠加流场展示
img: assets/img/case4/cover.png
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
  .cfg-card-warn {
    border-left: 4px solid #d29922;
  }
  .cfg-card-ok {
    border-left: 4px solid #2ea043;
  }
</style>

本页整理自火箭气动数值模拟项目 **Case 4**（10 km 高度、Mach 1.0 跨声速、60° 气动减速舵面）的实际运行参数 `inputs.used`、气体状态方程核验、16 GB GPU 显存瓶颈分析与 `chk00672`（`0.12 s` → `0.30 s`）断点续算流程，以及纯净流场和 **AMR 网格叠加（`--grids`）** 全套可视化结果。

---

## 一、算例工况与气体状态方程（EOS）核验

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/case5_stl_views.png" title="火箭不同舵面展开角 STL 几何构型（0° / 30° / 60° / 90°）" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  火箭减速舵面四种展开角 STL 模型对比（点击可放大）。Case 4 采用 60° 展开角构型 <code>model_v1_60.stl</code>，来流沿 <code>+y</code> 方向以跨声速 Mach 1.0 入射。
</div>

求解器采用理想空气状态方程（$$ \gamma = 1.4,\ \mu = 28.97 $$），由自由来流输入（`prob.mach = 1.0`，`prob.p_inf = 2.64e5` dyn/cm²，`prob.T_inf = 222.5` K）导出的热力学状态与标准飞行工况表高度吻合：

| 物理参数 | 飞行工况表目标值（Case 4, 10 km） | 实际配置 `inputs.used` 计算值 |
| :--- | :---: | :---: |
| **来流马赫数** $$ \text{Ma}_\infty $$ | `1` | **`1.0`** |
| **来流静温** $$ T_\infty $$ | `222.5 K` | **`222.5 K`** |
| **来流静压** $$ p_\infty $$ | `26.4 kPa` | **`26.4 kPa`**（`2.64e5 dyn/cm²`） |
| **来流声速** $$ c_\infty $$ | `299 m/s` | **`299.001 m/s`** |
| **来流速度** $$ U_\infty $$ | `300 m/s` | **`299.001 m/s`** |
| **来流密度** $$ \rho_\infty $$ | `0.413 kg/m³` | **`0.413415 kg/m³`** |

---

## 二、Case 4 实际运行参数分栏详解（`CASE4/inputs.used`）

本次算例实际使用的 `inputs.used` 复用 Case 5 已验证的 60° 舵面 4 cm 网格框架并替换为 Case 4 跨声速物理条件（独立直接读取，不拼接 `common.inputs`），以下按六大功能模块**分栏讲出**：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 自由来流与初始化模式（Free Stream &amp; Init）</h4>
      <ul>
        <li><b>跨声速来流</b>：<code>prob.mach = 1.0</code>，<code>prob.p_inf = 2.64e5</code> dyn/cm²，<code>prob.T_inf = 222.5</code> K，流向 <code>+y</code>（<code>0. 1. 0.</code>）</li>
        <li><b>实际初始化设置</b>：<code>prob.init_mode = 1</code>（域内初始静止，由入口来流推进发展；保留备用参数 <code>init_front = -120.</code>、<code>init_vel_frac = 0.5</code>）</li>
        <li><b>初始化对比说明</b>：与原 v4 Case 4 的均匀来流（<code>init_mode = 0</code>）不同，<code>init_mode = 1</code> 在跨声速下会在入口附近形成驻留启动波，使箭体前有效来流马赫数降低</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>2. 60° 舵面几何与嵌入式边界（STL &amp; EB2）</h4>
      <ul>
        <li><b>STL 几何输入</b>：<code>eb2.geom_type = stl</code>，运行目录内拷贝 <code>geometry/model_v1_60.stl</code> 为 <code>rocket.stl</code></li>
        <li><b>坐标缩放与对中</b>：<code>eb2.stl_scale = 0.1</code>（mm → cm），<code>eb2.stl_center = 0. 40.91 0.</code>，<code>stl_reverse_normal = 0</code></li>
        <li><b>EB 构建方式</b>：<code>eb2.cover_multiple_cuts = 1</code>，<code>eb2.coarse_by_coarsening = 0</code>（各加密层直接解析 STL，保持默认 <code>eb2.max_grid_size</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. 大尺度计算域与边界条件（Domain &amp; BCs）</h4>
      <ul>
        <li><b>计算域范围</b>：<code>x, z ∈ [-1024, 1024]</code> cm，<code>y ∈ [-1024, 2048]</code> cm（物理尺寸 <code>20.48 × 30.72 × 20.48 m</code>，适应跨声速大范围扰动传播）</li>
        <li><b>基础网格划分</b>：笛卡尔非周期网格（<code>coord_sys = 0</code>），粗网格 <code>amr.n_cell = 64 96 64</code>，L0 分辨率 <b>32 cm</b></li>
        <li><b>边界设定</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y = -1024 cm</code> 来流入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧壁与出口零梯度外流 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. 欧拉求解器与切割小单元修复（Euler &amp; Repair）</h4>
      <ul>
        <li><b>控制方程</b>：无粘可压缩欧拉方程（<code>cns.do_visc = 0</code>，<code>eos_gamma = 1.4</code>，<code>eos_mu = 28.97</code>）</li>
        <li><b>空间重构与时间步</b>：2 阶 PLM（<code>plm_iorder = 2</code>，<code>plm_theta = 1.0</code> minmod 限制器），<code>cfl = 0.3</code>，开启通量重分配与层间守恒修正 <code>do_reflux = 1</code></li>
        <li><b>小单元保护阈值</b>：<code>tiny_cell_vfrac = 0.1</code>，温度下限 <code>T_floor = 10.0 K</code>，小单元温度修复阈值 <code>repair_T_min = 50.0 K</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>5. AMR 四层网格与内存池配置（AMR &amp; Arena）</h4>
      <ul>
        <li><b>网格层级</b>：<code>amr.max_level = 3</code>，<code>ref_ratio = 2 2 2</code>，对应四层分辨率 <b>L0–L3 = 32 / 16 / 8 / 4 cm</b></li>
        <li><b>分块与重构控制</b>：<code>max_grid_size = 32</code>，<code>blocking_factor = 8</code>，<code>regrid_int = 4 4 4</code>，<code>n_error_buf = 2 2 2 2</code>，<code>grid_eff = 0.7</code></li>
        <li><b>负载均衡与内存池</b>：<code>loadbalance_with_workestimates = 1</code>（每 10 步更新 L0）；<code>amrex.the_arena_init_size = 536870912</code>（512 MiB 初始分配池）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 三重网格加密与采样输出（Refinement &amp; I/O）</h4>
      <ul>
        <li><b>表面与密度差加密</b>：<code>refine_cutcells = 1</code>（最高 <b>L3：4 cm</b>）；密度差加密 <code>refine_max_dengrad_lev = 3</code>（阈值 <code>1.1e-5</code>），<code>refine_shock_max_lev = 0</code></li>
        <li><b>固定加密盒</b>：<code>refine_box_lo_0 = -256. -192. -256.</code> 至 <code>hi_0 = 256. 1280. 256.</code> cm（完整覆盖箭体与尾流区）</li>
        <li><b>输出与续算时长</b>：<code>plot_per = 0.001 s</code>，<code>check_per = 0.1 s</code>，每 2 步输出 <code>forces.csv</code>（<code>ref_area = 31415.93 cm²</code>）；首段 <code>0.12 s</code>，续算至 <code>0.30 s</code></li>
      </ul>
    </div>
  </div>
</div>

---

## 三、显存瓶颈分析与两阶段续算方案（`LOGCASE4` 分栏复盘）

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-warn">
      <h4>1. 16 GB 单卡 GPU 显存瓶颈与降显存备选配置</h4>
      <p>测试表明单卡显存耗尽并非 <code>512 MiB</code> 初始内存池设置错误：AMReX 已实际占用约 <b>15 GB</b>，继续申请 <code>8 MiB</code> 时触发上限。原因在于大固定框全量细化至 4 cm（L3）仅框内就包含约 <b>603 万网格单元</b>。</p>
      <p class="mb-1">若需在 16 GB 单卡上全程独立运行，可保持壁面 <b>4 cm</b> 不变，将固定框与密度判据上限降为 <b>8 cm（L2）</b>：</p>
      <pre><code>cns.refine_cutcells_max_lev = 3   # 火箭表面保持 4 cm
cns.refine_max_dengrad_lev  = 2   # 密度判据上限降为 8 cm
cns.zones = body                  # 固定框填充上限降为 L2 (8 cm)
cns.zone.body.lo = -256. -192. -256.
cns.zone.body.hi =  256. 1280.  256.
cns.zone.body.fill_lev = 2</code></pre>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-ok">
      <h4>2. 完整 4 cm 精度断点续算方案（<code>chk00672</code> → <code>0.3 s</code>）</h4>
      <p>为严格保留原 <code>inputs.used</code> 的全套 <b>4 cm 固定框高精度网格</b>，在完成前 <code>0.12 s</code>（生成检查点 <code>chk00672</code>）后，使用 <b>CPU 20 进程 MPI</b> 无缝续算至目标 <code>0.30 s</code>（<code>plt01667</code>）：</p>
      <pre><code class="language-bash">RUN_DIR="$PROJECT_DIR/results/case4_10km_M1_f60_cpu_20261126_133245"
EXE="$PROJECT_DIR/build_cpu/bin/rocket_cns"
export OMP_NUM_THREADS=1

cd "$RUN_DIR"
mpirun -np 20 "$EXE" inputs.used \
    amr.restart=chk00672 \
    stop_time=0.3 \
    2&gt;&amp;1 | tee "run_restart_$(date +%Y%m%d_%H%M%S).log"</code></pre>
    </div>
  </div>
</div>

### 后处理与 AMR 网格框叠加（`--grids`）生成命令

```bash
conda activate rocket-post
RUN_DIR="$PWD/results/case4_10km_M1_f60_cpu_20261126_133245"

# 1. 生成 z=0 截面全时段帧图（同时支持纯净流场与 --grids 叠加 AMR 分块线）
python3 tools/post/make_frames.py "$RUN_DIR" \
    --vars mach pressure density temperature schlieren \
    --title "Case4: Mach 1, 10 km, flap 60 deg" \
    --jobs 4 --grids --overwrite

# 2. 按 15 fps 分别合成 MP4 与 GIF 动画
python3 tools/post/make_movie.py "$RUN_DIR" \
    --vars mach_grids pressure_grids density_grids temperature_grids schlieren_grids \
    --fps 15
```

---

## 四、算例结果展示（`0 → 300 ms` 流场演化与 AMR 网格叠加对比）

下方依次展示 **全计算域 AMR 网格结构**、**纯净流场（三图并列 + 视频）**、**叠加 AMR 多级分块网格流场（`--grids`，三图并列 + 视频）** 以及 **`z = 0` 全域二维切片前期演化**。所有图片支持**点击放大**，视频采用按需加载与单路互斥播放，流畅不卡顿。

<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach_grids.png" title="全计算域 z = 0 截面马赫数与 AMR 多级网格框分布 (plt00672, t = 0.12 s)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  续算起点检查点时刻（<code>plt00672</code>，<code>t = 0.12 s</code>，<code>z = 0 cm</code>）全计算域马赫数与 L1–L3 自适应网格分块布局：核心加密盒与入口波面加密层清晰可见。
</div>

---

### A. 纯净流场与 $$ C_d $$ 时历曲线（`298.12` / `299.03` / `300.00 ms`）

#### 4.1 马赫数流场（Mach Number）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach/plt01656.png" title="马赫数 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach/plt01661.png" title="马赫数 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach/plt01667.png" title="马赫数 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  马赫数分布关键帧并列对比（从左至右：<code>plt01656</code> / <code>298.12 ms</code>、<code>plt01661</code> / <code>299.03 ms</code>、<code>plt01667</code> / <code>300.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/mach/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_mach.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  马赫数流场与 <i>C<sub>d</sub></i> 同步演化视频（0 → 300 ms）。
</div>

#### 4.2 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/pressure/plt01656.png" title="压力流场 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/pressure/plt01661.png" title="压力流场 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/pressure/plt01667.png" title="压力流场 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 分布关键帧对比（<code>298.12 ms</code> / <code>299.03 ms</code> / <code>300.00 ms</code>），鼻锥滞止高压区、60° 舵面迎风高压与背风低压涡结构清晰。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/pressure/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 300 ms）。
</div>

#### 4.3 密度流场（Density, $$ \rho / \rho_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/density/plt01656.png" title="密度流场 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/density/plt01661.png" title="密度流场 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/density/plt01667.png" title="密度流场 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 分布关键帧对比（<code>298.12 ms</code> / <code>299.03 ms</code> / <code>300.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/density/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 演化视频（0 → 300 ms）。
</div>

#### 4.4 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/temperature/plt01656.png" title="温度流场 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/temperature/plt01661.png" title="温度流场 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/temperature/plt01667.png" title="温度流场 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>298.12 ms</code> / <code>299.03 ms</code> / <code>300.00 ms</code>），清晰呈现非定常高温尾流涡街结构。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/temperature/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  静温场演化视频（0 → 300 ms）。
</div>

#### 4.5 数值纹影（Numerical Schlieren）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/schlieren/plt01656.png" title="数值纹影 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/schlieren/plt01661.png" title="数值纹影 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/schlieren/plt01667.png" title="数值纹影 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影关键帧对比（深色代表强密度梯度），精细刻画 60° 舵面分离剪切层与尾流涡系耗散过程。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/schlieren/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影演化视频（0 → 300 ms）。
</div>

---

### B. 叠加 AMR 自适应网格分块框流场展示（`--grids`）

叠加各层级 `32³` 网格块边界（红/黄/绿线框对应由粗到细的 AMR 层级），直观验证密度梯度判据与固定加密盒对尾流涡街的动态捕获能力：

#### 4.6 马赫数 + AMR 网格（Mach + Grids）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach_grids/plt01656.png" title="马赫数+网格 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach_grids/plt01661.png" title="马赫数+网格 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/mach_grids/plt01667.png" title="马赫数+网格 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  叠加 AMR 网格分块边界的马赫数关键帧对比（<code>298.12 ms</code> / <code>299.03 ms</code> / <code>300.00 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/mach_grids/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_mach_grids.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  马赫数 + AMR 动态网格重构演化视频（0 → 300 ms）。
</div>

#### 4.7 压力 + AMR 网格（Pressure + Grids）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/pressure_grids/plt01656.png" title="压力+网格 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/pressure_grids/plt01661.png" title="压力+网格 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/pressure_grids/plt01667.png" title="压力+网格 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  叠加 AMR 网格分块边界的无量纲压力关键帧对比。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/pressure_grids/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_pressure_grids.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  压力场 + AMR 网格演化视频（0 → 300 ms）。
</div>

#### 4.8 密度 + AMR 网格（Density + Grids）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/density_grids/plt01656.png" title="密度+网格 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/density_grids/plt01661.png" title="密度+网格 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/density_grids/plt01667.png" title="密度+网格 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  叠加 AMR 网格分块边界的无量纲密度关键帧对比。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/density_grids/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_density_grids.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  密度场 + AMR 网格演化视频（0 → 300 ms）。
</div>

#### 4.9 温度 + AMR 网格（Temperature + Grids）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/temperature_grids/plt01656.png" title="温度+网格 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/temperature_grids/plt01661.png" title="温度+网格 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/temperature_grids/plt01667.png" title="温度+网格 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  叠加 AMR 网格分块边界的静温关键帧对比。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/temperature_grids/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_temperature_grids.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  温度场 + AMR 网格演化视频（0 → 300 ms）。
</div>

#### 4.10 数值纹影 + AMR 网格（Schlieren + Grids）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/schlieren_grids/plt01656.png" title="数值纹影+网格 plt01656 (t = 298.12 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/schlieren_grids/plt01661.png" title="数值纹影+网格 plt01661 (t = 299.03 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/schlieren_grids/plt01667.png" title="数值纹影+网格 plt01667 (t = 300.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  叠加 AMR 网格分块边界的数值纹影关键帧对比：L3 绿色细网格完整包络尾迹强剪切层区域。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/schlieren_grids/plt01667.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/movie_schlieren_grids.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影 + AMR 网格演化视频（0 → 300 ms）。
</div>

---

### C. 首阶段全计算域 `z = 0` 二维切片演化（`0 → 0.12 s`）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/slices_z0/plt00660.png" title="z=0 全域密度切片 plt00660" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/slices_z0/plt00666.png" title="z=0 全域密度切片 plt00666" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case4/slices_z0/plt00672.png" title="z=0 全域密度切片 plt00672 (t = 0.12 s)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  首阶段末期（<code>plt00660</code> / <code>plt00666</code> / <code>plt00672</code>，<code>t → 0.12 s</code>）<code>z = 0 cm</code> 全计算域切片三帧并列对比。
</div>

<div class="row mt-3">
  <div class="col-sm-4 mt-3 mt-md-0">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/mach_grids.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/slices_z0/movie_mach.mp4' | relative_url }}" type="video/mp4">
    </video>
    <div class="caption">全域切片：马赫数（<code>movie_mach.mp4</code>）</div>
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none">
      <source src="{{ '/assets/video/case4/slices_z0/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
    <div class="caption">全域切片：压力（<code>movie_pressure.mp4</code>）</div>
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case4/slices_z0/plt00672.png' | relative_url }}">
      <source src="{{ '/assets/video/case4/slices_z0/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
    <div class="caption">全域切片：密度（<code>movie_density.mp4</code>）</div>
  </div>
</div>

<div class="row mt-2">
  <div class="col-sm-4 mt-3 mt-md-0">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none">
      <source src="{{ '/assets/video/case4/slices_z0/movie_Temp.mp4' | relative_url }}" type="video/mp4">
    </video>
    <div class="caption">全域切片：温度（<code>movie_Temp.mp4</code>）</div>
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none">
      <source src="{{ '/assets/video/case4/slices_z0/movie_x_velocity.mp4' | relative_url }}" type="video/mp4">
    </video>
    <div class="caption">全域切片：<i>x</i> 向速度（<code>movie_x_velocity.mp4</code>）</div>
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none">
      <source src="{{ '/assets/video/case4/slices_z0/movie_z_velocity.mp4' | relative_url }}" type="video/mp4">
    </video>
    <div class="caption">全域切片：<i>z</i> 向速度（<code>movie_z_velocity.mp4</code>）</div>
  </div>
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
