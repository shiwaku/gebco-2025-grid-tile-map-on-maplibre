# 海底地形タイルの作成方法

## 使用データ

[GEBCO_2025 Grid (sub-ice topo/bathy) GeoTIFF](https://www.gebco.net/data-products/gridded-bathymetry-data)

## tif ファイル一覧作成

```
dir /b gebco_2025_sub_ice_*.tif > list.txt
```

## VRT 作成

```
gdalbuildvrt gebco_2025_sub_ice.vrt -input_file_list list.txt ^
  -resolution highest ^
  -r bilinear
```

## GeoTIFF 作成

```
gdalwarp gebco_2025_sub_ice.vrt gebco_2025_sub_ice_3857.tif \
  -s_srs EPSG:4326 -t_srs EPSG:3857 \
  -te_srs EPSG:4326 -te -180 -85.051129 180 85.051129 \
  -r bilinear -multi -wo NUM_THREADS=ALL_CPUS \
  -dstnodata -9999 -ot Float32 \
  -co TILED=YES -co COMPRESS=DEFLATE -co PREDICTOR=3 -co ZLEVEL=9 -co BIGTIFF=YES
```

## NoData を外す（rio rgbify でエラーになるので NoData を外す）

```
gdal_edit.py -unsetnodata gebco_2025_sub_ice_3857.tif
```

## Terrain‑RGB GeoTIFF 作成

```
rio rgbify -b -10000 -i 0.1 \
  gebco_2025_sub_ice_3857.tif gebco_2025_sub_ice_3857_terrainrgb.tif \
  --co BIGTIFF=YES --co TILED=YES --co COMPRESS=DEFLATE --co PREDICTOR=2 --co ZLEVEL=9
```

## ラスタータイル作成

```
gdal2tiles.py gebco_2025_sub_ice_3857_terrainrgb.tif gebco_2025 grid_tile_terrain-rgb -z0-9 --resampling=near --xyz --processes=6
```

## タイル URL

```
https://shiworks2.xsrv.jp/raster-tiles/gebco/gebco_2025 grid_tile_terrain-rgb/{z}/{x}/{y}.png
```

- ズームレベル：0 ～ 9
- タイルサイズ：256x256 ピクセル
