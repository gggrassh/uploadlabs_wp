# pass-1
## 代码分析

function checkFile() {
    //获取上传文件的名字
    var file = document.getElementsByName('upload_file')[0].value;
    if (file == null || file == "") {
        alert("请选择要上传的文件!");
        return false;
    }
    //定义允许上传的文件类型，白名单
    var allow_ext = ".jpg|.png|.gif";
    //提取上传文件的类型
    var ext_name = file.substring(file.lastIndexOf("."));
    //判断上传文件类型是否允许上传
    if (allow_ext.indexOf(ext_name + "|") == -1) {
        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
        alert(errMsg);
        return false;
    }
}
## 绕过方法
本关定义了允许上传的文件类型白名单，通过前端对上传的文件进行检查，有两种方法绕过验证上传php文件

### 禁用JavaScript
F12打开开发者工具→设置→禁用JavaScript，上传一句话木马文件，使用蚁剑连接

### burp suite抓包
修改文件后缀为jpg→上传文件并使用burp suite抓包→在bp中修改文件后缀为php，即可上传成功

# pass-2

## 代码分析

$is_upload = false;
//提示代码
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
    }
}
### 关键函数
$_FILES['upload_file']['type'] == 'image/jpeg'  

作用：

通过检测content-type判断上传文件内容

- content-type：通常是 HTTP 协议中用于标识消息主体内容类型的头部字段。在文件上传场景里，当客户端（如浏览器）向服务器发送文件时，会在请求头中包含 content-type 字段，用来告知服务器所发送内容的类型。不过在 PHP 中处理文件上传时，$_FILES['upload_file']['type'] 本质上是客户端在上传文件时提供的 MIME 类型信息，虽然和 content-type 有一定关联，但它并非直接的 content-type 字段。

$_FILES 超全局变量

在 PHP 里，当通过 HTML 表单上传文件时，上传的文件信息会被存储在 $_FILES 这个超全局变量中。$_FILES 是一个二维数组，其键名是 HTML 表单中 input 元素的 name 属性值。例如，若 HTML 表单中有一个 input 元素的 name 属性值为 upload_file，那么上传文件的相关信息就会存储在 $_FILES['upload_file'] 中。

$_FILES['upload_file']['type']

表示上传文件的 MIME 类型。

MIME（Multipurpose Internet Mail Extensions）类型是一种标准，用于表示文档、文件或字节流的性质和格式。在文件上传的场景中，它可以帮助我们识别上传文件的具体类型，比如图片、文本、音频等。

条件判断的意义

($_FILES['upload_file']['type'] == 'image/jpeg') 这个条件判断会检查上传文件的 MIME 类型是否为 image/jpeg，也就是判断上传的文件是否为 JPEG 格式的图像。如果是，则条件成立，返回 true；如果不是，则条件不成立，返回 false。

## 绕过方法
本题是通过检测上传文件的MIME内容判断文件类型，可以通过修改文件的content-type绕过。

上传一句话木马的php文件→burp suite抓包，修改content-type:image/jpeg实现绕过

或是同第一关上传jpg文件，抓包修改文件后缀为php

# pass-3

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        //黑名单
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

### 关键函数

trim()

用于去除字符串首尾的空白字符。

在代码中，trim($_FILES['upload_file']['name']) 用于去除上传文件名首尾的空白字符。

strrchr()

用于查找字符串中最后一次出现某个字符的位置，并返回从该位置到字符串末尾的所有字符。

在代码中，strrchr($file_name, '.') 用于获取上传文件的扩展名。

strtolower()

用于将字符串转换为小写。代码中使用它将文件扩展名转换为小写，以便进行不区分大小写的比较。

str_ireplace()

用于在字符串中进行不区分大小写的替换操作。

在代码中，str_ireplace('::$DATA', '', $file_ext) 用于去除文件扩展名中的 ::$DATA 字符串，这是为了防止 Windows 系统下的 NTFS 流攻击。

## 绕过方法
本题设置黑名单，过滤了.asp、.aspx、.php、.jsp文件，但是没有过滤php3 php5 phtml等文件类型，可以通过上传此类文件绕过过滤

此处需要修改服务器配置文件，以小皮面板为例：

设置→配置文件→httpd。conf→apache→修改语句#AddType application/x-httpd-php .php .phtml为AddType application/x-httpd-php .php .php3 .phtml→重启服务器

修改成功后上传php文件，抓包修改后缀为php3即可上传成功

# pass-4

## 代码分析


$is_upload = false;
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2","php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2","pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传!';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

## 扩展知识

.htaccess

● 作用：分布式配置文件，一般用于url重写、认证、控制访问等

● 作用范围：特定目录（一般是网站根目录），及其子目录

● 优先级：较高，可覆盖apache的主要配置文件（httpd-conf）

● 生效方式：修改后立刻生效

httpd-conf

● 作用：包含apache·HTTP服务器的全局行为和默认设置

● 作用范围：整个服务器

● 优先级：较低

● 生效方式：管理员权限，重启服务器后生效

Apache服务器中 AllowOverride 指令用于指定 .htaccess 文件可以覆盖哪些服务器配置。它的取值决定了用户通过 .htaccess 文件对服务器设置的控制程度。

AllowOverride 的常用值说明如下：

None：这是最严格的设置。当设置为 None 时，服务器将完全忽略并不允许 .htaccess 文件覆盖任何配置选项。这通常是为了提升性能和安全性。

