# 面向对象

- 封装

  - 组装： 将数据和操作组装到一起

  - 隐藏数据： 对外只暴露一些借口，通过借口访问对象。

    比如开车，司机不需要了解车的构造，只需要知道使用什么部件怎么驾驶就行。

- 继承

  - 多复用，继承来的就不需要重复写
  - 多继承，少修改。 OCP（open-closed principle），使用继承来改变，体现个性。

- 多态

  - 面向对象编程最灵活的地方，动态绑定

- 栗子

  ```python
  # 使用 class 创建一个类
  class Student(object):
      # 类属性
      _name = ''
      _age = 0

      # init方法， 在创建一个实例的时候会执行
      def __init__(self, name, age):  # self是默认的第一个参数，调用时会把当前实例传入
          self._name = name
          self._age = age

      # 存取器
      # 使用装饰器 property 表示这是一个存取器函数， 可以只有get（只读）
      @property
      def age(self):
          return self._age

      # 使用装饰器 @xxx.setter
      @age.setter
      def age(self, num):
          self._age = num

      # 静态方法
      @staticmethod
      def staSay():
          print('i can speak')

      # 类方法
      @classmethod
      def alive(cla):  # 传入的cla参数 是这个类
          print('i am {}'.format(cla.__name__))
          print('i am still alive')




  # 实例化
  tom = Student('tom', 10)  # 实例化一个对象,  这个时候会调用 __init__

  # 通过属性 读写
  print(tom._name)
  tom._name = 'bigTom'

  # 通过存取器 读写
  print(tom.age)
  tom.age = 18

  # 类方法
  Student.alive()

  # 静态方法
  Student.staSay()


  print(tom._name,  tom.age)


  ```
