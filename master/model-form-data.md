# 表单数据源

## 模型与数据仓库

`Dcat Admin`中表单对数据的操作并非直接依赖于`Eloquent Model`，而是依赖于`Repository`提供的数据操作接口。

> {tip} `Repository`提供更简单更清晰数据操作接口，开发者可以通过实现不同的数据操作接口轻松支持多数据源，具体使用方法请参照[数据仓库](model-repository.md)。


## 数据来自模型
当数据源支持模型时，只需创建一个非常简单的`Repository`类既可：
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // 这里定义你的模型类名
    protected $eloquentClass = MovieModel::class;
    
    // 通过这个方法可以指定表单页查询的字段，默认"*"
    public function getFormColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
}
```
使用：
```php
use App\Admin\Repositories\Movie;

$form = new Form(new Movie);

...
```

## 数据来自外部API

当下面用`豆瓣电影`的API为例子，来展示`Repository`的写操作相关接口：

```php
<?php
namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\Repository;
use Dcat\Admin\Form;

class ComingSoon extends Repository
{
    protected $api = 'https://api.douban.com/v2/movie/coming_soon';
    
    // 返回你的id字段名称，默认“id”
    protected $keyName = '_id';

    // 查询编辑页数据
    // 这个方法需要返回一个数组
    public function edit(Form $form): array
    {
        // 获取id
        $id = $form->builder()->getResourceId();

        $data = file_get_contents("http://api.douban.com/v2/movie/subject/$id");

        return json_decode($data, true);
    }
    
    // 这个方法用于在修改数据前查询原记录
    // 如果使用了文件上传表单，当文件发生变更时会根据这个原始记录自动删除旧文件
     // 如果不需要此数据返回空数组即可
    public function getDataWhenUpdating(Form $form): array
    {
        // 获取id
        $id = $form->builder()->getResourceId();
        
        return [];
    }

    // 修改操作
    // 返回一个bool类型的数据
    public function update(Form $form)
    {
        // 获取id
        $id = $form->builder()->getResourceId();

        // 获取要修改的数据
        $attributes = $form->getUpdates();

        // TODO
        // 这里写你的修改逻辑
        
        return true;
    }

    // 这个方法用于在修改数据前查询原始数据
    // 如果使用了文件上传表单，会根据这个数据自动删除文件
    // 如果不需要此数据返回空数组即可
    public function getDataWhenDeleting(Form $form): array
    {
        $id = $form->builder()->getResourceId();
        
        $id = explode(',', $id);

//        $data = file_get_contents("http://api.douban.com/v2/movie/subject/$id");
//
//        return json_decode($data, true);

        return [];
    }

    // 删除数据
    // $deletingData 是由 getDataWhenDeleting 方法返回的数据
    public function destroy(Form $form, array $deletingData)
    {
        // 注意这里的id可能是多个
        $id = $form->builder()->getResourceId();
        
        // 当使用批量删除功能时，这里的id是用“,”隔开的字符串
        $id = explode(',', $id);

        // TODO
//        var_dump($id, $deletingData);

        return true;
    }

}
```


