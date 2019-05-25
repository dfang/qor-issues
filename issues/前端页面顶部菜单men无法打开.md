# 前端页面顶部菜单项目 men、women、kids 打开空白的问题

products/views/gender.tmpl 有问题  
测试方法, 清空 gender.tmpl，刷新页面

如果修复，具体看你对面页面的需要，修改 gender.tmpl 就可以了

原因是因为数据库里Products表gender字段存的是Men、Kids、Women, 导致查询出来的是空数组
修复方法: 

将`utils.URLParam("gender", req)` 获取出来的首字母大写

app/products/handlers.go

```
tx.Where(&products.Product{Gender: strings.Title(utils.URLParam("gender", req))}).Preload("Category").Find(&Products)
```

