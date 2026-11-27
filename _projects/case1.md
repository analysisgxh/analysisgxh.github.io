---
layout: page
title: "Case 1: 火箭 100 km / Ma 27 再入仿真"
description: 60° 舵面极高马赫数强激波仿真、稀薄过渡流工况分析、近真空保护与流场演化展示
img: assets/img/case1/cover.png
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

火箭气动数值模拟项目 **Case 1**（100 km 卡门线高空、Mach 27 极高超声速、60° 气动减速舵面）的物理工况分栏分析、`inputs.used` 核心配置分栏详解及多物理场与数值纹影演化结果。

---

## 一、飞行工况分栏详解（100 km / Mach 27）

<div class="row justify-content-sm-center">
  <div class="col-sm-12 mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/case5_stl_views.png" title="火箭不同舵面展开角 STL 几何构型（0° / 30° / 60° / 90°）" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  火箭减速舵面四种展开角 STL 模型对比。Case 1 采用 60° 展开角构型 <code>model_v1_60.stl</code>，来流沿 <code>+y</code> 方向以 Mach 27 入射。
</div>

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 再入起始段自由来流参数</h4>
      <ul>
        <li><b>飞行高度与马赫数</b>：<code>100 km</code>，<code>prob.mach = 27.0</code></li>
        <li><b>来流静温与声速</b>：<code>prob.T_inf = 195.1 K</code>，对应声速 <code>c_inf ≈ 280 m/s</code></li>
        <li><b>来流速度与雷诺数</b>：<code>U_inf = 7560 m/s</code>，特征雷诺数 <code>Re ≈ 2.1 × 10³</code>（箭体长 6.5 m）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>2. 大气静压与状态方程自洽修正</h4>
      <ul>
        <li><b>来流密度</b>：<code>ρ_inf = 5.6 × 10⁻⁷ kg/m³</code>（CGS：<code>5.6 × 10⁻¹⁰ g/cm³</code>）</li>
        <li><b>自洽静压取值</b>：<code>prob.p_inf = 0.3136 dyn/cm²</code>（<code>3.136 × 10⁻⁵ kPa</code> / <code>0.03136 Pa</code>），动压 <code>q_inf ≈ 0.016 kPa</code></li>
        <li><b>修正原因</b>：原始简表近似值 <code>0.3 Pa</code> 与 100 km 标准大气密度不满足 <code>p = ρRT</code>，故采用状态方程严格自洽静压</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-warn">
      <h4>3. 稀薄过渡流与高温真实气体效应评估</h4>
      <ul>
        <li><b>努森数（Knudsen）</b>：<code>Kn = λ / D ≈ 0.06</code>（分子平均自由程达厘米量级，处于稀薄过渡流区，严格物理预测需采用 DSMC 方法）</li>
        <li><b>理想气体滞止高温</b>：无化学反应理想空气（<code>γ = 1.4</code>）下滞止温度达 <code>T_0 ≈ 2.86 × 10⁴ K</code></li>
        <li><b>算例定位</b>：检验嵌入式边界欧拉求解器在压比超 <code>500</code> 的极端强激波与背风近真空条件下的数值保正性与稳定性</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>4. 高波速时间尺度与稳态收敛特征</h4>
      <ul>
        <li><b>极短流经时间</b>：<code>15.36 m</code> 全计算域流经时间仅约 <code>2.0 ms</code>，入射激波约 <code>0.6 ms</code> 抵达鼻锥</li>
        <li><b>两阶段接力推进</b>：首阶段 GPU 算至 <code>chk01833</code>（<code>8.0 ms</code>），二阶段重启加密输出推进至 <code>15.35 ms</code>（<code>plt000187</code>，约 7.5 倍全域流经时间）</li>
        <li><b>气动力特征</b>：<code>t &gt; 6 ms</code> 后阻力系数曲线完全平坦，稳定于 <b><i>C<sub>d</sub></i> ≈ 1.9</b></li>
      </ul>
    </div>
  </div>
</div>

---

## 二、Case 1 算例配置分栏详解（`case1/inputs.used`）

`case1/inputs.used` 由 `tools/run_case.py` 组合 v4 `common.inputs` 与 `case1_100km_M27/inputs` 生成，采用 **L0 = 16 cm 紧凑计算域 + `tip` / `nose` 双分区 2 cm 局部加密**，核心配置分栏如下：

