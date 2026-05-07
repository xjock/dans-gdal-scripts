dans-gdal-scripts
=================

一套与 GDAL 配合使用的实用工具集，专注于栅格数据的轮廓提取、对比度拉伸、波段合并等操作。

## 工具列表

| 工具 | 说明 |
|---------|-------------|
| `gdal_contrast_stretch` | 对比度拉伸，支持 8/16 位或浮点图像转 8 位 |
| `gdal_dem2rgb` | 从 DEM 生成山体阴影彩色图像 |
| `gdal_get_projected_bounds` | 投影多边形并返回其外接矩形 |
| `gdal_landsat_pansharp` | 全色锐化，对 Landsat 7 图像效果最佳 |
| `gdal_list_corners` | 以 YAML 格式输出栅格地理编码信息（类似 gdalinfo，但输出 YAML） |
| `gdal_make_ndv_mask` | 生成无数据值（NDV）像素掩膜 |
| `gdal_merge_simple` | 将单个波段合并为单个 GeoTIFF 图像（仅 8 位） |
| `gdal_merge_vrt` | 将单个波段合并为单个 VRT 图像 |
| `gdal_raw2geotiff` | 将原始二进制文件转换为 GeoTIFF |
| `gdal_trace_outline` | 追踪图像轮廓并生成 WKT 或 Shapefile。支持边缘清理、特征分类、Douglas-Peucker 简化和坐标系转换 |
| `gdal_wkt_to_mask` | 根据多边形生成位图掩膜 |

## 依赖

- **CMake** >= 3.16
- **GDAL** >= 2.0（要求 `gdal-config` 在 PATH 中）
- **Boost** >= 1.37（仅头文件）
- 支持 C++11 的编译器

## 从源码构建

```bash
cmake -B build -S .
cmake --build build -j$(nproc)
```

指定安装前缀：

```bash
cmake -B build -S . -DCMAKE_INSTALL_PREFIX=/your/favorite/prefix
cmake --build build -j$(nproc)
cmake --install build
```

## 运行测试

```bash
cd build
ctest --output-on-failure
```

## Debian / Ubuntu

```bash
apt-get install dans-gdal-scripts
```

## macOS (Homebrew)

```bash
brew install gdal boost cmake
```

然后从源码构建即可。

## 致谢

本工具集由阿拉斯加地理信息网络（GINA）的 Dan Stahlke 编写。
如有问题或建议，请发送至 [dan@stahlke.org](mailto:dan@stahlke.org)。
