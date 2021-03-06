# Laravel-Vue-Admin
## 简介
Laravel-Vue-Admin 是一个开箱即用的Laravel后台扩展，很多地方都参考了 laravel-admin 和 Nova

## 安装
首先确保安装好了laravel，并且数据库连接设置正确。

由于现在是开发阶段，建议安装开发版本体验，暂不建议正式项目使用哦~~
``` bash
composer require smallruraldog/laravel-vue-admin:dev-master
```

然后运行下面的命令来发布资源：
``` bash
php artisan vendor:publish --provider="SmallRuralDog\Admin\AdminServiceProvider"
```
在该命令会生成配置文件`config/admin.php`，可以在里面修改安装的地址、数据库连接、以及表名，建议都是用默认配置不修改。
然后运行下面的命令完成安装：
``` bash
php artisan admin:install
```
启动服务后，在浏览器打开 `/admin` ,使用用户名 admin 和密码 admin登录.
## 开始使用
下面是一个简单使用的代码示例
```php
namespace SmallRuralDog\Admin\Controllers;
use SmallRuralDog\Admin\Components\Tag;
use SmallRuralDog\Admin\Components\Transfer;
use SmallRuralDog\Admin\Components\TransferData;
use SmallRuralDog\Admin\Form;
use SmallRuralDog\Admin\Grid;
class RoleController extends AdminController
{
    protected function grid()
    {
        $roleModel = config('admin.database.roles_model');
        $grid = new Grid(new $roleModel());
        $grid->columns([
            $grid->column('id', 'ID')->width('80px')->sortable(),
            $grid->column('slug', trans('admin::admin.slug')),
            $grid->column('name', trans('admin::admin.name')),
            $grid->column('permissions.name', trans('admin::admin.permission'))->displayComponent(Tag::make()->type('info')),
            $grid->column('created_at', trans('admin::admin.created_at')),
            $grid->column('updated_at', trans('admin::admin.updated_at'))
        ]);
        return $grid;
    }
    public function form()
    {
        $permissionModel = config('admin.database.permissions_model');
        $roleModel = config('admin.database.roles_model');
        $form = new Form(new $roleModel());
        $form->items([
            $form->item('slug', trans('admin::admin.slug'))->serveRules('required'),
            $form->item('name', trans('admin::admin.name'))->required()->serveRules('required'),
            $form->item('permissions', trans('admin::admin.permissions'), 'permissions.id')->displayComponent(
                Transfer::make()->data($permissionModel::get()->map(function ($item) {
                    return TransferData::make($item->id, $item->name);
                }))->titles(['可授权', '已授权'])->filterable()
            )
        ]);
        return $form;
    }
}
```
创建一个控制器继承 `AdminController`。

注册路由
```php
 $router->resource('auth/roles', 'RoleController')->names('admin.auth.roles');
```
添加菜单，菜单的Uri和注册的路由`auth/roles`一样

## 版本升级

### 查看当前版本
```bash
composer show smallruraldog/laravel-vue-admin
```
### 更新到最新版
```bash
composer require smallruraldog/laravel-vue-admin
```
### 更新到开发版
```bash
composer require smallruraldog/laravel-vue-admin:dev-master
```
### 注意事项
由于每个版本的静态资源或者语言包都有可能会有更新，所以升级版本之后最好运行下面的命令
```bash
// 强制发布静态资源文件
php artisan vendor:publish --tag=laravel-vue-admin-assets --force

// 强制发布语言包文件
php artisan vendor:publish --tag=laravel-vue-admin-lang --force

// 清理视图缓存
php artisan view:clear
```
最后不要忘记清理浏览器缓存