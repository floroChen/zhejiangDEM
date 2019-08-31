# zhejiangDEM
浙江区域数字地形高程模型（DEM）,提供raster（对应demZj.grd和demZj.gri）和GeoTiff（demZj.tif）两种格式数据。
另zjmap_With_Zhoushan.shp.rar是包含舟山群岛的shp格式的浙江区域地图

DEM的原始数据来自中国科学院计算机网络信息中心地理空间数据云平台(http://www.gscloud.cn)的SRTM3地形产品数据，已对该数据进行了聚合处理，将分辨率降低了9倍，经纬度网格从原来的0.000833°*0.000833°降低到0.00826°*0.00806°。
shp地图的原始数据来自Micaps自带的浙江区域地图，并拼接了舟山群岛的信息。

降低分辨率的R语言程序代码如下：
##########################################
#地理空间数据云 生成的浙江90m*90m DEM数据（GeoTiff格式）,fn的值即为对应的文件名（已经gscloud在线计算重设边界）
fn<-c( "srtm_60_06_20190829204613.tiff", "srtm_60_07_20190829204613.tiff", "srtm_61_06_20190829204614.tiff", "srtm_61_07_20190829204615.tiff") #区域所涉GeoTiff文件
demZj.sppts<-readGDAL(fn[1]) %>%  #读取GeoTiff文件
      raster(.) %>%  #栅格化
      aggregate(., fact=9) %>%  #降低分辨率（聚合 9倍）
      rasterToPoints(. , spatial=T )  
for(f in fn[-1]){
   a<-readGDAL(f) %>%  #读取GeoTiff文件
      raster(.) %>%  #栅格化
      aggregate(., fact=9) %>%  #降低分辨率（聚合 9倍）
      rasterToPoints(. , spatial=T ) 
   demZj.sppts<spRbind(demZj.sppts, a)  #空间单元拼合
}
