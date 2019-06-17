---
title: CI3.0.2支持smarty3.1.29
date: 2016-06-02 16:37:12
tags: [PHP]
categories: [技术]
---

1. 下载smarty并解压复制libs文件夹到CI的applica/libraries文件夹中，将libs文件夹重命名为smarty
2. 在application/libraries文件夹下新建Ci_smarty.php文件，在其中加入以下代码：

```php
<?php

if(!defined('BASEPATH')) exit('No direct script access allowed');

require(APPPATH.'libraries/smarty/Smarty.class.php');

class Ci_smarty extends Smarty
{
    protected $ci;
    public function __construct()
    {
        parent::__construct();
        $this->ci = & get_instance();
        $this->ci->load->config('smarty');//加载smarty的配置文件

        //获取相关的配置项
        $this->cache_lifetime  = $this->ci->config->item('cache_lifetime');
        $this->caching         = $this->ci->config->item('caching');
        $this->template_dir    = $this->ci->config->item('template_dir');
        $this->compile_dir     = $this->ci->config->item('compile_dir');
        $this->cache_dir       = $this->ci->config->item('cache_dir');
        $this->use_sub_dirs    = $this->ci->config->item('use_sub_dirs');
        $this->left_delimiter  = $this->ci->config->item('left_delimiter');
        $this->right_delimiter = $this->ci->config->item('right_delimiter');
    }
}
```
<!--more-->
3. 在application/config目录下新建smarty.php文件，在其中加入以下代码：

```php
<?php

if(!defined('BASEPATH')) exit('No direct script access allowed');

$config['cache_lifetime']   = 30*24*3600;
$config['caching']          = false;
$config['template_dir']     = APPPATH .'views';
$config['compile_dir']      = APPPATH .'views/template_c';
$config['cache_dir']        = APPPATH . 'views/cache';
$config['use_sub_dirs']     = false;		//子目录变量(是否在缓存文件夹中生成子目录)
$config['left_delimiter']   = '{';
$config['right_delimiter']  = '}';
```
4. 在application/core目录下新建MY_Controller.php文件，在其中加入以下代码：

```php
<?php

if (!defined('BASEPATH')) exit('No direct access allowed.');

class MY_Controller extends CI_Controller{

	public function __construct(){
		parent::__construct();
	}

	public function assign($key,$val){
		$this->ci_smarty->assign($key,$val);
	}

	public function display($html){
		$this->ci_smarty->display($html);
	}
}
```
5. 在application/config/autoload.php文件中找到`$autoload['libraries']=array();`，在其中加入对Ci_smarty类的自动加载

6. 使用以下方法测试CI是否支持smarty，修改application/controller/Welcome.php如下所示：

```php
<?php
if ( ! defined('BASEPATH')) exit('No direct script access allowed');

class Welcome extends MY_Controller
{
    public function index() {
        $test='ci 3.0.2 + smarty 3.1.29 配置成功';
        $this->assign('test',$test);
        $this->display('test.html');
    }
}
```
在application/views下新建smarty_test.html，在其中加入以下代码：

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>smarty配置测试</title>
</head>
<body>
    <{$test}>
</body>
</html>
```
若能正确输出结果，则说明配置正确
