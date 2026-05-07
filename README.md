dans-gdal-scripts
=================

This package consists of a number of utilities for use in conjunction with
GDAL.

## Programs

| Program | Description |
|---------|-------------|
| `gdal_contrast_stretch` | Contrast stretch and conversion from 8/16-bit or floating point to 8-bit |
| `gdal_dem2rgb` | Generate hillshaded images from DEMs |
| `gdal_get_projected_bounds` | Project a polygon and return its bounding rectangle |
| `gdal_landsat_pansharp` | Pansharpening - works best for Landsat 7 images |
| `gdal_list_corners` | Print raster geocode information in YAML format (similar to gdalinfo but gives YAML) |
| `gdal_make_ndv_mask` | Generate a mask of No-Data-Value (NDV) pixels |
| `gdal_merge_simple` | Merge individual bands into a single GeoTIFF image (8-bit only) |
| `gdal_merge_vrt` | Merge individual bands into a single VRT image |
| `gdal_raw2geotiff` | Convert raw binary files into GeoTIFFs |
| `gdal_trace_outline` | Trace the outline of an image and generate WKT or Shapefile. Supports edge cleanup, feature classification, Douglas-Peucker simplification, and coordinate system transforms |
| `gdal_wkt_to_mask` | Generate a bitmap of the area covered by a polygon |

## Dependencies

- **CMake** >= 3.16
- **GDAL** >= 2.0 (with `gdal-config` in PATH)
- **Boost** >= 1.37 (headers only)
- C++11 compatible compiler

## Build from source

```bash
cmake -B build -S .
cmake --build build -j$(nproc)
```

To specify an install prefix:

```bash
cmake -B build -S . -DCMAKE_INSTALL_PREFIX=/your/favorite/prefix
cmake --build build -j$(nproc)
cmake --install build
```

## Run tests

```bash
cd build
ctest --output-on-failure
```

## Credits

These programs were written by Dan Stahlke of the Geographic Information Network of Alaska.
Send questions or comments to [guoruiuser@163.com](mailto:guoruiuser@163.com).