All：这是最宽松的设置。当设置为 All 时，服务器允许 .htaccess 文件覆盖所有配置选项。这给予了目录最大的灵活性，但也需要更高的安全关注。

AuthConfig：此选项允许 .htaccess 文件覆盖与身份验证（Authorization） 相关的配置选项。常用的指令包括 AuthType、AuthName、AuthUserFile 等，常用于设置目录的密码保护。

FileInfo：此选项允许 .htaccess 文件覆盖与文件和文档类型 相关的配置选项。常用的指令包括 DirectoryIndex（默认首页）、DefaultType（默认MIME类型）、ErrorDocument（自定义错误页面）等。

Indexes：此选项允许 .htaccess 文件覆盖与目录索引（Indexing） 相关的配置选项。常用的指令包括 Options（用于启用或禁用目录列表等功能）、IndexOptions、IndexIgnore 等，这些指令控制着当目录中没有默认文件时，如何向访问者显示文件列表。

Limit：此选项允许 .htaccess 文件覆盖与访问控制（Access Control） 相关的配置选项，例如 Require 指令，可以基于IP地址、用户等因素来限制对目录的访问。

Options：此选项允许 .htaccess 文件覆盖与目录特性（Options） 相关的配置选项，主要是指 Options 指令本身，用于启用或禁用特定的目录功能，例如是否允许执行CGI脚本、是否支持服务器端包含（SSI）等。

## 绕过方法

### 方法一

在配置文件httpd-conf中修改AllowOverride none为AllowOverride ALL，创建一个.htaccess文件，内容为AddType application/x-httpd-php .jpg .txt（将txt文件与jpg文件当作php文件解析）

接着将木马文件后缀修改为jpg并上传，即可成功连接

### 方法二

分析代码功能

获取上传文件的文件名

$file_name = trim($_FILES['upload_file']['name']);

删除文件末尾的点

$file_name = deldot($file_name);

获取文件最后一个点后的内容，即文件后缀

$file_ext = strrchr($file_name, '.');

将文件后缀转换为小写

$file_ext = strtolower($file_ext);

去除字符串::$DATA

$file_ext = str_ireplace('::$DATA', '', $file_ext);

去除空格

$file_ext = trim($file_ext);

随后通过if语句判断经过以上处理的后缀名是否在黑名单内

可以在上传文件后使用burp suite抓包修改文件后缀为php. .，这样经过处理后（去除末尾的点，去空）的文件后缀名为php. ，不会被黑名单过滤，而Windows系统会自动去除文件末尾的点，达到上传php文件的效果

> linux不会自动去空，所以此方法在Linux环境下不可用

# pass-5

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

观察代码发现缺少语句$file_ext = strtolower($file_ext); //转换为小写 ，并且黑名单中过滤不完全

## 绕过方法

上传一句话木马php文件，抓包修改文件后缀为Php即可成功上传

# pass-6

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file,$img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

分析代码发现缺少语句$file_ext = trim($file_ext); //首尾去空 ，

## 绕过方法

可以上传php文件，使用burp suite抓包，在文件后添加空格，是后缀变为php 绕过黑名单过滤。Windows系统和Linux系统对文件后缀空格会自动去空，所以php 文件上传后能被正常执行

# pass-7

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

缺少语句$file_name = deldot($file_name);//删除文件名末尾的点

## 绕过方法

上传php文件，burp suite抓包修改后缀名为.php.，绕过黑名单过滤。Windows系统会自动去除文件末尾的点，将文件还原为.php文件，使得PHP代码能被正常执行

# pass-8

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

缺少语句$file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA 

## 扩展知识

**额外数据流**

在 Windows 操作系统中，如果你在文件名后看到 ::$DATA 的标识，这表示你正在查看该文件的一个 附加数据流。这是 NTFS 文件系统提供的一种特殊数据存储机制。

例如，1.txt是一个文件，1.txt::$DATA就是这个文件的一个附加数据流

1、什么是额外数据流？

如果将文件想象成一个容器，在大多数文件系统中，容器只有一个主隔间，里面存放着文件的实际内容。而 NTFS 文件系统允许这个容器拥有多个隔间，这些额外的“隔间”就是附加数据流。

- 默认数据流：我们平常通过文件名（如 document.txt）访问的就是这个主隔间，它的完整名称实际上是 document.txt::$DATA。

- 附加数据流：你可以创建其他命名的隔间，例如 document.txt:secret_stream:$DATA，用于存储与主文件相关的额外信息。

2. ::$DATA 的作用是什么？

附加数据流通常用于存储不会影响文件主要内容的元外信息，例如：

- 文件的元数据（如作者、标题等属性）。

- 备份信息。

- 安全标签或来源信息。

- 在某些情况下，也被软件（如 iTunes）用来存储特定数据。


3. 如何访问和使用？

对于普通用户和大多数常规文件操作工具（如 Windows 记事本、基础的文件管理器）来说，它们只能感知和操作文件的默认数据流，会自动忽略附加数据流。

要创建、查看或操作这些附加数据流，通常需要使用特定的命令行工具或编程接口。

示例，在命令提示符中使用 echo :-D > test.txt:helli.txt 可以创建一个附加数据流，使用 notepad test.txt:hello.txt 可以查看它。

## 过滤方法

本题不去除::$DATA，可以上传php文件后抓包修改后缀为.php::$DATA，验证时不匹配黑名单绕过过滤，成功上传后，系统则根据NTFS规则，自动忽略文件后的::$DATA，将其解释为php文件

注意连接时要删去::$DATA后缀

# pass-9

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

## 绕过