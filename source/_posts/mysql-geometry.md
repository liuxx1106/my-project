---
title: MySQL空间数据操作及MyBatisPlus接入
tags: [mybatisplus, mysql, geometry]
categories: [mysql]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/mysql-geometry.jpg    # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

### 前言

最近接手同事做了一半的项目，其中有个需求是元数据入库需要存储空间数据，支持空间查询。
当前项目后台使用的是`SpringBoot` + `MyBatisPlus` + `MySQL`, 前端使用的是vue2。
由于之前做空间数据都是使用的`PostgreSql`, 它有着强大空间数据处理能力，且项目中使用也很方便。
潜意识里认为`MySQL`不支持空间数据，没想到一查资料居然是支持的，于是开始了连续几个小时的踩坑，终于搞定，整理成完成路线供大家参考。

### 1. MySQL空间数据

MySQL为空间数据存储及处理提供了专用的类型geometry(支持所有的空间结构)，还有有细分类型Point, LineString,
Polygon, MultiPoint, MultiLineString, MultiPolygon等等，同时提供了大量的空间函数支持空间运算和查询。

### 2. GeoJson介绍

GeoJSON是一种对各种地理数据结构进行编码的格式。GeoJSON对象可以表示几何、特征或者特征集合。GeoJSON支持下面几何类型：

点、线、面、多点、多线、多面和几何集合。

GeoJSON里的特征包含一个几何对象和其他属性，特征集合表示一系列特征，一个完整的GeoJSON数据结构总是一个（JSON术语里的）对象。
以下为常见示例：

```java
{
    "type": "FeatureCollection",
    "features": [{
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [102.0, 0.5]
            },
            "properties": {
                "prop0": "value0"
            }
        }, {
            "type": "Feature",
            "geometry": {
                "type": "LineString",
                "coordinates": [[102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]]
            },
            "properties": {
                "prop0": "value0",
                "prop1": 0.0
            }
        }, {
            "type": "Feature",
            "geometry": {
                "type": "Polygon",
                "coordinates": [[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]]
            },
            "properties": {
                "prop0": "value0",
                "prop1": {
                    "this": "that"
                }
            }
        }
    ]
}
```

### 3.MySql格式化空间数据类型（geometry相互转换geojson）

MySQL提供了几个空间函数用来解析及格式化空间数据，geojson是gis空间数据展示的标准格式，前端地图框架及后端空间分析相关框架都会支持geojson格式。

|  转换   | 空间函数  |
|  ----  | ----  |
| geojson -> geometry  | ST_GeomFromGeoJSON |
| geometry -> geojson   | ST_ASGEOJSON |
| geometry(字符串) -> geometry  | ST_GEOMFROMTEXT |

- eg1. 查询示例

```mysql
select id,point_name,ST_ASGEOJSON(point_geom) as geojson from meteorological_point where id = 1
```

- eg2. 新增示例

```mysql
insert into meteorological_point(point_name, point_geom) values("新帅集团监测点", ST_GEOMFROMTEXT("POINT(117.420671499 40.194914201)"))
```

### 4. SpringBoot + MyBatisPlus + MySQL 集成空间数据实战

在我们的Java项目中操作空间数据一般有两种方式：

- 使用上面的`ST_ASGEOJSON`,`ST_GEOMFROMTEXT`等方法用原生sql直接操作数据；
- 使用MyBatisPlus中的typeHandler在切面层做类型转换；

因为我们的项目已经集成了MyBatisPlus且集成度较高，而且切面处理更为优雅便捷，所以这里使用第二种。

#### 4.1 实现GeometryTypeHandler.class工具类

