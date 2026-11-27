---
layout: page
title: "Case 3: 火箭 30 km / Ma 5 GPU 仿真"
description: 60° 舵面高超声速流场、AMR 显存诊断调优（inputs2 → inputs3）与激波演化展示
img: assets/img/case3/cover.png
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
    border-left: 4px solid #e05252;
  }
  .cfg-card-ok {
    border-left: 4px solid #2ea043;
  }
</style>

火箭气动数值模拟项目 **Case 3**（30 km 高空、Mach 5 高超声速、60° 气动减速舵面 `case3_30km_M5_f60_gpu_inputs3_full08`）的算例分栏配置、GPU 显存溢出（OOM）单参数诊断调优、后台运行与后处理脚本以及多物理场/光学流场结果展示。

---

## 一、算例背景与几何构型

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/case5_stl_views.png" title="火箭不同舵面展开角 STL 几何构型（0° / 30° / 60° / 90°）" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  火箭减速舵面四种展开角 STL 模型对比。Case 3 采用 60° 展开角构型 <code>model_v1_60.stl</code>，来流沿 <code>+y</code> 方向以 Mach 5 入射。
</div>

Case 3 对应火箭再入返回段 **30 km 高空、Mach 5** 的典型高超声速连续流工况（Kn ≈ 2×10⁻⁶，滞止温度约 1360~1480 K，理想气体欧拉模型适用性良好），用于评估大展开角减速舵面在强弓形激波与舵面脱体激波干扰下的气动阻力特性。

---

## 二、Case 3 算例配置（`inputs3`）

`cases/case3_30km_M5/inputs3` 
为独立完整的 GPU 配置文件（直接读取运行，无需拼接 `common.inputs`），以下按六大功能模块**分栏列出**全部核心配置：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 自由来流与静止启动（Free Stream &amp; Init）</h4>
      <ul>
        <li><b>飞行高度与马赫数</b>：30 km，<code>prob.mach = 5.0</code>，来流方向 <code>+y</code>（<code>0. 1. 0.</code>）</li>
        <li><b>来流静压 / 静温（CGS）</b>：<code>prob.p_inf = 1.2e4</code> dyn/cm²（1.2 kPa），<code>prob.T_inf = 227.0</code> K（声速约 302 m/s，来流速度约 1510 m/s）</li>
        <li><b>初始流场模式</b>：<code>prob.init_mode = 1</code>（<b>全域气体静止</b>，由入口施加 Mach 5 来流向下游推进建立流场；高超声速下启动激波可顺利向下游洗出计算域）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>2. STL 几何与嵌入式边界（STL &amp; AMReX-EB）</h4>
      <ul>
        <li><b>几何文件</b>：<code>eb2.geom_type = stl</code>，<code>eb2.stl_file = rocket.stl</code>（复制自 <code>geometry/model_v1_60.stl</code>）</li>
        <li><b>单位缩放与平移</b>：<code>eb2.stl_scale = 0.1</code>（mm → cm），<code>eb2.stl_center = 0. 40.91 0.</code>（适配 60° 舵面坐标系）</li>
        <li><b>切割单元构建</b>：<code>eb2.cover_multiple_cuts = 1</code>（处理舵面铰链狭缝多重切割），<code>eb2.coarse_by_coarsening = 0</code>（各层直接由 STL 构建）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. 计算域尺寸与边界条件（Domain &amp; BCs）</h4>
      <ul>
        <li><b>计算域范围</b>：<code>x, z ∈ [-1024, 1024]</code> cm，<code>y ∈ [-1024, 2048]</code> cm（总尺度 <code>20.48 × 30.72 × 20.48 m</code>，复用 Case 5 大域将入口前移）</li>
        <li><b>粗网格划分</b>：<code>amr.n_cell = 64 96 64</code>，对应 L0 基础网格间距 <b>32 cm</b></li>
        <li><b>边界类型</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y = -1024 cm</code> 设为超声速自由来流入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧向及下游设为零梯度外流 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. 欧拉格式与小切割单元保护（Solver &amp; Repair）</h4>
      <ul>
        <li><b>气体物性</b>：理想气体无粘求解（<code>cns.do_visc = 0</code>，<code>eos_gamma = 1.4</code>，<code>eos_mu = 28.97</code>）</li>
        <li><b>重构与推进</b>：<code>cns.plm_iorder = 2</code>，<code>plm_theta = 1.0</code>（minmod 限制器抑制强激波振荡），<code>cfl = 0.3</code>，开启层间通量守恒修正 <code>do_reflux = 1</code></li>
        <li><b>小单元温度保护</b>：<code>tiny_cell_vfrac = 0.1</code>，<code>T_floor = 10.0 K</code>，<code>repair_T_min = 50.0 K</code>（防止背风膨胀区切单元非物理低温崩溃）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>5. AMR 网格层级与 GPU 负载均衡（AMR &amp; GPU）</h4>
      <ul>
        <li><b>4 层网格体系</b>：<code>amr.max_level = 3</code>，<code>ref_ratio = 2 2 2</code>，对应分辨率 <b>L0–L3 = 32 / 16 / 8 / 4 cm</b></li>
        <li><b>分块与缓冲</b>：<code>blocking_factor = 8</code>，<code>max_grid_size = 32</code>，<code>regrid_int = 4 4 4</code>，<code>n_error_buf = 2 2 2 2</code>，<code>grid_eff = 0.7</code></li>
        <li><b>GPU 内存池与负载</b>：开启工作量估计负载均衡（<code>loadbalance_with_workestimates = 1</code>）；<code>amrex.the_arena_init_size = 536870912</code>（512 MiB 初始池，非显存硬上限）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 网格加密区域与输出监测（Refinement &amp; I/O）</h4>
      <ul>
        <li><b>表面与固定盒加密</b>：开启切割单元表面加密 <code>refine_cutcells = 1</code>（至 <b>L3：4 cm</b>）；核心固定加密盒 <code>[-256, 256] × [-192, 1280] × [-256, 256] cm</code></li>
        <li><b>动态判据开关</b>：<code>refine_max_dengrad_lev = 0</code>，<code>refine_shock_max_lev = 0</code>（显存受控关键设置，详见下节）</li>
        <li><b>采样与气动力</b>：<code>plot_per = 5.0e-4 s</code>，<code>check_per = 0.005 s</code>；每 2 步积分气动力至 <code>forces.csv</code>（参考面积 <code>31415.93 cm²</code>）</li>
      </ul>
    </div>
  </div>
