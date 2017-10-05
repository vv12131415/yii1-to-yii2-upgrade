# WIP
Disclaimer - it's not my responsibility if you broke your app using those rules, they helped me, but I don't know if they will help you , I'm just describing my own experience.

## Things to do before
* use git
* have a copy of everything that you will update
* make sure the code is stable (everything works fine and etc.). If you will update broken app things will get even worse (own experience :disappointed:)
* To do so - **test everything!!!** , if you don't know how to test, learn to test first and then make update.
* make incremental updates. So first update your code to latest `yii1` version and then to the first `yii2` version and then to the latest `yii2` version.
* learn `yii1` and `yii2` (not one of them but both, **important**, I didn't so I was having not that much idea what is going on there) before update.
* namespace everything before update
* make code clean and neat - so nothing will confuse you.
* old one has project in `protected` folder - if app is not too complex and it fits basic/advanced template, try to match it.
* use CS fixer and php inspections (EA extended) to solve common problems before
* if the app is old and all the data has no migrations and stored in some `sql` file. Make migration that will restore your DB.
* Update to the latest core thing possible. I mean use `latest versions of php`, `mysql` and etc.
* you can change most of the logging from `Yii::log()` to `Yii::info()`/`Yii::trace()` or others. `Yii::log()` will be removed.

## General
* most of the classes names are the same, just `C` dropped and namespace added
* don't forget to include classes (`use` keyword)
* change `Yii::app()` to`Yii::$app`
* there was `Yii::app()->user` and now there are `Yii::$app->user` and `Yii::$app->user->identity`
* flash methods moved was something like `Yii::app()->user->hasFlash('someKey')` to `Yii::$app->session->hasFlash('someKey')`
* if you use some external dependency (and you probably do) make sure it has the same (best) or at least similar heir
* in your view, make sure you have included `<?php $this->head() ?>` and similar ([see](https://github.com/yiisoft/yii2-app-basic/blob/master/views/layouts/main.php)), other way your assets will not be included
* widget.
```php
//was
$this->beginWidget('wigdet.class.Name',['config' => 'array'])
//now
$widget = WidgetClassName::begin(['config' => 'array'])

//was
$this->endWidget();
//now
WidgetClassName::end();

//was
$this->widget('wigdet.class.Name', [])
//now
WidgetClassName::end();
```
* view. You now can't set properties in view (before it was controller class with included file, now it's separate class). For example to set/get previously you can do `$this->menu`, now this code produces
`Setting unknown property: yii\web\View::menu`  error. Use `$this->params['menu']` instead.
* [not sure] if you where using bootstrap widgets previously you had class with `Tb` at the beginning now it's dropped, for example
```php
//was
$form = $this->beginWidget('bootstrap.widgets.TbActiveForm', [
//now
$form = \yii\bootstrap\ActiveForm::begin([
```
* Bootstrap button widget. config properties `buttonType` and `type` was dropped
* Logging. `Yii::log()` was dropped. Use [one of those instead](http://www.yiiframework.com/doc-2.0/guide-runtime-logging.html#log-messages)
* no `Html::listData()` method
```php
//was
$form->dropDownListRow($model, 'group_id',
    //don't make such things in view prepare data in controller/model first
    CHtml::listData(Group::model()->findAllU(), 'id', 'name')
    );
//now
$form->field($model, 'group_id')
    ->dropDownList(
    //don't make such things in view prepare data in controller/model first
    ArrayHelper::map(Group::findAll(), 'id', 'name')
    );
```
* `ActiveField` changes
```php
//was
$form->textFieldRow($model, 'name', ['maxlength' => 255]);
//now
$form->field($model, 'from', ['options' => ['maxlength' => '255']])->textInput();

//was
$form->checkBoxRow($model, 'active');
//now
$form->field($model, 'active')->checkbox();
```
* 
```php
//was
$this->widget('bootstrap.widgets.TbButton', [
    'buttonType' => 'submit',
    'type' => 'primary',
    'label' => Yii::t('app', 'Import') ,
]);
//now
echo \yii\bootstrap\Button::widget([
        'options' => ['type' => 'submit',
            'class' => 'btn btn-primary',],
    'label' => Yii::t('app', 'Import') ,
]);
```
* validator
```php
//was
public function rules()
{
    return [
        ['first_name, last_name, age', 'required'],
        ['file', 'file', 'allowEmpty' => false, 'types' => 'sql, bak'],
    ];
}
//now
public function rules()
{
    return [
        [['first_name', 'last_name', 'age'], 'required'],
        ['file', 'file', 'skipOnEmpty' => false, 'extensions' => 'sql, bak'],
    ];
}
```
`length` validator changed to `string`
* db component
```php
//was
[
    'class' => 'CDbConnection',
    'connectionString' => 'mysql:host=localhost;dbname=user_db',
    'username' => 'homestead',
    'password' => 'secret',
    'charset' => 'utf8',
    'emulatePrepare' => true,
    'tablePrefix' => '',
]
//now
[
    'class' => Connection::class,
    'dsn' => 'mysql:host=localhost;dbname=user_db',
    'username' => 'homestead',
    'password' => 'secret',
    'charset' => 'utf8',
    'emulatePrepare' => true,
    'tablePrefix' => '',
]
```
* config changes see [yii1 project example](https://github.com/yiisoft/yii/tree/master/demos/blog/protected/config) and [yii2 basic template](https://github.com/yiisoft/yii2-app-basic/tree/master/config)
* don't forget to include `<?= Html::csrfMetaTags() ?>` into `head` of your HTML template/layout
* file sending. yii1(no link google for it) and [yii2](http://www.yiiframework.com/doc-2.0/guide-runtime-responses.html#sending-files)
```php
//was
return Yii::app()->getRequest()->sendFile('/var/www/myProject/files/readme.md');
//now
return Yii::$app->response->sendFile('/var/www/myProject/files/readme.md');
```
* `db` method rename 
```php
//was
$db = Yii::app()->db;
return $db->getSchema()->getTables();
//now
$db = Yii::$app->db;
return $db->getSchema()->getTableSchemas();

//was
$db = Yii::app()->db;
$pdo = $db->getPdoInstance();;
//now
$db = Yii::$app->db;
$pdo = $db->getMasterPdo();
//or
$db = Yii::$app->db;
$pdo = $db->getSlavePdo();
```
method `getPdoInstance` was split into `getMasterPdo` and `getSlavePdo`
method change from `public function queryRow($fetchAssociative=true,$params=array())` to `public function queryOne($fetchMode = null)`
* there is no more `$model->unsetAttributes();` see https://github.com/yiisoft/yii2/issues/7742 , https://stackoverflow.com/questions/25131885/yii2-activerecord-how-to-unload-unset-a-model-of-allsome-attributes or just google for it
* AR relations there is only `hasOne` and `hasMany` left, see this https://toster.ru/q/246824
* in widgets you'll write your options a little bit different now (at least for `\yii\bootstrap\Button` and `bootstrap.widgets.TbButton` pair) 
```php
//was
$this->widget('bootstrap.widgets.TbButton', [
    'buttonType' => 'submit',
    'type' => 'primary',
]);
//now
echo \yii\bootstrap\Button::widget([
    'options' => [
        'type' => 'submit',
        'class' => 'btn btn-primary',
    ],
]);
```
* request
```php
//was
Yii::app()->request->isPostRequest
//now
Yii::$app->request->isPost
```
* `ActiveDataProvider` there is no need to provide first param now, also 
```php
//was
return new CActiveDataProvider($this, [
    'criteria' => $criteria,
]);
//now
return new ActiveDataProvider([
    'query' => $query,
]);
```

## Active record

* change name from `CActiveRecord` to  `yii\db\ActiveRecord`
* `public function tableName()` is now `public static function tableName()`
see [yii2](http://www.yiiframework.com/doc-2.0/guide-db-active-record.html#setting-a-table-name) and [yii1](http://www.yiiframework.com/doc/guide/1.1/en/database.ar#defining-ar-class)
* same for `public function primaryKey()` to `public static function primaryKey()`. Also you can drop it, it's resolved by framework now
* find works diffrent. 
```php
//was
User::model()->findAll()
//now
User::findAll()
```
* as far as I understood you can drop `public static function model()`
* view. `$this->pageTitle` was changed to `$this->title` 
* in `\yii\grid\GridView` (previously `bootstrap.widgets.TbGridView`) `filter` changed to `filterModel`
* in `\yii\widgets\DetailView` (previously `bootstrap.widgets.TbDetailView`) `data` changed to `model`
* also
```php
//was
//that's GridView
'columns' => [
        'id',
        [
            'header' => Yii::t('app', 'Something'),
            'value' => '$data->getSomething()'
        ],
        [
            'class' => 'bootstrap.widgets.TbButtonColumn',
        ],
    ],
//now
'columns' => [
    'id',
    [
        'header' => Yii::t('app', 'Something'),
        'value' => function ($data) {
                return $data->getSomething();
        }
    ],
    [
        'class' => \yii\grid\ActionColumn::class,
    ],
],
```
* model finding was changed, some examples
```php
//was
$models=User::model()->findAllByAttributes(['parent_id'=>$this->id]);
$model = User::model()->findByPk($id);
//now
$models = User::findAll(['parent_id' => $this->id]);
$model = User::findOne($id);
```
* view. there was $this->breadcrumbs now it works with `\yii\widgets\Breadcrumbs` and/or `$this->params['breadcrumbs']`. google for `yii1 breadcrumbs` and `yii2 breadcrumbs`
* console class name changed
```php
//was
class LogCommand extends CConsoleCommand {}
//now
use yii\console\Controller;
class LogCommand extends \yii\console\Controller {}
```
* console config. add `controllerNamespace` because without it, it will not find your commands
* console config. (todo other changes if they are included)
```php

```
* validation classes changes. If the `Internal change` is empty it might be because, that I didn't notice but they exist. See http://www.yiiframework.com/doc/api/1.1/CValidator and/or http://www.yiiframework.com/doc-2.0/yii-validators-validator.html


| Added(+)/deleted(-) | Old string representation | Old class                     | New string representation | New class                                    | Internal change                  |
|---------------------|---------------------------|-------------------------------|---------------------------|----------------------------------------------|----------------------------------|
|                     | boolean                   | `CBooleanValidator`           | boolean                   | `\yii\validators\BooleanValidator`           |                                  |
|                     | captcha                   | `CCaptchaValidator`           | captcha                   | `\yii\captcha\CaptchaValidator`              |                                  |
|                     | compare                   | `CCompareValidator`           | compare                   | `\yii\validators\CompareValidator`           |                                  |
|                     | date                      | `CDateValidator`              | date                      | `\yii\validators\DateValidator`              |                                  |
|                     | default                   | `CDefaultValueValidator`      | default                   | `\yii\validators\DefaultValueValidator`      |                                  |
|                     | email                     | `CEmailValidator`             | email                     | `\yii\validators\EmailValidator`             |                                  |
|                     | exist                     | `CExistValidator`             | exist                     | `\yii\validators\ExistValidator`             |                                  |
|                     | file                      | `CFileValidator`              | file                      | `\yii\validators\FileValidator`              |                                  |
|                     | filter                    | `CFilterValidator`            | filter                    | `\yii\validators\FilterValidator`            |                                  |
|                     | in                        | `CRangeValidator`             | in                        | `\yii\validators\RangeValidator`             |                                  |
|                     | match                     | `CStringValidator`            | match                     | `\yii\validators\RegularExpressionValidator` |                                  |
|                     | required                  | `CRegularExpressionValidator` | required                  | `\yii\validators\RequiredValidator`          |                                  |
|                     | safe                      | `CSafeValidator`              | safe                      | `\yii\validators\SafeValidator`              |                                  |
|                     | unique                    | `CUniqueValidator`            | unique                    | `\yii\validators\UniqueValidator`            |                                  |
|                     | url                       | `CUrlValidator`               | url                       | `\yii\validators\UrlValidator`               |                                  |
| +                   |                           |                               | string                    | `\yii\validators\StringValidator`            | it's kind of works like `length` |
| +                   |                           |                               | time                      | `\yii\validators\DateValidator`              |                                  |
| +                   |                           |                               | trim                      | `\yii\validators\FilterValidator`            |                                  |
| +                   |                           |                               | datetime                  | `\yii\validators\DateValidator`              |                                  |
| +                   |                           |                               | double                    | `\yii\validators\NumberValidator`            |                                  |
| +                   |                           |                               | each                      | `\yii\validators\EachValidator`              |                                  |
| +                   |                           |                               | image                     | `\yii\validators\ImageValidator`             |                                  |
| +                   |                           |                               | integer                   | `\yii\validators\NumberValidator`            |                                  |
| +                   |                           |                               | ip                        | `\yii\validators\IpValidator`                |                                  |
| -                   | length                    | `CStringValidator`            |                           |                                              | it's kind of works like `string` |
| -                   | numerical                 | `CNumberValidator`            |                           |                                              |                                  |
| -                   | type                      | `CTypeValidator`              |                           |                                              |                                  |
| -                   | unsafe                    | `CUnsafeValidator`            |                           |                                              |                                  |



## useful links
* http://www.yiiframework.com/doc-2.0/guide-intro-upgrade-from-v1.html
* http://www.yiiframework.com/news/77/yii-2-0-beta-is-released/
* http://www.yiiframework.com/news/80/yii-2-0-rc-is-released/
* http://www.yiiframework.com/news/81/yii-2-0-0-is-released/
* http://www.yiiframework.com/doc/api/
* http://www.yiiframework.com/doc-2.0/
* http://www.yiiframework.com/doc/guide/1.1/en/index
* http://www.yiiframework.com/doc-2.0/guide-README.html
* http://docs.mirocow.com/doku.php?id=yii2:migrate-yii1
* search for `yii1 to yii2 site:github.com` in google and in google but also in incognito window 
(you'll get other results because they'll unpersionalized)
* search for `yii2 site:github.com` in issues in yii1 github repo
* search for `yii1 site:github.com` in issues in yii2 github repo


## TODOs
* make diff (like validator) for - BaseYii, base namespace, cache namespace, console namespace,
i18n namespace, helpers/utils namespace, widgets namespace
* make those diffs with php (laravel/symfony console/laravel zero to generate markdown)
so I can just call cli command to do things

##
things that can be used/(get inspiration of) on building cli tool

https://github.com/FriendsOfPHP/PHP-CS-Fixer

https://github.com/squizlabs/PHP_CodeSniffer

ext-tokenizer

https://github.com/barryvdh/laravel-ide-helper

https://github.com/nikic/PHP-Parser

https://github.com/Roave/BetterReflection

https://github.com/illuminate/console

https://github.com/symfony/console

and others
