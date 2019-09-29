**super()的使用**

super() 函数是用于调用父类(超类)的一个方法。

Python3.x 和 Python2.x 的一个区别是: Python 3 可以使用直接使用 super().xxx 代替 super(Class, self).xxx
```python
class A:
     def add(self, x):
         y = x+1
         print(y)
class B(A):
    def add(self, x):
        super().add(x)
b = B()
b.add(2)  # 3
```
**类方法、静态方法、实例方法**
```python
class C(object):
    def __init__(self):
		self.k = k

	@classmethod
	def x(cls):
		return cls.k
   
	@staticmethod
	def y():
		return self.k

	def z(self):
		return self.k
```
	@classmethod 类方法第一个参数cls
	类调用
	C.x(k)
	实例化调用
	c=C(k)
	c.x()

	@staticmethod 静态方法不需要参数
	类调用
	C.y(k)
	实例化调用
	c=C(k)
	c.y()

	实例方法，需要self参数，
	只有类实例化调用
	c=C(k)
	c.z()

**@property用法**

Python内置的@property装饰器就是负责把一个方法变成属性调用
```python
class C(object):
    def __init__(self):
        self._x = None
 
    @property
    def x(self):
        return self._x
 
    @x.setter
    def x(self, value):
        self._x = value
 
    @x.deleter
    def x(self):
        del self._x
```
	 c.x = 10  #触发setter
	 c.x       #触发getter，也就是返回self.x
	 del c.x   #触发deleter

```python
class Student(object):

    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2014 - self._birth
```
	birth是可读写属性，而age就是一个只读属性，因为age可以根据birth和当前时间计算出来。

**类的继承**
```python
class SchoolMember(object):
    '''学校成员基类'''
    member = 0
    def __init__(self, name, age, sex):
        self.name = name
        self.age = age
        self.sex = sex
        self.enroll()
    def enroll(self):
        '注册'
        print('just enrolled a new school member [%s].' % self.name)
        SchoolMember.member += 1
    def tell(self):
        print('----%s----' % self.name)
        for k, v in self.__dict__.items():
            print(k, v)
        print('----end-----')
    def __del__(self):
        print('开除了[%s]' % self.name)
        SchoolMember.member -= 1
    @classmethod   #类方法
    def aa(cls):
         cls.name
    @staticmethod  #静态方法
    def bb():
        pass

class Teacher(SchoolMember):
    '教师'
    def __init__(self, name, age, sex, salary, course):
        SchoolMember.__init__(self, name, age, sex)  
        #继承父类的构造方法，也就是调用父类schoolmember的属性，也可以写成：super(Teacher,self).__init__(name,age,sex)
		self.salary = salary
        self.course = course
    def teaching(self):
        print('Teacher [%s] is teaching [%s]' % (self.name, self.course))

class Student(SchoolMember):
    '学生'
    def __init__(self, name, age, sex, course, tuition):
        SchoolMember.__init__(self, name, age, sex)
        self.course = course
        self.tuition = tuition
        self.amount = 0
    def pay_tuition(self, amount):
        print('student [%s] has just paied [%s]' % (self.name, amount))
        self.amount += amount

t1 = Teacher('Wusir', 28, 'M', 3000, 'python')
t1.tell()
s1 = Student('haitao', 38, 'M', 'python', 30000)
s1.tell()
s2 = Student('lichuang', 12, 'M', 'python', 11000)
print(SchoolMember.member)
del s2
print(SchoolMember.member)
```
输出结果:
```
just enrolled a new school member [Wusir].
----Wusir----
name Wusir
age 28
sex M
salary 3000
course python
----end-----
just enrolled a new school member [haitao].
----haitao----
name haitao
age 38
sex M
course python
tuition 30000
amount 0
----end-----
just enrolled a new school member [lichuang].
3
开除了[lichuang]
2
开除了[Wusir]
开除了[haitao]
```
