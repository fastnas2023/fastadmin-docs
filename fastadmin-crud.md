# FastAdmin CRUD操作文档

FastAdmin框架提供了强大的CRUD（Create, Read, Update, Delete）一键生成功能，可以根据数据库表结构自动生成控制器、模型、视图和JS文件，大大提高开发效率。本文将详细介绍FastAdmin CRUD的生成与使用方法。

## CRUD概述

CRUD操作是Web应用开发中最常见的基础功能，包括：

- **Create**：创建（添加）数据
- **Read**：读取（查询）数据
- **Update**：更新（编辑）数据
- **Delete**：删除数据

FastAdmin通过命令行工具，可以快速生成这些功能的代码框架，开发者只需专注于业务逻辑的实现。

## CRUD生成命令

### 基本命令

```bash
php think crud -t 表名
```

这个命令会根据指定的数据库表，生成对应的控制器、模型、验证器、视图和JS文件。

### 完整命令格式

```bash
php think crud --table=表名 --controller=控制器名 --model=模型名 --fields=字段列表 --force=强制覆盖 --relation=关联表 --relationmode=关联模式 --relationforeignkey=外键 --relationprimarykey=主键 --relationfields=关联显示字段
```

### 常用参数说明

| 参数 | 缩写 | 说明 | 示例 |
|------|------|------|------|
| --table | -t | 数据库表名 | fa_oms_order |
| --controller | -c | 控制器名称 | order |
| --model | -m | 模型名称 | order |
| --fields | -f | 字段列表，默认为表的所有字段 | id,name,status |
| --force | 无 | 强制覆盖已存在的文件，默认为false | true |
| --relation | -r | 关联表名 | fa_oms_category |
| --relationmode | 无 | 关联模式，默认为belongsto | belongsto |
| --relationforeignkey | 无 | 关联外键，默认为表名_id | category_id |
| --relationprimarykey | 无 | 关联主键，默认为id | id |
| --relationfields | 无 | 关联显示字段，默认为name | name,code |

## CRUD生成示例

### 基础示例

生成订单表的CRUD操作：

```bash
php think crud -t fa_oms_order
```

生成的文件包括：

- 控制器：`application/admin/controller/Order.php`
- 模型：`application/admin/model/Order.php`
- 验证器：`application/admin/validate/Order.php`
- 视图文件：
  - `application/admin/view/order/index.html`
  - `application/admin/view/order/add.html`
  - `application/admin/view/order/edit.html`
- JS文件：`public/assets/js/backend/order.js`

### 指定控制器模块

生成订单表的CRUD操作，并指定控制器模块：

```bash
php think crud -t fa_oms_order -c oms/order
```

生成的控制器文件路径为：`application/admin/controller/oms/Order.php`

### 使用关联表

生成订单表的CRUD操作，并关联商品分类表：

```bash
php think crud -t fa_oms_order -r fa_oms_category --relationforeignkey=category_id --relationfields=name,code
```

这将在订单管理界面中显示分类的名称和编码字段。

## CRUD目录结构

生成的CRUD文件主要包括以下几个部分：

### 1. 控制器（Controller）

控制器位于`application/admin/controller/`目录下，负责处理用户请求，调用模型获取数据，并将数据传递给视图。包含以下主要方法：

- `index()`：列表页
- `add()`：添加页
- `edit($ids = null)`：编辑页
- `del($ids = "")`：删除操作
- `multi($ids = "")`：批量操作

### 2. 模型（Model）

模型位于`application/admin/model/`目录下，继承自`app\common\model\BaseModel`，负责数据库操作。

### 3. 验证器（Validate）

验证器位于`application/admin/validate/`目录下，负责表单数据验证，定义字段验证规则和错误提示信息。

### 4. 视图（View）

视图文件位于`application/admin/view/控制器名/`目录下，使用原生PHP语法编写，包括以下文件：

- `index.html`：列表页
- `add.html`：添加页
- `edit.html`：编辑页

### 5. JavaScript

JS文件位于`public/assets/js/backend/控制器名.js`，负责前端数据交互和表格初始化。

## CRUD功能定制

### 1. 字段显示与格式化

在JS文件中，可以通过`columns`配置表格列的显示方式：

```javascript
columns: [
    {field: 'id', title: 'ID'},
    {field: 'name', title: __('Name')},
    {field: 'status', title: __('Status'), formatter: Table.api.formatter.status},
    {field: 'operate', title: __('Operate'), table: table, events: Table.api.events.operate, formatter: Table.api.formatter.operate}
]
```

