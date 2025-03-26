# FastAdmin插件开发文档

FastAdmin提供了强大的插件机制，可以方便地扩展系统功能，实现功能模块化和可插拔式开发。本文将详细介绍FastAdmin插件的开发与使用方法。

## 插件系统概述

FastAdmin的插件系统具有以下特点：

1. **独立性**：每个插件都是相对独立的功能模块
2. **可插拔**：支持在线安装、卸载、启用和禁用
3. **可升级**：支持在线升级插件
4. **权限控制**：插件可独立控制权限

插件在FastAdmin中称为Addon，位于项目根目录的`addons`目录下，每个插件都是一个独立的目录。

## 插件目录结构

一个标准的FastAdmin插件目录结构如下：

```
myaddon/                     # 插件目录
├── application/             # 应用目录
│   ├── admin/               # 后台模块
│   │   ├── controller/      # 控制器目录
│   │   ├── model/           # 模型目录
│   │   ├── view/            # 视图目录
│   │   └── lang/            # 语言包目录
│   ├── api/                 # API模块
│   └── index/               # 前台模块
├── public/                  # 静态资源目录
│   ├── assets/              # 资源目录
│   │   ├── css/             # CSS目录
│   │   ├── js/              # JS目录
│   │   └── images/          # 图片目录
│   └── uploads/             # 上传目录
├── bootstrap.js             # 前端启动文件
├── config.php               # 插件配置文件
├── info.ini                 # 插件信息文件
├── install.sql              # 安装SQL文件
├── LICENSE                  # 版权文件
├── Plugin.php               # 插件核心类
├── README.md                # 说明文档
└── uninstall.sql            # 卸载SQL文件
```

## 插件开发流程

### 1. 创建插件

FastAdmin提供了命令行工具，可以快速创建插件的基本结构：

```bash
php think addon -a myaddon -c demo
```

这个命令会创建一个名为`myaddon`的插件，并生成一个名为`demo`的控制器。

### 2. 配置插件信息

编辑`info.ini`文件，设置插件的基本信息：

```ini
name = 我的插件
title = 我的插件
intro = 这是一个示例插件
author = 作者名称
website = https://www.example.com
version = 1.0.0
state = 1
```

参数说明：
- name：插件名称
- title：插件标题
- intro：插件介绍
- author：作者名称
- website：作者网站
- version：版本号
- state：状态，1=启用，0=禁用

### 3. 编辑配置文件

编辑`config.php`文件，设置插件的配置参数：

```php
<?php

return [
    [
        'name' => 'appid',
        'title' => 'AppID',
        'type' => 'string',
        'content' => '',
        'value' => '',
        'rule' => 'required',
        'msg' => '',
        'tip' => '',
        'ok' => '',
        'extend' => ''
    ],
    [
        'name' => 'appsecret',
        'title' => 'AppSecret',
        'type' => 'string',
        'content' => '',
        'value' => '',
        'rule' => 'required',
        'msg' => '',
        'tip' => '',
        'ok' => '',
        'extend' => ''
    ]
];
```

配置参数说明：
- name：配置名称
- title：配置标题
- type：配置类型，string=字符，text=文本，array=数组，checkbox=复选框，radio=单选框，select=下拉，selects=多选下拉，image=图片，images=多图片，file=文件，files=多文件，number=数字，datetime=日期时间，date=日期，time=时间，select_page=关联选择，editor=富文本编辑器
- content：配置内容，用于radio、checkbox、select、selects等类型
- value：默认值
- rule：验证规则
- msg：验证失败提示
- tip：帮助提示
- ok：验证成功提示
- extend：扩展参数

### 4. 开发插件核心类

编辑`Plugin.php`文件，实现插件的核心功能：

