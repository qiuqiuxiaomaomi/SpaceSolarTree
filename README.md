# SpaceSolarTree
基于Solar的空间搜索

<pre>
空间搜索原理
      1）对Point（经纬度）和其他的几何图形建索引
      2）根据距离排序
      3）根据矩形，圆形或者其他的几何形状过滤搜索结果

      在Solar中，空间搜索主要基于GeoHash和Cartesian Tiers 这两个概念来实现
</pre>


###对经纬度点（116.3906, 39.92324）区间编码

![](https://i.imgur.com/HcR3kQX.png)

![](https://i.imgur.com/hYRcmNO.png)

<pre>
GeoHash算法
      通过GeoHash算法，可以将经纬度的二维坐标变成一个可排序，可比较的字符串编码。
      在编码中的每个字符代表一个区域，并且前面的字符是后边字符的父区域。

      地球的维度区间是[-90, 90],经度区间是[180,180]

      经过上述计算，纬度产生的编码为：1011 1000 1100 0111 1001
                  经度产生的编码为：1101 0010 1100 0100 0100

      偶数位放经度，奇数位放纬度，把2串编码组合生成新串：
            11100 11101 00100 01111 00000 01101 01011 00001

      最后使用0-9, b-z(去掉a,i,l,o)32个字母进行base32编码，
      首先将
           11100 11101 00100 01111 00000 01101 01011 00001
      转成十进制 
           28，29，4，15，0，13，11，1，
      十进制对应的编码就是wx4g0ec1。同理，将编码转换成经纬度的解码算法与之相反，具体不再赘述。

      由上可知，字符串越长，表示的范围越精确，当GeoHash base32编码长度为8时，精度在19米
      左右，而当编码长度为9时，精度在2米左右，编码长度需要根据数据情况进行选择。不过从
      GeoHash的编码算法中可以看出它的一个缺点，位于边界两侧的点，虽然十分接近，但是编码
      完全不同，实际应用中，可以同时搜索该点所在区域的其他8个区域的点 ，即可解决该问题。
</pre>

笛卡尔层

![](https://i.imgur.com/RLCB3rB.png)

每层以2的平方递增，第一层4个网格，第二层16个网格

![](https://i.imgur.com/Owlb9yB.png)

<pre>
Cartesian Tiers笛卡尔层
      笛卡尔分层模型的思想是将经纬度转换成一个更大粒度的分层网络，该模型创建了很多的地理层
      ，每一层在前一层的基础上细化切分粒度，每一个网格被分配一个ID，代表一个地理位置。

      基于Cartesian Tier的搜索步骤为：
         1）根据Cartesian Tier层获得坐标点的地理位置gridId
         2) 与系统索引gridId匹配计算
         3）计算结果集与目标坐标点的距离返回特定范围内的结果集合

      使用笛卡尔层，能有效减少过滤范围，快速定位坐标点。
</pre>