FastAdmin提供了多种内置的格式化器（formatter），如：

- `Table.api.formatter.status`：状态格式化
- `Table.api.formatter.toggle`：开关格式化
- `Table.api.formatter.image`：图片格式化
- `Table.api.formatter.url`：链接格式化
- `Table.api.formatter.date`：日期格式化
- `Table.api.formatter.datetime`：日期时间格式化

### 2. 自定义搜索

在控制器中，使用`$this->request->get('field')`获取搜索参数，然后在模型查询中添加条件：

```php
public function index()
{
    if ($this->request->isAjax()) {
        $searchName = $this->request->get('name');
        $query = $this->model->where('name', 'like', "%{$searchName}%");
        // 其他代码
    }
    // 其他代码
}
```

### 3. 关联查询

在模型中定义关联关系：

```php
public function category()
{
    return $this->belongsTo('Category', 'category_id', 'id');
}
```

在控制器中使用关联查询：

```php
$list = $this->model->with('category')->where($where)->order($sort, $order)->paginate($limit);
```

### 4. 自定义表单字段

在视图文件中，可以自定义表单字段的显示方式：

```html
<div class="form-group">
    <label class="control-label col-xs-12 col-sm-2">{:__('Status')}:</label>
    <div class="col-xs-12 col-sm-8">
        {:build_radios('row[status]', ['normal'=>__('Normal'), 'hidden'=>__('Hidden')], $row['status'] ?? 'normal')}
    </div>
</div>
```

FastAdmin提供了多种表单构建函数：

- `build_select`：下拉选择框
- `build_radios`：单选按钮组
- `build_checkboxs`：复选框组
- `build_toolbar`：工具栏
- `build_editor`：富文本编辑器

## 常见问题与解决方案

### 1. 生成的CRUD不包含某些字段

问题：生成的CRUD表格或表单中缺少某些字段。

解决方案：使用`--fields`参数指定需要的字段，或在JS文件中手动添加列定义。

### 2. 表单验证规则不符合需求

问题：默认生成的验证规则不满足业务需求。

解决方案：修改验证器文件中的规则定义，添加自定义验证规则：

```php
protected $rule = [
    'name' => 'require|max:50',
    'mobile' => 'require|mobile',
    'email' => 'email',
];
```

### 3. 表格数据格式化显示不正确

问题：表格中的数据显示格式不符合预期。

解决方案：在JS文件中使用自定义格式化函数：

```javascript
{
    field: 'amount', 
    title: __('Amount'), 
    formatter: function(value, row, index) {
        return '￥' + value.toFixed(2);
    }
}
```

### 4. 如何添加自定义按钮

问题：需要添加自定义操作按钮。

解决方案：在JS文件中的events中添加自定义事件处理：

```javascript
events: {
    operate: {
        'click .btn-custom': function (e, value, row, index) {
            e.stopPropagation();
            Layer.alert('自定义操作:' + row.id);
        }
    }
},
formatter: function(value, row, index) {
    var normal = Table.api.formatter.operate.call(this, value, row, index);
    var custom = '<a class="btn btn-xs btn-info btn-custom">自定义</a>';
    return normal + custom;
}
```

## CRUD开发最佳实践

### 1. 命名规范

- 控制器名称：使用大驼峰命名法（如`OrderLog`）
- 模型名称：与控制器同名
- 数据库表名：使用前缀+下划线命名法（如`fa_order_log`）

### 2. 合理使用关联查询

- 对于一对多关系，使用`hasMany`方法
- 对于多对一关系，使用`belongsTo`方法
- 对于多对多关系，使用`belongsToMany`方法

### 3. 表单字段验证

- 必填字段使用`require`规则
- 数字字段使用`number`或`integer`规则
- 文本长度使用`max`和`min`规则
- 特殊格式使用`email`、`url`、`ip`等规则

### 4. 权限控制

- 为每个控制器方法添加注释，以便生成菜单和权限规则
- 使用`$this->auth->check()`方法检查权限
- 合理设置数据权限范围

### 5. 数据过滤与安全

- 使用`$this->request->filter()`方法过滤请求数据
- 对用户输入进行严格验证
- 使用事务保证数据完整性

## 实际案例

### 订单管理CRUD

以订单管理为例，演示完整的CRUD操作流程：

1. 创建数据库表：