```java
package org.jeecg.config.mybatis;

import com.vividsolutions.jts.geom.Geometry;
import com.vividsolutions.jts.geom.GeometryFactory;
import com.vividsolutions.jts.geom.PrecisionModel;
import com.vividsolutions.jts.io.*;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;
import org.apache.ibatis.type.MappedTypes;

import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@MappedTypes({String.class})
@MappedJdbcTypes({JdbcType.OTHER})
public class GeometryTypeHandler extends BaseTypeHandler<String> {


    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, String s, JdbcType jdbcType) throws SQLException {


        Geometry geo = null;
        try{
            //String转Geometry
            geo = new WKTReader(new GeometryFactory(new PrecisionModel())).read(s);
            // Geometry转WKB
            byte[] geometryBytes = new WKBWriter(2, ByteOrderValues.LITTLE_ENDIAN, false).write(geo);
            // 设置SRID为mysql默认的 0
            byte[] wkb = new byte[geometryBytes.length+4];
            wkb[0] = wkb[1] = wkb[2] = wkb[3] = 0;
            System.arraycopy(geometryBytes, 0, wkb, 4, geometryBytes.length);
            preparedStatement.setBytes(i,wkb);
        }catch (ParseException e){

        }
    }


    @Override
    public String getNullableResult(ResultSet resultSet, String s){
        try(InputStream inputStream = resultSet.getBinaryStream(s)){
            Geometry geo = getGeometryFromInputStream(inputStream);
            if(geo != null){
                return geo.toString();
            }
        }catch(Exception e){

        }
        return null;
    }

    @Override
    public String getNullableResult(ResultSet resultSet, int i){
        try(InputStream inputStream = resultSet.getBinaryStream(i)){
            Geometry geo = getGeometryFromInputStream(inputStream);
            if(geo != null){
                return geo.toString();
            }
        }catch(Exception e){

        }
        return null;
    }

    @Override
    public String getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return "";
    }
    /**
     * 流 转 geometry
     * */
    private  Geometry getGeometryFromInputStream(InputStream inputStream) throws Exception {

        Geometry dbGeometry = null;

        if (inputStream != null) {
            // 二进制流转成字节数组
            byte[] buffer = new byte[255];

            int bytesRead = 0;
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesRead);
            }
            // 得到字节数组
            byte[] geometryAsBytes = baos.toByteArray();
            // 字节数组小于5 异常
            if (geometryAsBytes.length < 5) {

            }

            //字节数组前4个字节表示srid 去掉
            byte[] sridBytes = new byte[4];
            System.arraycopy(geometryAsBytes, 0, sridBytes, 0, 4);
            boolean bigEndian = (geometryAsBytes[4] == 0x00);
            // 解析srid
            int srid = 0;
            if (bigEndian) {
                for (int i = 0; i < sridBytes.length; i++) {
                    srid = (srid << 8) + (sridBytes[i] & 0xff);
                }
            } else {
                for (int i = 0; i < sridBytes.length; i++) {
                    srid += (sridBytes[i] & 0xff) << (8 * i);
                }
            }

            WKBReader wkbReader = new WKBReader();
            // WKBReader 把字节数组转成geometry对象。
            byte[] wkb = new byte[geometryAsBytes.length - 4];
            System.arraycopy(geometryAsBytes, 4, wkb, 0, wkb.length);
            dbGeometry = wkbReader.read(wkb);
            dbGeometry.setSRID(srid);
        }
        return dbGeometry;
    }
}
```

#### 4.2 实体类字段上加类型转换注解

```java
@ApiModelProperty(value = "空间点位")
@TableField(typeHandler = GeometryTypeHandler.class)
private String geomPoint;
```

此处有坑，还需在实体类加`autoResultMap = true`注解

```java
@TableName(value = "f_metadata",autoResultMap = true)
public class Metadata extends JeecgEntity {
    //省略
}
```

> 由于能查到的资料很有限，这里卡了半天，中间还尝试了MySQL方言 + Geometry格式化注解的方式,失败了
> database-platform: org.hibernatespatial.mysql.MySQLSpatialDialect
> @JsonSerialize(using = GeometrySerializer.class)
> @JsonDeserialize(using = GeometryDeserializer.class)
> 此方法在postgresql中可用

至此，我们的后台服务已经可以支持基本的增加、分页查询操作，空间数据可以完成自动转换。

#### 4.3 MyBatisPlus的QueryWrapper构造空间查询

我们在项目开发中除了基本的分页查询，还要用到区域检索，而前面提到MySQL提供了大量的空间函数去支持空间查询。
比如：

- MBRContains(A,B) –> A包含B
- MBRWithin(A,B) –> A在B中

*文末附MySQL空间处理函数和方法*

所以，我们可以利用相关函数去做区域查询，如：

```java
select * from f_metadata where MBRContains(ST_GeomFromText('Polygon((89.76 41.31,125.36 44.56,117.07 23.29,92.33 23.39,89.76 41.31))'),geom_point)
```

该sql语句可以检索到geom_point在此闭合区域内的所有数据。

现在我们需要考虑下QueryWrapper有没有MBRContains方法，查了下果然没有，那么怎么用QueryWrapper来构造空间查询呢？

> 通过查看QueryWrapper的所有内部方法，我发现了exist函数：

```java
    default Children exists(String existsSql) {
        return this.exists(true, existsSql);
    }
```

它可以传入原生sql语句，然后先执行外层查询，再用结果去匹配是否存在于exists内，如果为true，则作为结果返回。
先在数据库写了下，没问题。

```java
select * from f_metadata where EXISTS(select* from (select * from f_metadata where Contains(ST_GeomFromText('Polygon((89.76 41.31,125.36 44.56,117.07 23.29,92.33 23.39,89.76 41.31))'),geom_point)) as b where f_metadata.id = b.id  );
```

后台代码中的QueryWrapper构造：

