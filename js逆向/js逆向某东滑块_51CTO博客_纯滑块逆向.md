# js逆向某东滑块_51CTO博客_纯滑块逆向
#\-*\-encoding:utf-8\-*\-  
"""  
@File:trances.py  
@Contact:jyj345559953@qq.com  
@Author:Esword  
"""  
"""  
用于生成坐标轨迹  
"""  
importrandom  
importmatplotlib.pyplotasplt  
importnumpyasnp  
  
  
  
classGTrace(object):  
def\_\_init\_\_(self):  
self.\_\_pos\_x=\[\]  
self.\_\_pos\_y=\[\]  
self.\_\_pos\_z=\[\]  
  
def\_\_set\_pt_time(self):  
"""  
设置各节点的时间  
分析不同时间间隔中X坐标数量的占比  
统计结果:1.80%~90%的X坐标在15~20毫秒之间  
2.10%~15%在20~200及以上，其中\[-a,0,x,...\]这里x只有一个，取值在110~200之间  
坐标集最后3~5个坐标取值再50~400之间，最后一个坐标数值最大  
  
滑动总时间的取值规则:图片宽度260，去掉滑块的宽度剩下200;  
如果距离小于100，则耗时1300~1900之间  
如果距离大于100，则耗时1700~2100之间  
"""  
\_\_end\_pt_time=\[\]  
\_\_move\_pt_time=\[\]  
self.\_\_pos\_z=\[\]  
  
total\_move\_time=self.\_\_need\_time*random.uniform(0.8,0.9)  
start\_point\_time=random.uniform(110,200)  
\_\_start\_pt_time=\[0,0,int(start\_point\_time)\]  
  
sum\_move\_time=0  
  
\_tmp\_total\_move\_time=total\_move\_time  
whileTrue:  
delta_time=random.uniform(15,20)  
if\_tmp\_total\_move\_time<delta_time:  
break  
  
sum\_move\_time+=delta_time  
\_tmp\_total\_move\_time-=delta_time  
\_\_move\_pt\_time.append(int(start\_point\_time+sum\_move_time))  
  
last\_pt\_time=\_\_move\_pt_time\[-1\]  
\_\_move\_pt\_time.append(last\_pt\_time+\_tmp\_total\_move_time)  
  
sum\_end\_time=start\_point\_time+total\_move\_time  
other\_point\_time=self.\_\_need\_time-sum\_end\_time  
end\_first\_ptime=other\_point\_time/2  
  
whileTrue:  
delta_time=random.uniform(110,200)  
ifend\_first\_ptime-delta_time<=0:  
break  
  
end\_first\_ptime-=delta_time  
sum\_end\_time+=delta_time  
\_\_end\_pt\_time.append(int(sum\_end_time))  
  
\_\_end\_pt\_time.append(int(sum\_end_time+(other\_point\_time/2+end\_first\_ptime)))  
self.\_\_pos\_z.extend(\_\_start\_pt_time)  
self.\_\_pos\_z.extend(\_\_move\_pt_time)  
self.\_\_pos\_z.extend(\_\_end\_pt_time)  
  
def\_\_set\_distance(self,_dist):  
"""  
设置要生成的轨迹长度  
"""  
self.__distance=_dist  
  
if_dist<100:  
self.\_\_need\_time=int(random.uniform(500,1500))  
else:  
self.\_\_need\_time=int(random.uniform(1000,2000))  
  
def\_\_get\_pos_z(self):  
returnself.\_\_pos\_z  
  
def\_\_get\_pos_y(self):  
\_pos\_y=\[random.uniform(-40,-18),0\]  
point_count=len(self.\_\_pos\_z)  
x=np.linspace(-10,15,point_count-len(\_pos\_y))  
arct_y=np.arctan(x)  
  
for_,valinenumerate(arct_y):  
\_pos\_y.append(val)  
  
return\_pos\_y  
  
def\_\_get\_pos_x(self,_distance):  
"""  
绘制标准的数学函数图像:以tanh开始以arctan结尾  
根据此模型用等比时间差生成X坐标  
"""  
#first_val=random.uniform(-40,-18)  
#_distance+=first_val  
\_pos\_x=\[random.uniform(-40,-18),0\]  
self.\_\_set\_distance(_distance)  
self.\_\_set\_pt_time()  
  
point_count=len(self.\_\_pos\_z)  
x=np.linspace(-1,19,point\_count-len(\_pos_x))  
ss=np.arctan(x)  
th=np.tanh(x)  
  
foridxinrange(0,len(th)):  
ifth\[idx\]<ss\[idx\]:  
th\[idx\]=ss\[idx\]  
  
th+=1  
th*=(_distance/2.5)  
  
i=0  
start_idx=int(point_count/10)  
end_idx=int(point_count/50)  
delta_pt=abs(np.random.normal(scale=1.1,size=point\_count-start\_idx-end_idx))  
foridxinrange(start_idx,point_count):  
ifidx*1.3>len(delta_pt):  
break  
  
th\[idx\]+=delta_pt\[i\]  
i+=1  
  
\_pos\_x.extend(th)  
return\_pos\_x\[-1\],\_pos\_x  
  
defget\_mouse\_pos_path(self,distance):  
"""  
获取滑动滑块鼠标的滑动轨迹坐标集合  
"""  
result=\[\]  
_distance,x=self.\_\_get\_pos_x(distance)  
y=self.\_\_get\_pos_y()  
z=self.\_\_get\_pos_z()  
foridxinrange(len(x)):  
result.append(\[int(x\[idx\]),int(y\[idx\]),int(z\[idx\])\])  
importmatplotlib.pyplotasplt  
plt.plot(z,x)  
plt.show()  
returnint(_distance),result