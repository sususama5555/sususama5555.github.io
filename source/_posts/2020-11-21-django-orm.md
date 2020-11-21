---
title: Django之ORM操作
date: 2020-11-21 03:07:42
tags: 
- Python
- Django
- ORM
- 数据库
categories: 
- Django
- ORM
---

# 查询类操作

## 查询所有的结果

相当sql中的select * from

```python
list = Test.objects.all()
```

## 条件查询filter相关sql中的where

用于过滤查询结果

```python
result = Test.objects.filter(id=1, name=’test’) # 传多个参数
```

如果多条件与查询，直接用逗号隔开，filter函数里面的参数都是Test Model中的字段

## 获取单个对象

get方法的参数一般为Model的主键，如果找不到会报错

```python
test_obj = Test.objects.get(id=1)
```

## 限制返回的结果数据的数量

相当于sql中的limit，其中order_by是用于排序，如果根据字段a倒序排序，就是order_by(“-time”)

```python
Test.objects.order_by('name')[0:2]
```

## 链式查询

```python
Test.objects.filter(name=’test’).order_by(“-ctime”)
```

## 多条件参数查询

传字典，构造查询条件

```python
data = Test.objects.filter(**query_dict).order_by(“-ctime”).values
```

其中query_dict为一个字典，key为条件字段，value为条件值

```python
query_dict = {'id':123,'name':’yyp’}
```

## 传Q对象

### 构造查询条件

1. 在 filter() 等函式中关键字参数彼此之间都是 “and” 关系。但是要执行更复杂的查询(比如，实现筛选条件的 or 关系)，可以使用 Q 对象。

2. Q对象包括 AND 关系和OR 关系

3. Q 对象可以用&和 | 运算符进行连接。当某个操作连接两个 Q 对象时，就会产生一个新的等价的 Q 对象。
    - 第一步，构造Q对象：
    ```python
    from django.db.models import Q
    Q(name__startswith=’h’) | Q(name__startswith=’p’)
    ```
    - 第二步，Q对象以查询参数方式使用，多个Q对象是and关系:
    ```python
    Test.objects.filter(
    Q(date=’2018-10-10 00:00:00’),
    Q(name__startswith=’h’) | Q(name__startswith=’p’)
    ```

> filter() 等函数可以接受 Q对象和条件参数，但Q对象必须放在条件参数前面 

### 传入条件查询 

```python
q1 = Q()
q1.connector = 'OR'              #连接方式
q1.children.append(('id', 1))
q1.children.append(('id', 2))
q1.children.append(('id', 3))

models.Tb1.objects.filter(q1)
```



### 合并条件查询

```python
con = Q()

q1 = Q()
q1.connector = 'OR'
q1.children.append(('id', 1))
q1.children.append(('id', 2))
q1.children.append(('id', 3))

q2 = Q()
q2.connector = 'OR'
q2.children.append(('status', '在线'))

con.add(q1, 'AND')
con.add(q2, 'AND')

models.Tb1.objects.filter(con)
```



## 过滤不满足条件的操作

```python
data = Test.objects.exclude(id=1)
```



# 增加类操作

## 新增一条记录

```python
test1 = Test(name='yyp')
test1.save()
```



# 更新类操作

## 先查询获取对象，再修改对象的值，再保存

```python
test1 = Test.objects.get(id=1)
test1.name = 'Google'
test1.save()
```



## 条件链式更新

```python
Test.objects.filter(id=1).update(name=‘Google’)
```



# 删除类操作

## 先查询获取要删除的对象，然后直接delete操作

```python
# 删除id=1的数据
test1 = Test.objects.get(id=1)
test1.delete()
```

## 条件删除：

```python
Test.objects.filter(id=1).delete()
```

## QuerySet相关

Django中model查询出来的结构类型为QuerySet，本质是一个查询对象集。

## 将多个查询结果转换为字典列表

```python
# all()方法查询出来的是QuerySet，用values方法转成字典集
data= Test.objects.all().values()
data_dict_list = list(data)
<QuerySet [<Test:  test>]>    ----><QuerySet [{“id”:XXX, “name”:XXX}]>
```



## QuerySet对象转换成字典对象

```python
fromdjango.forms.models import model_to_dict
data = Test.objects.get(id=1)
data_dict = model_to_dict(data)
```

## 序列化成json数据

对于很多web开发接口的时候，要返回的是json数据，而django从DB查询出来的是对象集，可以考虑django-rest-framework 库的serializers类，具体可参考：[**Tutorial 1: 序列化**](https://q1mi.github.io/Django-REST-framework-documentation/tutorial/1-serialization_zh/)

# 查询条件总结

## 字段名__op:

__exact 精确等于 like ‘aaa’

__iexact精确等于忽略大小写ilike‘aaa’

__contains 包含 like ‘%aaa%’

__icontains包含忽略大小写ilike‘%aaa%’，但是对于sqlite来说，contains的作用效果等同于icontains。

__gt大于

__gte大于等于

__lt小于

__lte小于等于

__in 存在于一个list范围内

__startswith以…开头

__istartswith以…开头忽略大小写

__endswith以…结尾

__iendswith以…结尾，忽略大小写

__range 在…范围内

__year 日期字段的年份

__month 日期字段的月份

__day 日期字段的日

__isnull=True/False

## 使用sql语句进行查询：

```python
fromdjango.db import connection
cursor = connection.cursor()
cursor.execute(“select * from Test where name = %s”, "yyp")
row = cursor.fetchone()
```



******

参考链接：
[【经验分享】Django开发中常用到的数据库操作总结](https://mp.weixin.qq.com/s/okaGAbNkFLhwZXvZV7eUBw) 