```sql
CREATE TABLE `fa_oms_order` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `order_no` varchar(50) NOT NULL COMMENT '订单编号',
  `customer_id` int(10) unsigned NOT NULL COMMENT '客户ID',
  `amount` decimal(10,2) NOT NULL COMMENT '订单金额',
  `status` enum('pending','processing','completed','cancelled') NOT NULL DEFAULT 'pending' COMMENT '订单状态',
  `remark` text COMMENT '备注',
  `createtime` int(10) unsigned DEFAULT NULL COMMENT '创建时间',
  `updatetime` int(10) unsigned DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `order_no` (`order_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='订单表';
```

2. 生成CRUD代码：

```bash
php think crud -t fa_oms_order -c oms/order -r fa_user --relationforeignkey=customer_id --relationfields=username,nickname
```

3. 优化控制器代码：

```php
/**
 * 订单管理
 *
 * @icon fa fa-list
 */
class Order extends Backend
{
    protected $model = null;
    
    // 定义关联表
    protected $relationSearch = true;
    
    // 定义可搜索字段
    protected $searchFields = 'id,order_no,customer.username,customer.nickname';
    
    public function _initialize()
    {
        parent::_initialize();
        $this->model = model('app\admin\model\oms\Order');
        
        // 设置状态选项
        $this->view->assign('statusList', [
            'pending' => __('Status pending'),
            'processing' => __('Status processing'),
            'completed' => __('Status completed'),
            'cancelled' => __('Status cancelled')
        ]);
    }
    
    /**
     * 默认生成的控制器所继承的父类中有index/add/edit/del/multi五个基础方法、destroy/restore/recyclebin三个回收站方法
     * 因此在当前控制器中可不用编写增删改查的代码,除非需要自己控制这部分逻辑
     * 需要将application/admin/library/traits/Backend.php中对应的方法复制到当前控制器,然后进行修改
     */
}
```

4. 优化JS代码：

```javascript
define(['jquery', 'bootstrap', 'backend', 'table', 'form'], function ($, undefined, Backend, Table, Form) {

    var Controller = {
        index: function () {
            // 初始化表格参数配置
            Table.api.init({
                extend: {
                    index_url: 'oms/order/index' + location.search,
                    add_url: 'oms/order/add',
                    edit_url: 'oms/order/edit',
                    del_url: 'oms/order/del',
                    multi_url: 'oms/order/multi',
                    table: 'oms_order',
                }
            });

            var table = $("#table");

            // 初始化表格
            table.bootstrapTable({
                url: $.fn.bootstrapTable.defaults.extend.index_url,
                pk: 'id',
                sortName: 'id',
                columns: [
                    [
                        {checkbox: true},
                        {field: 'id', title: __('Id')},
                        {field: 'order_no', title: __('Order_no')},
                        {field: 'customer.username', title: __('Customer')},
                        {field: 'amount', title: __('Amount'), operate: 'BETWEEN'},
                        {field: 'status', title: __('Status'), searchList: {"pending":__('Status pending'),"processing":__('Status processing'),"completed":__('Status completed'),"cancelled":__('Status cancelled')}, formatter: Table.api.formatter.status},
                        {field: 'createtime', title: __('Createtime'), operate:'RANGE', addclass:'datetimerange', formatter: Table.api.formatter.datetime},
                        {field: 'operate', title: __('Operate'), table: table, events: Table.api.events.operate, formatter: Table.api.formatter.operate}
                    ]
                ]
            });

            // 为表格绑定事件
            Table.api.bindevent(table);
        },
        add: function () {
            Controller.api.bindevent();
        },
        edit: function () {
            Controller.api.bindevent();
        },
        api: {
            bindevent: function () {
                Form.api.bindevent($("form[role=form]"));
            }
        }
    };
    return Controller;
});
```

5. 添加语言包：

```php
<?php

return [
    'Id'                => 'ID',
    'Order_no'          => '订单编号',
    'Customer_id'       => '客户ID',
    'Customer'          => '客户',
    'Amount'            => '订单金额',
    'Status'            => '订单状态',
    'Status pending'    => '待处理',
    'Status processing' => '处理中',
    'Status completed'  => '已完成',
    'Status cancelled'  => '已取消',
    'Remark'            => '备注',
    'Createtime'        => '创建时间',
    'Updatetime'        => '更新时间'
];
```

## 总结

FastAdmin的CRUD生成功能大大简化了Web应用开发的过程，通过命令行工具可以快速生成基础代码框架，开发者只需专注于业务逻辑的实现，提高了开发效率。在实际开发中，可以根据业务需求对生成的代码进行定制和优化，实现更复杂的功能。 