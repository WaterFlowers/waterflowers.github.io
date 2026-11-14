---
home: true
type: "page"
disableAnchoredHeadings: true
---

&nbsp;
<div class="hero-divider"></div>


## Liangjing Ce: Historical-Geographic Spatial Data Science and Real-Time Decision System with C++ Parallel Optimization and RAG Enhancement

---
**两京智策 · 基于C++并行优化与RAG增强的历史地理空间数据科学与实时决策系统**


**HPC · Spatial Computing · RAG**

### Project Overview

This project constructs a high-performance spatial computing and data science system tailored for historical geography research. Centering on the core decision-making scenario of grain transport and storage optimization between Chang'an (the capital) and Luoyang (the eastern capital) of the Tang Dynasty, the system integrates multi-source spatial data and historical documents to establish a complete technical closed-loop spanning data acquisition, spatial analytical modeling, knowledge-augmented retrieval, and immersive visualization.   

In terms of system architecture, the project strictly adheres to the design principle of "separating heavy offline computation from real-time online services": a Python offline pipeline handles computation-intensive spatial analysis and knowledge base construction, while a C++ online backend undertakes high-concurrency, low-latency real-time dialogue and RAG retrieval tasks. To address the server's performance bottlenecks under concurrent scenarios, the project implements a fixed-size thread pool based on std::thread, std::mutex, and std::condition_variable, replacing the original dynamic thread creation approach using std::async, and validates the optimization results through perf flame graph analysis.   

This system is not merely a technical exploration in historical geography research, but also a comprehensive embodiment of high-performance computing, data science, and systems engineering.  

Core Objective of "Liangjing Ce"   

How can we optimize the granary site selection and transport routes between Chang'an and Luoyang, given the topographical, hydrological, and security constraints of the Tang Dynasty?   
This project realizes a complete technical pipeline: Raw Geographic Data → Modeling & Analysis → High-Performance Computing → Immersive 3D Visualization.   

>Integrated System Modules   

- Data Science (Spatial Modeling and Analysis)   
- High-Performance Computing (C++ Concurrency Optimization)   
- Retrieval-Augmented Generation (RAG)   
- Real-time Voice Interaction (UE5 C++ - Tencent ASR/TTS - DeepSeek API)   
- 3D Immersive Visualization (UE5 + Cesium)   

### 系统演示
<!-- ===== 视频封面（放在这里） ===== -->
{{< raw >}}
<a href="你的B站链接" target="_blank" class="video-preview" style="display:block; text-decoration:none; margin:1.5rem auto; max-width:1200px;">
  <div style="position:relative; width:100%; aspect-ratio:16/9; background:#0d1b2a; border-radius:12px; border:1px solid rgba(255,255,255,0.06); display:flex; flex-direction:column; align-items:center; justify-content:center; transition:all 0.3s ease;">
    <!-- 播放图标 -->
    <div style="width:64px; height:64px; border-radius:50%; border:2px solid rgba(255,255,255,0.15); display:flex; align-items:center; justify-content:center; margin-bottom:16px; transition:all 0.3s ease;">
      <span style="font-size:28px; color:rgba(255,255,255,0.7);">▶</span>
    </div>
    <!-- 提示文字 -->
    <span style="font-size:16px; color:rgba(255,255,255,0.5); font-family:'Inter',sans-serif; letter-spacing:0.5px;">Watch the system demo →</span>
  </div>
</a>
{{< /raw >}}
<!-- ===== 视频封面结束 ===== -->

### 系统架构

{{< raw >}}
<div style="width:calc(100vw - 8vw); position:relative; left:50%; transform:translateX(-50%); margin:1.5rem 0;">
  <img src="/images/arch.png" alt="系统架构图" 
       style="width:100%; display:block; border:none; border-radius:0; padding:0;">
</div>
{{< /raw >}}

### 核心成果

| 🚀 吞吐量提升 40% | ⚡ 检索延迟 < 200ms | 🏛️ 覆盖 200+ 实体 |
|:---:|:---:|:---:|


[View Project Details →](/projects/liangjing/)

---

## 📝 Technical Notes

Notes and Reflections on Project Development.

[View All Technical Notes →](/posts/)