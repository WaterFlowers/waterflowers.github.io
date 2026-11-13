---
title: "DEM Clipping Algorithm Overview"
date: 2026-03-05
draft: false
tags: ["DEM", "Clippingg", "c++"]
summary: "DEM Clipping Algorithm Overview"
---

## DEM Clipping Algorithm Overview

高程图的数据源来自地理空间数据云上，截取长安-洛阳之间的区域作为数据范围。地理空间数据云上框选了“长安洛阳这部分”，下载到的瓦片拼接后，实际范围仍然比你需要的要大得多。 这是因为瓦片是按固定的图幅分幅提供的，不得不下载覆盖更大区域的多个瓦片才能拼出完整的走廊。
在计算坡度构建成本面的时候，数据量过大导致pc内存爆满而无法计算的情况；为了缩小计算量，必须对原高程图进行缩减也裁剪。
原合并后的数组形状是 (10801, 28801)，即 10801 行（纬度方向）、28801 列（经度方向），这个范围基本覆盖了东西从陕西区域到山东区域，覆盖过大。
长安（西安）到洛阳的漕运走廊，大致范围是：
```python
经度：107.5°E 到 113.5°E（跨度约 6°，约 550 公里）
纬度：33.5°N 到 35.5°N（跨度约 2°，约 220 公里）
```
用这个范围裁剪后的 DEM，大小约为：
总像素比原来减少一半多，占用也也会大幅下降，计算时间也会大幅缩短。
如果裁剪后的计算量还是超过了内存承载，则进一步缩小到只关注两个城市周边（如长安周边 100km，洛阳周边 100km），数据量会更小。
```python
import rasterio
from rasterio.windows import from_bounds
# 精确的研究区范围
xmin, ymin, xmax, ymax = 107.5, 33.5, 113.5, 35.5
with rasterio.open('changan_luoyang_merged.tif') as src:
    window = from_bounds(xmin, ymin, xmax, ymax, src.transform)
    dem = src.read(1, window=window).astype('float32')
    transform = src.window_transform(window)
    profile = src.profile
profile.update({
    'height': dem.shape[0],
    'width': dem.shape[1],
    'transform': transform,
    'dtype': 'float32'
})
with rasterio.open('changan_luoyang_corridor.tif', 'w', **profile) as dst:
    dst.write(dem, 1)
print(f"裁剪后尺寸: {dem.shape}")
```