</div>

---

## 三、显存诊断分栏对比：`inputs2`（OOM）vs `inputs3`（稳定运行）

在 GPU 移植调试中，`inputs2` 因启动激波触发全计算域大规模细网格导致显存溢出；`inputs3` 仅修改**单个参数**即定位并解决了显存瓶颈：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-warn">
      <h4>原方案 <code>inputs2</code>：三重加密导致 GPU 显存溢出</h4>
      <p>同时启用了<b>固定盒加密</b>、<b>火箭表面加密</b>与<b>全域密度差动态加密</b>：</p>
      <pre><code>cns.refine_dengrad = 1.8e-7       # 约 1% rho_inf，高度敏感
cns.refine_max_dengrad_lev = 3    # 允许全域触发 L3 (4 cm)</code></pre>
      <p class="mt-2 mb-0"><b>失败机理</b>：Case 3 采用静止气体 + Mach 5 强入口启动，启动波在入口附近形成横跨大面积区域的密度梯度，高敏阈值瞬间触发海量 L3 细网格，在第 8 粗步（<code>t ≈ 6.78×10⁻⁴ s</code>）耗尽显存（GPU OOM）。</p>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-ok">
      <h4>改进方案 <code>inputs3</code>：关闭动态标定，全程稳定计算</h4>
      <p>相对于 <code>inputs2</code> <b>仅修改一项</b>，关闭密度梯度触发的额外网格增殖：</p>
      <pre><code>cns.refine_max_dengrad_lev = 0    # 关闭密度差动态加密
cns.refine_shock_max_lev   = 0    # 压力跳变动态加密同样保持关闭</code></pre>
      <p class="mt-2 mb-0"><b>核心结论</b>：关闭的是“按流场梯度自动追加细网格”，<b>并非关闭求解器的激波捕捉能力</b>——求解器依然在固定加密盒与近壁 4 cm 网格上精确计算强激波，显存占用稳定可控。</p>
    </div>
  </div>
</div>

### `inputs3` 各区域网格分辨率与激波捕捉效果对照

| 激波所在区域 | 实际网格分辨率 | 对流场与激波捕捉的影响 |
| :--- | :--- | :--- |
| **固定加密盒内部**（`[-256, 256] × [-192, 1280] × [-256, 256] cm`） | **最细 4 cm（L3）** | 完整包络箭体头部弓形激波、60° 舵面激波与近尾迹，高分辨率清晰刻画波系 |
| **火箭表面附近**（STL 切割单元） | **表面加密至 4 cm（L3）** | 保证壁面压力积分与气动阻力系数 $$ C_d $$ 计算精度 |
| **固定盒外远场区域**（`\|x\| > 2.56 m`） | 逐级过渡至 **32 cm（L0）** | 激波仍连续正常传播并顺利穿出外边界，仅远场波面厚度变宽、细节略平滑 |

---

## 四、GPU 两阶段运行与监控流程

