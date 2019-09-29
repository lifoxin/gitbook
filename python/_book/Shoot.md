```python

class Person(object):
	""""people"""
	def __init__(self,name):
		super(Person,self).__init__()
		self.name = name
		self.gun = None
		self.hp = 100

	def anzhuang_zidan(self,dan_jia_temp,zi_dan_temp):
		#弹夹，保存子弹（子弹）
		dan_jia_temp.baocun_zidan(zi_dan_temp)

	def anzhuang_danjia(self,gun_temp,dan_jia_temp):
		#安装弹夹
		gun_temp.baocun_danjia(dan_jia_temp)

	def naqiang(self,gun_temp):
		self.gun = gun_temp

	def __str__(self):
		if self.gun:
			return "%s的血量为：%d,他有枪, %s"%(self.name,self.hp,self.gun)
		else:
			if self.hp>0:
				return "%s的血量为%d,快挂了"%(self.name,self.hp)
			else:
				return "%s 已挂。。。"%self.name

	def kou_ban_ji(self,diren):
		# 发射子弹
		self.gun.fire(diren)

	def diao_xue(self, sha_shang_li):
		self.hp -= sha_shang_li

class Gun(object):
	""""枪类"""
	def __init__(self,name):
		super(Gun,self).__init__()
		self.name = name
		self.danjia = None
        
	def baocun_danjia(self,dan_jia_temp):
		self.danjia = dan_jia_temp
        
	def __str__(self):
		if self.danjia:
			return "枪的信息：%s,%s"%(self.name,self.danjia)
		else:
			return "枪的信息：%s,这把枪中没有弹夹" % (self.name)
        
	def fire(self,diren):
		#取出一发子弹
		zidan_temp = self.danjia.tanchu_zidan()
		# 伤害敌人
		if zidan_temp:
			zidan_temp.dazhong(diren)
		else:
			print("弹夹中没有子弹了。。。")

class Danjia(object):
	""""弹夹"""
	def __init__(self,max_num):
		super(Danjia,self).__init__()
		self.max_num = max_num
		self.zidan_list = [ ]

	def baocun_zidan(self,zi_dan_temp):
		self.zidan_list.append(zi_dan_temp)

	def __str__(self):
		return "弹夹的信息为： %d/%d"%(len(self.zidan_list),self.max_num)

	def tanchu_zidan(self):
		if self.zidan_list:
			return self.zidan_list.pop()
		else:
			return None

class Zidan(object):
	""""子弹"""
	def __init__(self,sha_shang_li):
		super(Zidan,self).__init__()
		self.sha_shang_li = sha_shang_li

	def dazhong(self,diren):
		diren.diao_xue(self.sha_shang_li)

def main():
	laowang = Person("老王")
	ak47 = Gun("ak47")
	dan_jia = Danjia(20)
    
    # 老王.安装子弹到弹夹中，安装15次子弹，子弹是杀伤力10，因为弹夹为0，不断添加子弹对象就可以添加子弹。
	for i in range(15):    
		zi_dan = Zidan(10)
		laowang.anzhuang_zidan(dan_jia,zi_dan)
	
	#老王.安装弹夹到枪中(枪，弹夹)
	laowang.anzhuang_danjia(ak47,dan_jia)
	
	#print(dan_jia)
	#print(ak47)

	#老王.拿枪（枪）
	laowang.naqiang(ak47)
    
	#创建一个敌人
	gebi_laosong = Person("隔壁老宋")

	#老王.扣扳机(隔壁老宋)
	for i in range(8):
		laowang.kou_ban_ji(gebi_laosong)
		print(gebi_laosong)

if	__name__ == "__main__":
	main()
```
