# zhejiangDEM
浙江区域数字地形高程模型（DEM）,提供raster（对应demZj.grd和demZj.gri）和GeoTiff（demZj.tif）两种格式数据。
另zjmap_With_Zhoushan.shp.rar是包含舟山群岛的shp格式的浙江区域地图

DEM的原始数据来自中国科学院计算机网络信息中心地理空间数据云平台(http://www.gscloud.cn)的SRTM3地形产品数据，已对该数据进行了聚合处理，将经纬度网格从原来的0.000833°*0.000833°降低到0.00826°*0.00806°(其中DEMZJ2为0.00833*0.00833）。
shp地图的原始数据来自Micaps自带的浙江区域地图，并拼接了舟山群岛的信息。

降低分辨率的R语言程序代码如下：
##########################################
#地理空间数据云 生成的浙江90m*90m DEM数据（GeoTiff格式）,fn的值即为对应的文件名（已经gscloud在线计算重设边界）
fn<-c( "srtm_60_06_20190829204613.tiff", "srtm_60_07_20190829204613.tiff", "srtm_61_06_20190829204614.tiff", "srtm_61_07_20190829204615.tiff") #区域所涉GeoTiff文件

demZj.grdsp<-GtiffUnion(fn,2, 2)
demZj.grdsp.ra<-raster(demZj.grdsp)
demZj.grdsp.ra<-aggregate(demZj.grdsp.ra, fact=10) #聚合处理，分辨率为原来的1/10

#raster格式保存，“*.grd、*.gri”
writeRaster(x=demZj.grdsp.ra, filename= "demZj2.raster", format="raster")
#GeoTiff格式保存  “*.tif”
writeRaster(x=demZj.grdsp.ra, filename= "demZj2", format="GTiff")

#############DEM拼接处理函数GtiffUnion  数据格式说明######
# gtiffs：GeoTiff格式(相同分辨率）文件名向量，方形区域(n列*m行
# gtiff文件 顺序：左上起，从上到下，从左到右（地理空间数据云平台默认方式）
#
#返回值是SpatialGridDataFrame对象
###########################################
GtiffUnion<-function(gtiffs,n,m){ 
   para.a<-NULL    #左下角Grid文件的网格参数
      f.mat<-NULL
      for(j in 1:m){  
          f.grd<-readGDAL(gtiffs[j])
          if(j==m ) #左下(i==0: i*m+j) 
              para.a<-gridparameters(f.grd)
          f.mat<-cbind(f.mat,as.matrix(f.grd))
      }
   demZ<-rbind(demZ, f.mat)
   Crss<-proj4string(f.grd) 

   for(i in 1:(n-1)){
      f.mat<-NULL
      for(j in 1:m){  
          f.grd<-readGDAL(gtiffs[i*m+j])
          f.mat<-cbind(f.mat,as.matrix(f.grd))
      }
      demZ<-rbind(demZ, f.mat)
    }
    demZ.dim<-dim(demZ)
    demZ.grd<-GridTopology(para.a$cellcentre.offset, para.a$cellsize, demZ.dim)
    #将demZ矩阵转成单列data.frame
    hh<-as.vector(demZ)
    demZ.grdsp<-SpatialGridDataFrame(grid=demZ.grd, 
        data=data.frame(hh=hh), proj4string = Crss)
    return(demZ.grdsp)
}