采用“先短程启动验证（`0.012 s`）、再长程稳态推进（设定 `stop_time = 0.8`）”的两阶段执行方案：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>阶段一：短程显存诊断验证（<code>0.012 s</code>）</h4>
      <p>读取 <code>inputs3</code> 验证启动激波阶段显存稳定性，输出至 <code>case3_30km_M5_f60_gpu_inputs3_trial</code>：</p>
      <pre><code class="language-bash">ROOT="$HOME/XiaohuiGao/rocket_cns_v4_gpu/rocket_cns_v4"
EXE="$ROOT/build_gpu/bin/rocket_cns"
OUT="$ROOT/results/case3_30km_M5_f60_gpu_inputs3_trial"

export OMP_NUM_THREADS=1 CUDA_VISIBLE_DEVICES=0
mkdir -p "$OUT"
cp "$ROOT/cases/case3_30km_M5/inputs3" "$OUT/inputs.used"
cp "$ROOT/geometry/model_v1_60.stl" "$OUT/rocket.stl"
cd "$OUT" && "$EXE" inputs.used 2&gt;&amp;1 | tee run.log</code></pre>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>阶段二：长程完整计算（<code>full08</code>）</h4>
      <p>在 <code>tmux</code> 会话中追加 <code>stop_time = 0.8</code> 覆盖原短程时长，保存至独立目录 <code>case3_30km_M5_f60_gpu_inputs3_full08</code>：</p>
      <pre><code class="language-bash">OUT="$ROOT/results/case3_30km_M5_f60_gpu_inputs3_full08"
mkdir -p "$OUT"
cp "$ROOT/cases/case3_30km_M5/inputs3" "$OUT/inputs.used"
cp "$ROOT/geometry/model_v1_60.stl" "$OUT/rocket.stl"
printf '\nstop_time = 0.8\n' &gt;&gt; "$OUT/inputs.used"
cd "$OUT" &amp;&amp; "$EXE" inputs.used 2&gt;&amp;1 | tee run.log

# 另开终端实时跟踪进度
tail -n 30 -F "$OUT/run.log"</code></pre>
    </div>
  </div>
</div>

---

## 五、后处理出图与动画合成流水线

算例生成的 `plt*` 时间步数据通过以下脚本生成单帧图、总览序列与多物理场 MP4/GIF 视频（`make_movie.py` 会自动扫描 `$R/frames/*` 与 `$R/summary_frames*` 下包含 `plt*.png` 的直接子目录）：

```bash
R="results/case3_30km_M5_f60_gpu_inputs3_full08"

# 1. 生成单张缩放总览图（马赫数、无量纲压力、温度 + Cd 曲线）
python3 tools/plot_case.py "$R" --zoom 0.8 -o "$R/summary_zoom.png"

# 2. 批量生成各物理量与光学流场帧图并导出动画
python3 tools/post/make_frames.py "$R" --vars mach pressure temperature schlieren --jobs 4
python3 tools/post/make_movie.py "$R" --vars mach pressure temperature schlieren

# 3. 逐个 plt 时刻批量生成总览帧图（加 --zoom 0.8 生成近场放大版）并合成 15 fps 视频
OUT="$R/summary_frames_zoom" && mkdir -p "$OUT"
for P in "$R"/plt*; do
  NAME=$(basename "$P")
  [[ "$NAME" =~ ^plt[0-9]+$ ]] && [ -s "$P/Header" ] || continue
  python3 tools/plot_case.py "$R" --plt "$NAME" --zoom 0.8 -o "$OUT/$NAME.png"
done
python3 tools/post/make_movie.py "$R" --vars summary_frames_zoom --fps 15
```

---

## 六、算例结果展示（Mach 5 高超声速激波演化与气动特性）

计算推进至 `plt09078`（`t = 316.04 ms`，已超过 30 倍箭体特征流经时间），启动激波早在约 `50 ms` 前后完全洗出，流场与气动力高度收敛稳定。下方所有图片均支持**点击放大**，视频默认显示高清预览帧并支持**点击播放、暂停与拖动进度条**（自动单路互斥播放，保证页面流畅不卡顿）：

| 关键气动与流场指标 | 统计结果（`plt09078`，`t = 316.04 ms`，最细网格 `4 cm`） |
| :--- | :--- |
| 稳态平均阻力系数 $$ C_d $$（末 25% 时段） | **2.194**（60° 舵面强激波增阻显著） |
| 阻力系数脉动幅值（`Cd swing`） | **0.045**（高超声速波系稳定，脉动极小） |
| 流场最大马赫数 `max Mach` | **5.00** |
| 流场最高温度 `max T` | **1481 K**（来流 227 K，鼻锥与舵面迎风侧激波气动加热） |
| 流场最大压比 `max p / p_inf` | **35.6**（强弓形激波滞止高压区） |

