# Orekit

作为航天动力学库，[Orekit](https://link.juejin.cn?target=https%3A%2F%2Fwww.orekit.org%2F) 最基本的功能性算法就是对航天器状态矢量进行预报，该功能由 `org.orekit.propagation` 包提供。[Orekit](https://link.juejin.cn?target=https%3A%2F%2Fwww.orekit.org%2F) 有三种预报方式，一是解析法预报，由 `org.orekit.propagation.analytical` 实现；二是数值法预报，由 `org.orekit.propagation.numerical` 和 `org.orekit.propagation.integration` 实现；三是半解析法预报，由 `org.orekit.propagation.semianalytical.dsst` 实现。



解析法预报速度快，但精度随时间下降快，且需要平根或拟平根作为预报输入，即需要事先定轨；数值法通过求解轨道动力学方程实现，模型精确的情况下，预报精度高，但由于必须递推中间过程量，故需消耗较多的计算资源；半解析法速度和精度都介于解析法和数值法之间，一般用于卫星轨道寿命与离轨分析

**经典开普勒轨道描述：** `KeplerianOrbit` *1).* **a**：轨道半场轴（m） *2).* **e**：轨道偏心率（NAN） *3).* **i**：轨道倾角（rad） *4).* **ω**：近地点幅角（rad） *5).* **Ω**：升交点赤经（rad） *6).* **ν**，E or M：真近角、偏近角或平近角（rad）

**圆轨道描述：** `CircularOrbit` *1).* **a**：轨道半场轴（m） *2).* **ex**：偏心率矢量 X 分量（NAN），ex=e×cos(ω) *3).* **ey**：偏心率矢量 Y 分量（NAN），ey=e×sin(ω) *4).* **i**：轨道倾角（rad） *5).* **Ω**：升交点赤经（rad） *6).* **uν**，uE or uM：真纬度幅角（ω+ν）、偏纬度幅角（ω+E）和平纬度幅角（ω+M）（rad）

**赤道轨道：** `EquinoctialOrbit` *1).* **a**：轨道半场轴（m） *2).* **ex**：偏心率矢量 X 分量（NAN），ex=e×cos(ω) *3).* **ey**：偏心率矢量 Y 分量（NAN），ey=e×sin(ω) *4).* **ix**：倾角矢量 X 分量（NAN），hx=tan(i/2)×cos(Ω) *5).* **iy**：倾角矢量 Y 分量（NAN），hy=tan(i/2)×sin(Ω) *5).* **Ω**：升交点赤经（rad） *6).* **lν**，lE or lM：真纬度幅角（ω+ν+Ω）、偏纬度幅角（ω+E+Ω）和平纬度幅角（ω+M+Ω）（rad）

**三维状态矢量：** `CartesianOrbit` *1).* **X,Y,Z**：位置矢量（m） *2).* **Vx,Vy,Vz**：速度矢量（m/s）

