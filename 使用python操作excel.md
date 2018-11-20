---
title: 使用python操作excel
date: '2018-09-26 23:35:44'
tag: 
  - python
meta:
  -
    name: python 操作excel
    content: python 操作excel
---

今天学习一下python如何操作excel，并记录一下自己的学习过程，方便自己以后需要用到的时候进行查阅。
<!-- more -->

# python 简单操作excel

今天学习一下python如何操作excel，并记录一下自己的学习过程，方便自己以后需要用到的时候进行查阅。

## 依赖
> 1. openpyxl 操作excel的库
> 2. pillow对图片处理的库
> 3. mysqlclient 这个就不用多说了，操作mysql的库

## 学习过程
> 1. 创建excel
> 2. 插入数据
> 3. 数据公式
> 4. 设置文字
> 5. 插入图片
> 6. 合并单元格
> 7. 读取excel的数据并存储到数据库
> 8. 从mysql到excel

### 创建excel
```python
# 本篇都会在ExcelUtils类下进行编写
class ExcelUtils(object):
        def __init__(self):
        self.wb = Workbook()
        self.ws = self.wb.active
        # 创建表单
        self.ws_two = self.wb.create_sheet('我的表单')
        # 设置标题
        self.ws.title = '第一个表单'
        # 设置颜色
        self.ws.sheet_properties.tabColor = 'ff0000'
        self.ws_three = self.wb.create_sheet('我的其它表单')
```

### 对excel的操作
```python
   def do_sth(self):
        # 插入数据
        self.ws['A1'] = '日期'
        self.ws['A2'] = datetime.now()

        # 区域设置数据
        for row in self.ws_two['A1:E5']:
            for cell in row:
                cell.value = 2

        # 对数据进行求和
        self.ws_two['G1'] = '=SUM(A1:E1)'

        # 设置文字
        font = Font(sz=18, color=colors.RED)
        self.ws['A2'].font = font

        # 插入图片
        # img = Image('./static/y.jpg')
        # self.ws.add_image(img, 'B1')

        # 合并单元格
        self.ws.merge_cells('A4:E5')
        self.ws.unmerge_cells('A4:E5')
        self.wb.save('./static/test.xlsx')
```

### 读取excel的数据
```python
    def read_xls(self):
        """ 读取excel的数据 """
        ws = load_workbook('./static/template.xlsx')
        names = ws.get_sheet_names()
        print(names)
        conn = self.get_conn()

        wb = ws.active
        for (i, row) in enumerate(wb.rows):
            if i < 2:
                continue
            year = wb['A{0}'.format(i + 1)].value
            max = wb['B{0}'.format(i + 1)].value
            avg = wb['C{0}'.format(i + 1)].value
            if year is None:
                continue
            # 游标,只有拿到游标之后才可以进行sql语句操作
            cursor = conn.cursor()
            cursor.execute(
                'INSERT INTO `score`(`year`,`max`,`avg`) VALUES ({year},{max},{avg})'.format(year=year, max=max,
                                                                                             avg=avg))
            conn.autocommit(True)
```

### 读取mysql的数据到excel
```python
 def export_xls(self):
        """ 从mysql导出到xlsx """
        conn = self.get_conn()
        cursor = conn.cursor()
        sql = 'SELECT `year`,`max`,`avg` FROM `score`'
        cursor.execute(sql)
        rows = cursor.fetchall()
        wb = Workbook()
        ws = wb.active
        for (i, row) in enumerate(rows):
            ws['A{0}'.format(i + 1)] = row[0]
            ws['A{0}'.format(i + 1)] = row[1]
            ws['A{0}'.format(i + 1)] = row[2]
        wb.save('./static/export.xlsx')
```

### 获取数据库的连接
```python
    def get_conn(self):
        conn = MySQLdb.connect(
            db='use_grade',
            host='127.0.0.1',
            user='root',
            password='',
            charset='utf8'
        )
        return conn
```