<div class="row cfg-grid">
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>1. 来流条件与静止气体启动（Free Stream &amp; Init）</h4>
      <ul>
        <li><b>CGS 来流输入</b>：<code>prob.mach = 27.0</code>，<code>prob.p_inf = 0.3136</code> dyn/cm²，<code>prob.T_inf = 195.1</code> K，流向 <code>+y</code>（<code>0. 1. 0.</code>）</li>
        <li><b>初始状态</b>：<code>prob.init_mode = 1</code>（计算域初始气体静止，由 <code>y = -384 cm</code> 入口边界注入 Mach 27 高速来流推动波系建立）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card cfg-card-accent">
      <h4>2. 极端强激波与近真空保护（Safeguards）</h4>
      <ul>
        <li><b>1 阶迎风重构</b>：<code>cns.plm_iorder = 1</code>（<code>plm_theta = 1.0</code>），<code>cns.cfl = 0.3</code>，层间通量重分配 <code>do_reflux = 1</code></li>
        <li><b>近真空防发散限幅</b>：开启密度下限 <code>cns.rho_floor = 5.6e-13</code> g/cm³（<code>10⁻³ ρ_inf</code>）与速度上限 <code>cns.vel_max = 1.2e6</code> cm/s（<code>1.6 U_inf</code>；若不开启约 11 步即在背风真空区发散）</li>
        <li><b>切割小单元保护</b>：<code>tiny_cell_vfrac = 0.2</code>，<code>T_floor = 10.0 K</code>，<code>repair_T_min = 20.0 K</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>3. 60° 舵面 STL 与 EB 几何构建（STL &amp; EB2）</h4>
      <ul>
        <li><b>几何加载与对中</b>：<code>eb2.geom_type = stl</code>，<code>eb2.stl_file = rocket.stl</code>（<code>model_v1_60.stl</code>），<code>stl_scale = 0.1</code>，<code>stl_center = 0. 40.91 0.</code></li>
        <li><b>EB 细网格参数</b>：<code>eb2.cover_multiple_cuts = 1</code>，<code>eb2.coarse_by_coarsening = 0</code>，<code>eb2.maxiter = 200</code>，<code>eb2.max_grid_size = 16</code></li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>4. 紧凑高超声速计算域与边界（Domain &amp; BCs）</h4>
      <ul>
        <li><b>紧凑计算域</b>：<code>x, z ∈ [-512, 512]</code> cm，<code>y ∈ [-384, 1152]</code> cm（物理尺寸 <code>10.24 × 15.36 × 10.24 m</code>，Mach 27 激波紧贴箭体且无扰动上传）</li>
        <li><b>基础网格划分</b>：<code>amr.n_cell = 64 96 64</code>，对应 L0 基础单元尺寸 <b>16 cm</b></li>
        <li><b>边界条件</b>：<code>cns.lo_bc = 2 1 2</code>（<code>y-low</code> 超声速入口 <code>1</code>），<code>cns.hi_bc = 2 2 2</code>（侧壁与出口零梯度外流 <code>2</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>5. AMR 四层网格与双分区加密（AMR &amp; Zones）</h4>
      <ul>
        <li><b>四层网格尺度</b>：<code>amr.max_level = 3</code>，对应分辨率 <b>L0–L3 = 16 / 8 / 4 / 2 cm</b>（<code>max_grid_size = 64</code>，<code>blocking_factor = 8</code>，<code>regrid_int = 4</code>，<code>grid_eff = 0.7</code>）</li>
        <li><b>鼻尖与鼻部分区（2 cm）</b>：<code>tip</code>（<code>[-16,16]×[-84,-44]×[-16,16]</code> cm）与 <code>nose</code>（<code>[-160,160]×[-144,40]×[-160,160]</code> cm）内 <code>fill_lev = cut_lev = shock_lev = 3</code></li>
        <li><b>全局表面与激波判据</b>：其余箭体/舵面表面 <code>refine_cutcells_max_lev = 2</code>（<b>4 cm</b>）；区外激波 <code>refine_shock_max_lev = 1</code>（<b>8 cm</b>，<code>pjump = 0.2</code>，<code>refine_shock_start = 6.0e-4 s</code>）</li>
      </ul>
    </div>
  </div>
  <div class="col-md-6 cfg-col">
    <div class="cfg-card">
      <h4>6. 断点续算与高频采样输出（Restart &amp; I/O）</h4>
      <ul>
        <li><b>首段计算设置</b>：<code>stop_time = 0.008 s</code>，<code>plot_per = 2.0e-4 s</code>，<code>check_per = 0.005 s</code></li>
        <li><b>重启加密输出</b>：由检查点 <code>chk01833</code> 重启推进至 <code>stop_time = 0.016 s</code>，输出间隔加密为 <code>amr.plot_per = 5.0e-5 s</code>（每 <code>0.05 ms</code> 一帧），<code>check_per = 0.002 s</code></li>
        <li><b>气动力监测</b>：<code>forces.int = 2</code>，实时输出 <code>forces.csv</code>（参考面积 <code>31415.93 cm²</code>）</li>
      </ul>
    </div>
  </div>
