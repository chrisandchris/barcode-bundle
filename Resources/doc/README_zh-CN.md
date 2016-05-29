# SGKBarcodeBundle

[![Build Status](https://travis-ci.org/shangguokan/SGKBarcodeBundle.svg)](https://travis-ci.org/shangguokan/SGKBarcodeBundle)
[![Latest Stable Version](https://poser.pugx.org/sgk/barcode-bundle/v/stable)](https://packagist.org/packages/sgk/barcode-bundle) [![Total Downloads](https://poser.pugx.org/sgk/barcode-bundle/downloads)](https://packagist.org/packages/sgk/barcode-bundle) [![Latest Unstable Version](https://poser.pugx.org/sgk/barcode-bundle/v/unstable)](https://packagist.org/packages/sgk/barcode-bundle) [![License](https://poser.pugx.org/sgk/barcode-bundle/license)](https://packagist.org/packages/sgk/barcode-bundle)

SGKBarcodeBundle 是一个用于生成条形码和二维码的 Symfony2 / Symfony3 Bundle。
这份 README 还有英语版（[English](https://github.com/shangguokan/SGKBarcodeBundle)）和法语版（[Français](README_fr.md)）。

特点：

1. 支持 3 种二维码和 30 种条形码类型
2. 可输出三种不同格式：HTML，PNG 和 SVG canvas
3. 集成 Twig：你可以方便的使用一个 Twig 扩展函数，直接在模板中进行调用来显示条形码和二维码
4. 这个 Bundle 移植于这个项目：[tc-lib-barcode](https://github.com/tecnickcom/tc-lib-barcode)

![SGKBarcodeBundle](README.png)

## 安装

执行这条指令来安装 SGKBarcodeBundle：
```sh
// Symfony version >= 3.0
$ php composer.phar require sgk/barcode-bundle:~3.0

// Symfony version >= 2.7 and < 3.0, use ~2.0
// Symfony version < 2.7, use ~1.0
```

或者，把 SGKBarcodeBundle 依赖添加到你的 ``composer.json`` 中，然后执行 ``php composer.phar update`` ：
```json
// Symfony version >= 3.0
"require": {
        "sgk/barcode-bundle": "~3.0"
    }

// Symfony version >= 2.7 and < 3.0, use ~2.0
// Symfony version < 2.7, use ~1.0
```

Composer 会把 Bundle 安装到你项目下的 vendor/sgk 文件夹中。

然后在 kernel 中注册这个 Bundle ：
```php
<?php
// app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        // ...
        new SGK\BarcodeBundle\SGKBarcodeBundle(),
    );
}
```

## 生成参数

共有 5 个参数可用于配置来生成条形码和二维码：

|参数   |类型        |是否必填 |允许的值          |描述                 |
|:----:|:---------:|:------:|:------------:|:-------------------:|
|code  |string     |必填     |              |要进行编码的信息        |
|type  |string     |必填     |[支持的条码类型](#支持的条形码和二维码类型)|条形码和二维码的类型|
|format|string     |必填     |html, svg, png|输出格式|
|width |**integer**|可选     |              |**单元宽度**|
|height|**integer**|可选     |              |**单元高度**|
|color |html和svg为string / png为array|可选|[HTML Color Names](http://www.w3schools.com/html/html_colornames.asp) / array(R, G, B)|颜色|

> 二维条码的默认宽高为5,5。一维条码的默认宽高为2,30。
> html，svg输出格式的默认颜色为 black。png输出格式的默认颜色为 array(0, 0, 0)。

## 用例：通过 service 使用

这个 Bundle 注册了一个 service ： ``sgk_barcode.generator`` ，你可以通过 Symfony 的服务容器来获得它并生成条码：

* 输出 html
```php
$options = array(
    'code'   => 'string to encode',
    'type'   => 'c128',
    'format' => 'html',
);

$barcode =
    $this->get('sgk_barcode.generator')->generate($options);
    
return new Response($barcode);
```

* 输出 svg
```php
$options = array(
    'code'   => 'string to encode',
    'type'   => 'qrcode',
    'format' => 'svg',
    'width'  => 10,
    'height' => 10,
    'color'  => 'green',
);

$barcode =
    $this->get('sgk_barcode.generator')->generate($options);
    
return new Response($barcode);
```

* 输出 png
```php
$options = array(
    'code'   => 'string to encode',
    'type'   => 'datamatrix',
    'format' => 'png',
    'width'  => 10,
    'height' => 10,
    'color'  => array(127, 127, 127),
);

$barcode =
    $this->get('sgk_barcode.generator')->generate($options);

return new Response('<img src="data:image/png;base64,'.$barcode.'" />');
```
> 对于 png 格式，生成器返回的是 png 图片的 based64 数据，你可以通过``base64_decode($barcode)``来获得原始数据。在这里我们利用 [Data URI scheme](http://en.wikipedia.org/wiki/Data_URI_scheme) 来将base64数据直接内嵌并显示到网页上。

## 用例：在 Twig 模板中使用

这个 Bundle 扩展了一个 Twig 函数 ``barcode`` ，你可以直接在 Twig 中调用它来生成条码。

``barcode`` 函数使用和上面一样的参数，唯一不同的是你的传参是一个 [Twig 数组](http://twig.sensiolabs.org/doc/templates.html#literals)（它看起来很像 Json ，但它不是。。。）

* 显示 html
```twig
{{ barcode({code: 'string to encode', type: 'c128', format: 'html'}) }}
```

* 显示 svg
```twig
{{ barcode({code: 'string to encode', type: 'qrcode', format: 'svg', width: 10, height: 10, color: 'green'}) }}
```

* 显示 png
```twig
<img src="data:image/png;base64,
{{ barcode({code: 'string to encode', type: 'datamatrix', format: 'png', width: 10, height: 10, color: [127, 127, 127]}) }}
" />
```

## 用例：不通过 service 使用

```php
use SGK\BarcodeBundle\Generator\Generator;
//...
$options = array(
    'code'   => 'string to encode',
    'type'   => 'qrcode',
    'format' => 'html',
);

$generator = new Generator();
$barcode = $generator->generate($options);

return new Response($barcode);
```

## 将生成的条码存储到文件

你已经看到，这个 Bundle 不会在文件系统上存储任何文件，但是如果你想把条码存到文件，也是没有问题的：

* 存储为 html
```php
$savePath = '/tmp/';
$fileName = 'sample.html';

file_put_contents($savePath.$fileName, $barcode);
```

* 存储为 svg
```php
$savePath = '/tmp/';
$fileName = 'sample.svg';

file_put_contents($savePath.$fileName, $barcode);
```

* 存储为 png
```php
$savePath = '/tmp/';
$fileName = 'sample.png';

file_put_contents($savePath.$fileName, base64_decode($barcode));
```

## 支持的条形码和二维码类型

阅读[维基百科页面](http://en.wikipedia.org/wiki/Barcode)来了解你应该用哪一种条码。 

### 二维码

|type      |Name                                                   |Example(encode 123456)|
|:--------:|:-----------------------------------------------------:|:--------------------:|
|qrcode    |[QR code](http://en.wikipedia.org/wiki/QR_code)        |![](barcode/qrcode.png)|
|pdf417    |[PDF417](http://en.wikipedia.org/wiki/PDF417)          |![](barcode/pdf417.png)|
|datamatrix|[Data Matrix](http://en.wikipedia.org/wiki/Data_Matrix)|![](barcode/datamatrix.png)|

### 条形码

|type    |Symbology                                              |Example(encode 123456)|
|:------:|:-----------------------------------------------------:|:--------------------:|
|c39     |[Code 39](http://en.wikipedia.org/wiki/Code_39)        |![](barcode/c39.png)|
|c39+    |Code 39 CHECK_DIGIT                                    |![](barcode/c39+.png)|
|c39e    |Code 39 EXTENDED                                       |![](barcode/c39e.png)|
|c39e+   |Code 39 EXTENDED CHECK_DIGIT                           |![](barcode/c39e+.png)|
|c93     |[Code 93](http://en.wikipedia.org/wiki/Code_93)        |![](barcode/c93.png)|
|s25     |[Standard 2 of 5](http://www.barcodeisland.com/2of5.phtml)           |![](barcode/s25.png)|
|s25+    |Standard 2 of 5 CHECK_DIGIT                                          |![](barcode/s25+.png)|
|i25     |[Interleaved 2 of 5](http://en.wikipedia.org/wiki/Interleaved_2_of_5)|![](barcode/i25.png)|
|i25+    |Interleaved 2 of 5 CHECK_DIGIT                                       |![](barcode/i25+.png)|
|c128    |[Code 128](http://en.wikipedia.org/wiki/Code_128)                    |![](barcode/c128.png)|
|c128a   |Code 128A|![](barcode/c128a.png)|
|c128b   |Code 128B|![](barcode/c128b.png)|
|c128c   |Code 128C|![](barcode/c128c.png)|
|ean2    |[EAN 2](http://en.wikipedia.org/wiki/EAN_2)                 |![](barcode/ean2.png)|
|ean5    |[EAN 5](http://en.wikipedia.org/wiki/EAN_5)                 |![](barcode/ean5.png)|
|ean8    |[EAN 8](http://en.wikipedia.org/wiki/EAN-8)                 |![](barcode/ean8.png)|
|ean13   |[EAN 13](http://en.wikipedia.org/wiki/EAN-13)               |![](barcode/ean13.png)|
|upca    |[UPC-A](http://en.wikipedia.org/wiki/Universal_Product_Code)|![](barcode/upca.png)|
|upce    |[UPC-B](http://en.wikipedia.org/wiki/Universal_Product_Code)|![](barcode/upce.png)|
|msi     |[MSI](http://en.wikipedia.org/wiki/MSI_Barcode)             |![](barcode/msi.png)|
|msi+    |MSI CHECK_DIGIT                                             |![](barcode/msi+.png)|
|postnet |[POSTNET](http://en.wikipedia.org/wiki/POSTNET)             |![](barcode/postnet.png)|
|planet  |[PLANET](http://en.wikipedia.org/wiki/Postal_Alpha_Numeric_Encoding_Technique)|![](barcode/planet.png)|
|rms4cc|[RMS4CC](http://en.wikipedia.org/wiki/RM4SCC)    |![](barcode/rms4cc.png)|
|kix     |[KIX-code](http://nl.wikipedia.org/wiki/KIX-code)|![](barcode/kix.png)|
|imb     |[IM barcode](http://en.wikipedia.org/wiki/Intelligent_Mail_barcode)|![](barcode/imb.png)|
|codabar |[Codabar](http://en.wikipedia.org/wiki/Codabar)                    |![](barcode/codabar.png)|
|code11  |[Code 11](http://en.wikipedia.org/wiki/Code_11)                    |![](barcode/code11.png)|
|pharma  |[Pharmacode](http://en.wikipedia.org/wiki/Pharmacode)              |![](barcode/pharma.png)|
|pharma2t|Pharmacode Two-Track                                               |![](barcode/pharma2t.png)|

## Requirements

如果你遇到了依赖问题，请检查你的 phpinfo()，查看你是否安装了以下两个 PHP 扩展。（一般情况下都是已默认安装的）

- 需要 [GD](http://php.net/manual/en/book.image.php) 和 [ImageMagick](http://php.net/manual/en/book.imagick.php) 来生成 PNGs in PHP 5.3。
- 需要 [PHP bcmath](http://php.net/manual/en/book.bc.php) 来生成 Intelligent Mail barcodes（IMB格式）。

## 测试

执行单元测试：
```sh
$ phpunit --coverage-text
```