```java
    // ......
    if (ObjectUtil.isNotEmpty(metadata.getGeomPoint())) {
            queryWrapper.exists("select * from (select * from f_metadata where Contains(ST_GeomFromText('" + metadata.getGeomPoint() + "'),geom_point)) as b where f_metadata.id = b.id");
    }
    queryWrapper.lambda().between(ObjectUtil.isNotEmpty(metadata.getStartTime()) && ObjectUtil.isNotEmpty(metadata.getEndTime()), Metadata::getCreateTime, metadata.getStartTime(), metadata.getEndTime());
    queryWrapper.lambda().orderByDesc(Metadata::getCreateTime);
    return queryWrapper;
```

搞定！

### 5. 附：MySQL空间数据处理函数和方法

#### 空间查询函数

- 包含相关
MBRContains(A,B) –> A包含B
MBRWithin(A,B) –> A在B中
注意:包含关系中，所要验证的集合必须全部包含在指定的集合中。如果只有部分在其中，应该使用相交

- 覆盖相关
MBRCoveredBy(A,B) –> A被B覆盖
MBRCovers(A,B) –> A覆盖B

- 相交相关
MBRDisjoint(A,B) –> A、B不相交
MBRIntersects(A,B) –> A、B相交

- 接触
MBRTouches(A,B) –> A、B接触，接触的概念类似于相切

- 重叠
MBROverlaps(A,B) –> A、B重叠

- 相同
MBREquals(A,B) –> A、B相同

#### 空间数据相关方法

- 点独有
开始、结束点
ST_StartPoint(A)
ST_EndPoint(A)

- 获取x或y
ST_X(A)
ST_Y(A)

- 凸包
ST_ConvexHull(A) –> 多点A的凸包面

- 返回矩形
ST_MakeEnvelope(A,B) –> A、B为对角点

- 线独有
  - 线是否闭合
    ST_IsClosed(A)
  - 线中点数量
    ST_NumPoints
  - 线中第n个点
    ST_PointN(A,n)
  - 线长度
    ST_Length(A)
  - 生成矩形
    ST_Envelope(A) –> A只有两个点，且不是水平或竖直线

- 面积
    ST_Area(A)

  - 面的内外边界
    ST_ExteriorRing(A) –> 获取A面外环边界，返回值为LineString
    ST_InteriorRingN(A,num) –> 获取A面中第num个内环边界，返回值为LineString。num从1开始。
    ST_NumInteriorRings(A) –> 获取A面内环数量(5.7.8后添加ST_NumInteriorRing，效果一样)
    部分geo对象可用

- 集合
  - 交集
    ST_Intersection(A,B)
  - 异或
    ST_SymDifference(A,B) –> A、B中独有的
  - 并集
    ST_Union(A,B)
  - 质心
    ST_Centroid(A)
  - 距离
    ST_Distance(A,B) –> A和B距离
    ST_Distance_Sphere(A,B) –> A和B的球面距离
  - 不同
    ST_Difference(A,B) –> 返回A中有B中没有的
  - 抽稀
    ST_Simplify(A,mix_distance) –> 将A抽稀，简化A中两点距离小于max的值(用起来有点迷。。待研究)

- 缓冲区
ST_Buffer(A,length) –> 通过A几何体，生成他周边范围为length距离的面

5.7.7后可以添加策略影响缓冲区的计算,设置的语句是ST_Buffer_Strategy()

- point策略
point_circle –> 点的缓冲区是一个圆(默认)
point_square –> 点的缓冲区是一个正方形，length是点到其中一边的距离

- join策略
join_round –> 连接处缓冲区边界为圆弧(默认)
join_miter –> 连接处缓冲区边界为尖角

- end策略
end_round –> 在结束处缓冲区为圆弧(默认)
end_flat –> 在结束处缓冲区为平坦的直线

- 举例生成缓冲区
ST_Buffer(point, 5, ST_Buffer_Strategy('point_square'))
ST_Buffer(line, 5, ST_Buffer_Strategy('join_miter', 10), ST_Buffer_Strategy('end_flat'))

- 相交
ST_Intersects(A,B) –> A和B是否相交
ST_Crosses(A,B) –> A和B是否相交(相交部分不等于A或B)
ST_Disjoint(A,B) –> A和B是否不相交

- 重叠
ST_Overlaps(A,B)

- 接触
ST_Touches(A,B)

- 包含
ST_Contains(A,B) –> A是否包含B
ST_Within(A,B) –> A是否在B中

- 验证数据是否合法
ST_IsValid(A)
ST_Validate(A)

- geo对象返回格式
ST_AsText(字段名) –> 以文本形式返回
ST_AsBinary(字段名) –> 以二进制形式返回

*注意:每个方法前的MBR、ST可要可不要，在mysql5.7.6之后，不带MBR、ST的方法开始弃用*