### 6.1 综合总览（近场放大视图 Summary Zoom）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/summary_zoom/plt09031.png" title="近场总览 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/summary_zoom/plt09057.png" title="近场总览 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/summary_zoom/plt09078.png" title="近场总览 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  近场放大综合总览关键帧并列对比（从左至右：<code>plt09031</code> / <code>315.00 ms</code>、<code>plt09057</code> / <code>315.50 ms</code>、<code>plt09078</code> / <code>316.04 ms</code>）。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/summary_zoom/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_summary_frames_zoom.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  Case 3 近场放大总览（马赫数、对数压力比、温度与 <i>C<sub>d</sub></i> 收敛过程）0 → 316 ms 全程演化视频。
</div>

### 6.2 综合总览（全计算域视图 Summary Full Domain）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/summary/plt09031.png" title="全域总览 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/summary/plt09057.png" title="全域总览 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/summary/plt09078.png" title="全域总览 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  全计算域（<code>20.48 × 30.72 m</code>）总览关键帧对比，可完整观察启动波洗出及大尺度尾迹波系结构。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-12">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/summary/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_summary_frames.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  全计算域总览演化视频（清晰展示 <code>init_mode = 1</code> 静止气体受 Mach 5 入射激波扫掠建立稳态流场的完整物理过程）。
</div>

### 6.3 马赫数流场（Mach Number）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/mach/plt09031.png" title="马赫数流场 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/mach/plt09057.png" title="马赫数流场 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/mach/plt09078.png" title="马赫数流场 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  马赫数分布关键帧对比（<code>315.00 ms</code> / <code>315.50 ms</code> / <code>316.04 ms</code>）：头部弓形激波、60° 舵面斜激波及背风侧低速回流区结构清晰；同时可见 <code>|x| = 2.56 m</code> 固定加密盒边界内外的网格分辨率过渡。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/mach/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_mach.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  马赫数流场与 <i>C<sub>d</sub></i> 同步演化视频（0 → 316 ms）。
</div>

### 6.4 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/pressure/plt09031.png" title="压力流场 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/pressure/plt09057.png" title="压力流场 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/pressure/plt09078.png" title="压力流场 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 分布关键帧对比（<code>315.00 ms</code> / <code>315.50 ms</code> / <code>316.04 ms</code>），鼻锥激波层与 60° 舵面迎风高压区峰值达 35.6 <i>p<sub>&infin;</sub></i>，箭体底部呈强膨胀低压区。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/pressure/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 316 ms）。
</div>

### 6.5 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/temperature/plt09031.png" title="温度流场 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/temperature/plt09057.png" title="温度流场 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/temperature/plt09078.png" title="温度流场 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>315.00 ms</code> / <code>315.50 ms</code> / <code>316.04 ms</code>），鼻锥脱体激波层与舵面再压缩区高温尾迹（最高 1481 K）对称清晰。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/temperature/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  温度场演化视频（0 → 316 ms）。
</div>

### 6.6 数值纹影（Schlieren，垂直刀口）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/schlieren/plt09031.png" title="数值纹影 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/schlieren/plt09057.png" title="数值纹影 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/schlieren/plt09078.png" title="数值纹影 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影（沿 <i>z</i> 视线积分，垂直刀口）关键帧对比：直观展示了固定加密盒内（4 cm）锐利的弓形激波/舵面激波/尾迹剪切层，以及盒外粗网格区的波系连续延伸。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/schlieren/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影动态演化视频（点击播放，支持暂停与进度拖动）。
</div>

### 6.7 数值阴影（Shadowgraph）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/shadowgraph/plt09031.png" title="数值阴影 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/shadowgraph/plt09057.png" title="数值阴影 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/shadowgraph/plt09078.png" title="数值阴影 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值阴影关键帧对比（<code>315.00 ms</code> / <code>315.50 ms</code> / <code>316.04 ms</code>），突出显示激波面密度二阶导数突变位置。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/shadowgraph/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_shadowgraph.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值阴影动态演化视频（点击播放，支持暂停与进度拖动）。
</div>

### 6.8 数值干涉（Interferogram，无限条纹）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/interferogram/plt09031.png" title="数值干涉 plt09031 (t = 315.00 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/interferogram/plt09057.png" title="数值干涉 plt09057 (t = 315.50 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case3/interferogram/plt09078.png" title="数值干涉 plt09078 (t = 316.04 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值干涉（无限条纹，&lambda; = 4662 nm 缩放）关键帧对比（<code>315.00 ms</code> / <code>315.50 ms</code> / <code>316.04 ms</code>），条纹疏密清晰反映激波层与膨胀尾迹内的等视线积分密度梯度。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case3/interferogram/plt09078.png' | relative_url }}">
      <source src="{{ '/assets/video/case3/movie_interferogram.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值干涉动态演化视频（点击播放，支持暂停与进度拖动）。
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