```php
<?php

namespace addons\myaddon;

use app\common\library\Menu;
use think\Addons;

/**
 * 插件核心类
 */
class Plugin extends Addons
{
    /**
     * 插件安装方法
     * @return bool
     */
    public function install()
    {
        // 安装菜单
        $menu = [
            [
                'name' => 'myaddon',
                'title' => '我的插件',
                'icon' => 'fa fa-plug',
                'sublist' => [
                    [
                        'name' => 'myaddon/demo',
                        'title' => '演示管理',
                        'icon' => 'fa fa-list',
                        'sublist' => [
                            ['name' => 'myaddon/demo/index', 'title' => '查看'],
                            ['name' => 'myaddon/demo/add', 'title' => '添加'],
                            ['name' => 'myaddon/demo/edit', 'title' => '编辑'],
                            ['name' => 'myaddon/demo/del', 'title' => '删除'],
                            ['name' => 'myaddon/demo/multi', 'title' => '批量更新'],
                        ]
                    ]
                ]
            ]
        ];
        Menu::create($menu);
        return true;
    }

    /**
     * 插件卸载方法
     * @return bool
     */
    public function uninstall()
    {
        // 删除菜单
        Menu::delete('myaddon');
        return true;
    }

    /**
     * 插件启用方法
     * @return bool
     */
    public function enable()
    {
        // 启用菜单
        Menu::enable('myaddon');
        return true;
    }

    /**
     * 插件禁用方法
     * @return bool
     */
    public function disable()
    {
        // 禁用菜单
        Menu::disable('myaddon');
        return true;
    }
}
```

### 5. 开发安装和卸载SQL

如果插件需要创建数据表，可以在`install.sql`和`uninstall.sql`文件中编写SQL语句：

#### install.sql

```sql
CREATE TABLE IF NOT EXISTS `__PREFIX__addon_myaddon` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(50) NOT NULL COMMENT '名称',
  `content` text COMMENT '内容',
  `createtime` int(10) unsigned DEFAULT NULL COMMENT '创建时间',
  `updatetime` int(10) unsigned DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='插件演示表';
```

#### uninstall.sql

```sql
DROP TABLE IF EXISTS `__PREFIX__addon_myaddon`;
```

注意：SQL文件中的`__PREFIX__`会被自动替换为系统的表前缀。

### 6. 开发控制器

在`application/admin/controller`目录下创建控制器文件：

```php
<?php

namespace addons\myaddon\application\admin\controller;

use think\addons\Controller;

class Demo extends Controller
{
    // 模型对象
    protected $model = null;
    
    public function _initialize()
    {
        parent::_initialize();
        $this->model = new \addons\myaddon\application\admin\model\Demo;
    }
    
    /**
     * 查看
     */
    public function index()
    {
        if ($this->request->isAjax()) {
            $list = $this->model->select();
            $result = array("total" => count($list), "rows" => $list);
            return json($result);
        }
        return $this->view->fetch();
    }
    
    /**
     * 添加
     */
    public function add()
    {
        if ($this->request->isPost()) {
            $params = $this->request->post("row/a");
            if ($this->model->save($params)) {
                $this->success();
            } else {
                $this->error();
            }
        }
        return $this->view->fetch();
    }
    
    /**
     * 编辑
     */
    public function edit($ids = null)
    {
        $row = $this->model->get($ids);
        if (!$row) {
            $this->error(__('No Results were found'));
        }
        if ($this->request->isPost()) {
            $params = $this->request->post("row/a");
            if ($row->save($params)) {
                $this->success();
            } else {
                $this->error();
            }
        }
        $this->view->assign("row", $row);
        return $this->view->fetch();
    }
    
    /**
     * 删除
     */
    public function del($ids = "")
    {
        if ($ids) {
            $pk = $this->model->getPk();
            $list = $this->model->where($pk, 'in', $ids)->select();
            $count = 0;
            foreach ($list as $k => $v) {
                $count += $v->delete();
            }
            if ($count) {
                $this->success();
            } else {
                $this->error(__('No rows were deleted'));
            }
        }
        $this->error(__('Parameter %s can not be empty', 'ids'));
    }
}
```

### 7. 开发模型

在`application/admin/model`目录下创建模型文件：

```php
<?php

namespace addons\myaddon\application\admin\model;

use think\Model;

class Demo extends Model
{
    // 表名
    protected $name = 'addon_myaddon';
    
    // 自动写入时间戳字段
    protected $autoWriteTimestamp = 'int';

    // 定义时间戳字段名
    protected $createTime = 'createtime';
    protected $updateTime = 'updatetime';
}
```

### 8. 开发视图

在`application/admin/view/demo`目录下创建视图文件：

#### index.html