</div>

---

## 三、算例结果展示（Mach 27 极强激波流场与数值纹影演化）

以下展示推进至稳态阶段（`plt000185` / `15.25 ms`、`plt000186` / `15.30 ms`、`plt000187` / `15.35 ms`）的马赫数、压力、密度、温度及数值纹影关键帧与全程演化视频：

### 3.1 马赫数流场（Mach Number）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/mach/plt000185.png" title="马赫数流场 plt000185 (t = 15.25 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/mach/plt000186.png" title="马赫数流场 plt000186 (t = 15.30 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/mach/plt000187.png" title="马赫数流场 plt000187 (t = 15.35 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  马赫数分布关键帧对比（从左至右：<code>plt000185</code> / <code>15.25 ms</code>、<code>plt000186</code> / <code>15.30 ms</code>、<code>plt000187</code> / <code>15.35 ms</code>）：Mach 27 极高超声速下弓形激波高度贴体，底部曲线显示 <i>C<sub>d</sub></i> 在约 4 ms 后即平稳收敛。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case1/mach/plt000187.png' | relative_url }}">
      <source src="{{ '/assets/video/case1/movie_mach.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  马赫数流场与 <i>C<sub>d</sub></i> 同步演化视频（0 → 15.35 ms）。
</div>

### 3.2 压力流场（Pressure, $$ p / p_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/pressure/plt000185.png" title="压力流场 plt000185 (t = 15.25 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/pressure/plt000186.png" title="压力流场 plt000186 (t = 15.30 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/pressure/plt000187.png" title="压力流场 plt000187 (t = 15.35 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 对数色标分布对比（<code>15.25 ms</code> / <code>15.30 ms</code> / <code>15.35 ms</code>），鼻锥激波层与 60° 舵面迎风侧压比超过 500。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case1/pressure/plt000187.png' | relative_url }}">
      <source src="{{ '/assets/video/case1/movie_pressure.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲静压 <i>p / p<sub>&infin;</sub></i> 演化视频（0 → 15.35 ms）。
</div>

### 3.3 密度流场（Density, $$ \rho / \rho_\infty $$）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/density/plt000185.png" title="密度流场 plt000185 (t = 15.25 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/density/plt000186.png" title="密度流场 plt000186 (t = 15.30 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/density/plt000187.png" title="密度流场 plt000187 (t = 15.35 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 对数色标分布对比：弓形激波前缘强压缩层与舵面背风侧极低密度近真空阴影区对比鲜明。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case1/density/plt000187.png' | relative_url }}">
      <source src="{{ '/assets/video/case1/movie_density.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  无量纲密度 <i>&rho; / &rho;<sub>&infin;</sub></i> 演化视频（0 → 15.35 ms）。
</div>

### 3.4 温度流场（Temperature, K）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/temperature/plt000185.png" title="温度流场 plt000185 (t = 15.25 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/temperature/plt000186.png" title="温度流场 plt000186 (t = 15.30 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/temperature/plt000187.png" title="温度流场 plt000187 (t = 15.35 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  静温分布关键帧对比（<code>15.25 ms</code> / <code>15.30 ms</code> / <code>15.35 ms</code>）：鼻锥脱体激波层与 60° 舵面迎风再压缩区理想气体峰值温度约 30000 K。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case1/temperature/plt000187.png' | relative_url }}">
      <source src="{{ '/assets/video/case1/movie_temperature.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  静温场演化视频（0 → 15.35 ms）。
</div>

### 3.5 数值纹影（Numerical Schlieren）

<div class="row">
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/schlieren/plt000185.png" title="数值纹影 plt000185 (t = 15.25 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/schlieren/plt000186.png" title="数值纹影 plt000186 (t = 15.30 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/case1/schlieren/plt000187.png" title="数值纹影 plt000187 (t = 15.35 ms)" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>
<div class="caption">
  数值纹影关键帧对比：清晰显示鼻锥 <code>2 cm</code> 精细加密区内锐利的头部激波，以及舵面斜激波与多级 AMR 网格过渡特征。
</div>

<div class="row justify-content-sm-center mt-3">
  <div class="col-sm-8">
    <video class="img-fluid rounded z-depth-1" style="width: 100%;" controls loop muted playsinline preload="none" poster="{{ '/assets/img/case1/schlieren/plt000187.png' | relative_url }}">
      <source src="{{ '/assets/video/case1/movie_schlieren.mp4' | relative_url }}" type="video/mp4">
    </video>
  </div>
</div>
<div class="caption">
  数值纹影动态演化视频（0 → 15.35 ms）。
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
