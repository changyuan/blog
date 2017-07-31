---
title: 浅析Yii2中GridView
date: 2017-01-25 12:08:15
updated: 2017-01-25 12:08:15
tags:
categories:
---

是否显示某列案例
我们举一个简单的案例
条件：有一个get形参数type
需求：仅且type的值等于1的时候，列name才显示，否则该列不显示
<!--more-->

```php
[
'attribute' => 'name',
'value' => $model->name,
'visible' => intval(Yii::$app->request->get('type')) == 1,
],
```

链接可点击跳转案例

```php
[
'attribute' => 'order_id',
'value' => function ($model) {
return Html::a($model->order_id, "/order?id={$model->order_id}", ['target' => '_blank']);
},
'format' => 'raw',
],
```

这里只需要指定format格式为image即可，format第二个参数可设定图片大小
```php
[
'label' => '头像',
'format' => [
'image', 
[
'width'=>'84',
'height'=>'84'
]
],
'value' => function ($model) { 
return $model->image; 
}
],


```

html渲染案例

```php
[
'attribute' => 'title',
	'value' => function ($model) { 
	return Html::encode($model->title); 
	},
	'format' => 'raw',
],

```

自定义按钮案例

```php
[
'class' => 'yii\grid\ActionColumn',
'template' => '{get-xxx} {view} {update}',
'header' => '操作',
'buttons' => [
'get-xxx' => function ($url, $model, $key) { 
return Html::a('获取xxx', $url, ['title' => '获取xxx'] ); 
},
],
],

```

设定宽度案例

```php
[
'attribute' => 'title',
'value' => 'title',
'headerOptions' => ['width' => '100'],
'contentOptions'=>['width'=>20]
],


'rowOptions' => function($model, $key, $index, $grid) {
return ['class' => $index % 2 ==0 ? 'label-red' : 'label-green'];
},


```

增加按钮调用js操作案例

```php
[
'class' => 'yii\grid\ActionColumn',
'header' => '操作',
'template' => '{view} {update} {update-status}',
'buttons' => [
'update-status' => function ($url, $model, $key) {
return Html::a('更新状态', 'javascript:;', ['onclick'=>'update_status(this, '.$model->id.');']); },
],
],

```



1. 处理时间
数据列的主要配置项是 yii\grid\DataColumn::format 属性。
它的值默认是使用 \yii\i18n\Formatter 应用组件。
```
[
'label'=>'更新日期',
'format' => ['date', 'php:Y-m-d'],
'value' => 'updated_at'
],
//or
[
//'attribute' => 'created_at',
'label'=>'更新时间',
'value'=>function($model){
return date('Y-m-d H:i:s',$model->created_at); 
},
'headerOptions' => ['width' => '170'],
],
```
2. 处理图片
```
[
'label'=>'封面图',
'format'=>'raw',
'value'=>function($m){
return Html::img($m->cover,
['class' => 'img-circle',
'width' => 30]
);
}
],
```

3. 数据列有链接
```
[
'attribute' => 'title',
'value' => function ($model, $key, $index, $column) {
return Html::a($model->title, 
['article/view', 'id' => $key]);
},
'format' => 'raw',
],
```
4. 数据列显示枚举值(男/女）
```
[
'attribute' => 'sex', 
'value'=>function ($model,$key,$index,$column){
return $model->sex==1?'男':'女'; 
}，
//在搜索条件（过滤条件）中使用下拉框来搜索
'filter' => ['1'=>'男','0'=>'女'],
//or
'filter' => Html::activeDropDownList($searchModel,
'sex',['1'=>'男','0'=>'女'],
['prompt'=>'全部']
)
],
[
'label'=>'产品状态', 
'attribute' => 'pro_name', 
'value' => function ($model) {
$state = [
'0' => '未发货',
'1' => '已发货',
'9' => '退货，已处理',
];
return $state[$model->pro_name];
},
'headerOptions' => ['width' => '120'] 
]

```