```html
<div class="panel panel-default panel-intro">
    <div class="panel-heading">
        <div class="panel-lead"><em>演示管理</em>用于演示插件功能</div>
        <ul class="nav nav-tabs">
            <li class="active"><a href="#all" data-toggle="tab">全部</a></li>
        </ul>
    </div>
    <div class="panel-body">
        <div id="myTabContent" class="tab-content">
            <div class="tab-pane fade active in" id="all">
                <div id="toolbar" class="toolbar">
                    {:build_toolbar('refresh,add,edit,del')}
                </div>
                <table id="table" class="table table-striped table-bordered table-hover" 
                       data-operate-edit="{:$auth->check('myaddon/demo/edit')}" 
                       data-operate-del="{:$auth->check('myaddon/demo/del')}" 
                       width="100%">
                </table>
            </div>
        </div>
    </div>
</div>
```

#### add.html

```html
<form id="add-form" class="form-horizontal" role="form" data-toggle="validator" method="POST" action="">

    <div class="form-group">
        <label class="control-label col-xs-12 col-sm-2">{:__('Name')}:</label>
        <div class="col-xs-12 col-sm-8">
            <input id="c-name" data-rule="required" class="form-control" name="row[name]" type="text">
        </div>
    </div>
    <div class="form-group">
        <label class="control-label col-xs-12 col-sm-2">{:__('Content')}:</label>
        <div class="col-xs-12 col-sm-8">
            <textarea id="c-content" class="form-control editor" rows="5" name="row[content]" cols="50"></textarea>
        </div>
    </div>
    <div class="form-group layer-footer">
        <label class="control-label col-xs-12 col-sm-2"></label>
        <div class="col-xs-12 col-sm-8">
            <button type="submit" class="btn btn-success btn-embossed disabled">{:__('OK')}</button>
            <button type="reset" class="btn btn-default btn-embossed">{:__('Reset')}</button>
        </div>
    </div>
</form>
```

#### edit.html

```html
<form id="edit-form" class="form-horizontal" role="form" data-toggle="validator" method="POST" action="">

    <div class="form-group">
        <label class="control-label col-xs-12 col-sm-2">{:__('Name')}:</label>
        <div class="col-xs-12 col-sm-8">
            <input id="c-name" data-rule="required" class="form-control" name="row[name]" type="text" value="{$row.name|htmlentities}">
        </div>
    </div>
    <div class="form-group">
        <label class="control-label col-xs-12 col-sm-2">{:__('Content')}:</label>
        <div class="col-xs-12 col-sm-8">
            <textarea id="c-content" class="form-control editor" rows="5" name="row[content]" cols="50">{$row.content|htmlentities}</textarea>
        </div>
    </div>
    <div class="form-group layer-footer">
        <label class="control-label col-xs-12 col-sm-2"></label>
        <div class="col-xs-12 col-sm-8">
            <button type="submit" class="btn btn-success btn-embossed disabled">{:__('OK')}</button>
            <button type="reset" class="btn btn-default btn-embossed">{:__('Reset')}</button>
        </div>
    </div>
</form>
```

### 9. 开发前端JS

在`public/assets/js/backend`目录下创建JS文件：

```javascript
define(['jquery', 'bootstrap', 'backend', 'table', 'form'], function ($, undefined, Backend, Table, Form) {

    var Controller = {
        index: function () {
            // 初始化表格参数配置
            Table.api.init({
                extend: {
                    index_url: 'myaddon/demo/index',
                    add_url: 'myaddon/demo/add',
                    edit_url: 'myaddon/demo/edit',
                    del_url: 'myaddon/demo/del',
                    multi_url: 'myaddon/demo/multi',
                    table: 'addon_myaddon',
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
                        {field: 'name', title: __('Name')},
                        {field: 'createtime', title: __('Createtime'), formatter: Table.api.formatter.datetime},
                        {field: 'updatetime', title: __('Updatetime'), formatter: Table.api.formatter.datetime},
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

### 10. 添加语言包

在`application/admin/lang/zh-cn`目录下创建语言包文件：

```php
<?php

