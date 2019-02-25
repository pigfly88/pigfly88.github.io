---
layout: post
category: "php"
title:  "无限极分类和递归"
---

```php
$data = array(
    array('id'=>1, 'pid'=>0, 'name'=>'IT'),
    array('id'=>2, 'pid'=>0, 'name'=>'金融'),
    array('id'=>3, 'pid'=>0, 'name'=>'文学'),
    array('id'=>4, 'pid'=>0, 'name'=>'摄影'),
    array('id'=>5, 'pid'=>1, 'name'=>'PHP'),
    array('id'=>6, 'pid'=>1, 'name'=>'Java'),
    array('id'=>7, 'pid'=>5, 'name'=>'Laravel'),
    array('id'=>8, 'pid'=>2, 'name'=>'股票'),
    array('id'=>9, 'pid'=>7, 'name'=>'Laravel数据库迁移'),
    array('id'=>10, 'pid'=>3, 'name'=>'红楼梦'),
    array('id'=>11, 'pid'=>6, 'name'=>'spring'),
    array('id'=>12, 'pid'=>7, 'name'=>'Laravel artisan'),
);

function get_tree($data, $pid=0) {
    $ret = array();
    foreach($data as $v) {
        if($v['pid']===$pid) {
            $v['children'] = get_tree($data, $v['id']);
            $ret[] = $v;
        }
    }
    return $ret;
}

function show_tree($data, $pid=0, $level=0) {

    foreach($data as $v) {
        if($v['pid']===$pid) {
            $level_str = str_repeat('-', $level);
            echo "<p>{$level_str}{$v['name']}</p>";
            show_tree($data, $v['id'], $level+1);
        }
    }

}

$tree = get_tree($data);
echo "<pre>";
print_r($tree);
echo "</pre>";

show_tree($data);
```