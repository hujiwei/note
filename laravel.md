# 数据库操作-查询构造器

## 新增数据
```php
// 新增数据
$bool = DB::table('table1')->insert(
  ['name' => 'name1', 'age' => 18],
  // 多条数据
  ['name' => 'name2', 'age' => 19]
);
var_dump($bool);

// 新增数据并返回自增id
$id = DB::table('table1')->insertGetId(
  ['name' => 'name1', 'age' => 18]
);
var_dump($id);
```

## 更新数据
```php
// 更新数据
$num = DB::table('table1')->where('id', 12)->update(
  ['name' => 'name1', 'age' => 18]
);
var_dump($num);

// 自增，默认是1
$num = DB::table('table1')->increment('age', 3);
// 自减，默认是1
$num = DB::table('table1')->decrement('age', 3);
// 带条件的自减
$num = DB::table('table1')
  ->where('id', 12)
  ->decrement('age', 3);
// 带条件的自减，同时修改其他字段
$num = DB::table('table1')
  ->where('id', 12)
  ->decrement('age', 3, ['name' => '123']);
var_dump($num);
```

## 删除数据
```php
// 删除数据，删除table1表中id=12的数据
$num = DB::table('table1')->where('id', 12)->delete();
// 删除数据，删除table1表中id>=12的数据
$num = DB::table('table1')->where('id', '>=', 12)->delete();
var_dump($num);

// 删除整个表数据，无返回值，危险操作，谨慎使用
DB::table('table1)->truncate();
```

## 查询数据
```php
// get()获取表的所以数据
$tables = DB::table('table1')->get();
dd($tables);
// first()获取结果集中的第一条数据
// table1表倒序排序，并获取结果集中的第一条数据
$table = DB::table('table1')
  ->orderBy('id', 'desc')
  ->first();
dd($table);
// where()条件查询
$table1s = DB::table('table1')
  ->where('id', '>=' ,10)
  ->get();
// 多条件查询，id>=10并且age>18
$table2s = DB::table('table1')
  ->whereRaw('id >= ? and age > ?', [10, 18])
  ->get();
dd($table1s);
// pluck()返回结果集中的字段name
$names = DB::table('table1')
  ->pluck('name');
dd($names);
// lists()返回结果集中的字段name，指定id作为下标
$names1 = DB::table('table1')
  ->lists('name', 'id');
dd($names1);
// select()返回结果集中指定字段
$tables1 = DB::table('table1')
  ->select('id', 'name', 'age')
  ->get();
dd($tables1);
// chunk()分段获取，每次查询2条
DB::table('table1')->chunk(2, function($tables2){
  var_dump($tables2);
  return false; 
})
```

## 聚合函数
```php
// count()返回数据集条数
$num = DB::table('table1')->count();
var_dump($num);
// max()返回数据集中age字段的最大值
$max = DB::table('table1')->max('age');
var_dump($max);
// min()返回数据集中age字段的最小值
$min = DB::table('table1')->min('age');
var_dump($min);
// avg()返回数据集中age字段的平均值
$avg = DB::table('table1')->avg('age');
var_dump($avg);
// sum()返回数据集中age字段的总和
$sum = DB::table('table1')->sum('age');
var_dump($sum);
```

# ORM
## 建立模型
```php
<?php
// 命名空间
namespace Fir\RiverChiefOffice\Models;
use Illuminate\Database\Eloquent\Model;

class Student extends Model{
  // 指定表名
  protected $table = 'Student';
  // 指定主键为id
  protected $primaryKey = 'id';
  // 自动维护时间戳
  public $timestamps = true;
  protected function getDateFormat(){
    return time();
  }
  // 不要格式化时间戳
  protected function asDateTime($val){
    return $val;
  }
  // 允许批量赋值的字段
  protected $fillable = [
    'name',
    'age'
  ];
  // 指定某个字段不允许批量赋值的字段
  protected $guarded = [
    'name'
  ];
}
```

## 使用模型
```php
// all() 查询所有记录，返回一个集合
$students = Student::all();
dd($students);
// find() 根据主键查找，查询单个数据，查询id为10的数据
$student = Student::find(10);
dd($student);
// findOrFail() 根据主键查找，如果没查到就抛出异常
$student1 = Student::findOrFail(10);
dd($student1);
// 查询构造器在ORM中的使用
$student2 = Student::get();
dd($student2);
// 条件查询
$student3 = Student::where('id', '>', '10')->orderBy('age', 'desc')->first();
dd($student3);
// ORM中用chunk()
Student::chunk(2, function($students){
  var_dump($students)
});
dd($students);
// 聚合函数
$num = Student::count();
var_dump($num);
$max = Student::where('id', '>', '10')->max('age');
var_dump($max);
```

## ORM中的新增，自定义时间戳及批量赋值
```php
// 使用模型新增数据并保存到数据库
$student = new Student();
$student->name = 'sean';
$student->age = 18;
$bool = $student->save();
dd($bool);
```
自定义时间戳，在模型中加入代码
```php
// 自动维护时间戳
public $timestamps = true;
```
自己格式化时间戳
```php
// 自己格式化时间戳
$student = Student::find(10);
echo date('Y-m-d H:i:s', $student->created_at);
```

```php
// 使用模型的create方法新增数据
$student = Student::create(
  ['name' => 'sean', 'age' => 18]
)
dd($student);
// firstOrCreate() 以属性查找，如果没有则新增
$student1 = Student::firstOrCreate(
  ['name' => 'sean']
)
dd($student1);
// firstOrNew() 以属性查找，如果没有则新增实例，如果需要保存用save()
$student2 = Student::firstOrNew(
  ['name' => 'sean']
)
$bool = $student2->save();
dd($bool);
```

### 使用ORM修改数据
```php
$student = Student::find(10);
$student->name = 'sean';
$student->age = 18;
$bool = $student->save();
dd($bool);
// 批量更新
$num = Student::where('id', '>', 10)->update(
  ['age' => 21]
);
var_dump($num);
```

### 使用ORM删除数据
```php
// 通过模型删除
$student = Student::find(10);
$bool = $student->delete();
dd($bool);
// 通过主键删除
$num = Student::destroy(10);
$num1 = Student::destroy([12, 11]);
var_dump($num);
// 批量删除
$num2 = Student::where('id', '>', 10)->delete();
var_dump($num2);
```