return [
    'Id' => 'ID',
    'Name' => '名称',
    'Content' => '内容',
    'Createtime' => '创建时间',
    'Updatetime' => '更新时间'
];
```

## 插件的安装与使用

### 1. 打包插件

将插件目录打包成zip格式，命名为插件名称.zip，如`myaddon.zip`。

### 2. 安装插件

FastAdmin提供了两种安装插件的方式：

#### 通过后台安装

1. 登录后台，进入"插件管理"
2. 点击"上传"按钮，选择插件zip包上传
3. 系统会自动解压并安装插件

#### 通过命令行安装

```bash
php think addon -a myaddon -c install
```

### 3. 使用插件

安装成功后，可以在后台"插件管理"中看到已安装的插件，可以进行启用、禁用、卸载等操作。

### 4. 插件的升级

当插件有新版本时，可以通过以下步骤升级：

1. 准备新版本的插件包
2. 在后台"插件管理"中点击"升级"按钮
3. 上传新版本的插件包
4. 系统会自动解压并升级插件

## 插件钩子系统

FastAdmin提供了钩子系统，可以实现插件间的交互和系统功能的扩展。

### 1. 定义钩子

在`Plugin.php`文件中定义钩子：

```php
/**
 * 插件钩子
 */
public function hooks()
{
    return [
        'testhook' => 'hookTest'
    ];
}

/**
 * 钩子测试
 * @param array $params 参数
 * @return string
 */
public function hookTest($params = [])
{
    return '这是一个钩子示例';
}
```

### 2. 调用钩子

在系统中调用钩子：

```php
// 调用钩子
$result = \app\common\library\Addons::hook('testhook', ['param1' => 'value1']);
```

### 3. 常用钩子

FastAdmin系统预定义了一些常用钩子：

- `app_init`：应用初始化
- `app_begin`：应用开始
- `module_init`：模块初始化
- `action_begin`：控制器开始
- `view_filter`：视图输出过滤
- `app_end`：应用结束
- `admin_login_after`：管理员登录后
- `admin_logout_after`：管理员退出后

## 插件前台访问

FastAdmin的插件支持前台访问，可以通过以下URL访问插件的前台页面：

```
https://example.com/addons/myaddon/index/index
```

这个URL会访问`addons\myaddon\application\index\controller\Index`控制器的`index`方法。

## 插件API访问

FastAdmin的插件也支持API访问，可以通过以下URL访问插件的API接口：

```
https://example.com/api/addons/myaddon/index/index
```

这个URL会访问`addons\myaddon\application\api\controller\Index`控制器的`index`方法。

## 插件静态资源

插件的静态资源位于`public`目录下，访问方式为：

```
https://example.com/assets/addons/myaddon/css/style.css
```

## 插件开发最佳实践

### 1. 命名规范

- 插件名称：使用小写字母，多个单词用下划线连接
- 控制器：使用大驼峰命名法
- 模型：与控制器同名
- 视图：使用小写字母，与控制器方法同名

### 2. 配置设计

- 合理设计插件配置项，避免过多或过少
- 为配置项提供默认值和帮助提示
- 对必填项添加验证规则

### 3. 数据表设计

- 表名使用前缀`addon_插件名`，如`addon_myaddon`
- 包含必要的字段，如ID、创建时间、更新时间等
- 使用合适的字段类型和长度

### 4. 权限控制

- 合理设计插件的权限结构
- 在关键操作处添加权限验证
- 使用数据权限控制数据访问范围

### 5. 错误处理

- 对可能出现的错误进行捕获和处理
- 提供友好的错误提示信息
- 记录关键操作日志

### 6. 性能优化

- 避免重复查询数据库
- 合理使用缓存
- 优化SQL查询语句
- 精简前端资源文件

## 常见问题与解决方案

### 1. 插件安装失败

可能原因：
- 插件目录结构不正确
- 安装SQL语句错误
- 插件依赖的扩展或库不存在

解决方案：
- 检查插件目录结构是否符合规范
- 检查install.sql文件是否存在语法错误
- 检查插件依赖是否已安装

### 2. 插件前台访问404

可能原因：
- 路由配置错误
- 控制器文件不存在
- 方法不存在或非public

解决方案：
- 检查路由配置
- 确认控制器文件存在且命名正确
- 确认方法存在且为public

### 3. 插件样式或JS不生效

可能原因：
- 资源文件路径错误
- 资源文件没有正确引入
- 存在JS错误导致后续JS不执行

解决方案：
- 检查资源文件路径是否正确
- 确认资源文件已正确引入
- 检查控制台是否有JS错误

## 总结

FastAdmin的插件系统提供了强大的扩展能力，可以方便地实现功能模块化和可插拔式开发。通过合理设计和开发插件，可以大大提高系统的可维护性和可扩展性。希望本文能够帮助开发者更好地理解和使用FastAdmin的插件系统。 