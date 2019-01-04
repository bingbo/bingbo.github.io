通过GD和图像处理函数来生成

### 所涉及函数说明

__imagecreatetruecolor__

```php
resource imagecreatetruecolor ( int $width , int $height )
```

> 新建一个指定宽高的真彩色图像，返回一个图像标识符，代表了一幅大小为 x_size 和 y_size 的黑色图像

__imagecreate__

```php
resource imagecreate ( int $x_size , int $y_size )
```

> 新建一个基于调色板的图像，返回一个图像标识符，代表了一幅大小为 x_size 和 y_size 的空白图像

__imagecolorallocate__

```php
int imagecolorallocate ( resource $image , int $red , int $green , int $blue )
```

> 为一幅图像分配颜色,返回一个标识符，代表了由给定的 RGB 成分组成的颜色。red，green 和 blue 分别是所需要的颜色的红，绿，蓝成分。这些参数是 0 到 255 的整数或者十六进制的 0x00 到 0xFF。imagecolorallocate() 必须被调用以创建每一种用在 image 所代表的图像中的颜色

__imagestring__

```php
bool imagestring ( resource $image , int $font , int $x , int $y , string $s , int $col )
```

> 水平地画一行字符串,用 col 颜色将字符串 s 画到 image 所代表的图像的 x，y 坐标处（这是字符串左上角坐标，整幅图像的左上角为 0，0）。如果 font 是 1，2，3，4 或 5，则使用内置字体


__imagepng__

```php
bool imagepng ( resource $image [, string $filename ] )
```

> 将 GD 图像流（image）以 PNG 格式输出到标准输出（通常为浏览器），或者如果用 filename 给出了文件名则将其输出到该文件

__imagedestroy__

```php
bool imagedestroy ( resource $image )
```

> 销毁一图像,释放与 image 关联的内存。image 是由图像创建函数返回的图像标识符

__imageline__

```php
bool imageline ( resource $image , int $x1 , int $y1 , int $x2 , int $y2 , int $color )
```

> 画一条线段,用 color 颜色在图像 image 中从坐标 x1，y1 到 x2，y2（图像左上角为 0, 0）画一条线段

__imagesetpixel__

```php
bool imagesetpixel ( resource $image , int $x , int $y , int $color )
```

> 画一个单一像素,在 image 图像中用 color 颜色在 x，y 坐标（图像左上角为 0，0）上画一个点

其他函数可以查看：http://php.net/manual/zh/ref.image.php

### 实例

这里是用GD和图片相关函数生成图片验证码，主要用到了imagexxx等相应的函数据，如下所示:

```php
<?php

//创建一个真彩色图像
$im = @imagecreatetruecolor(120, 20)
      or die('Cannot Initialize new GD image stream');
//创建画板
//$im = @imagecreate(110, 20)
//    or die("Cannot Initialize new GD image stream");
//创建背景色
$background_color = imagecolorallocate($im, 0, 0, 0);
//创建文本颜色
$text_color = imagecolorallocate($im, 233, 14, 91);
//在一定的坐标上绘制一个字符串
imagestring($im, 5, 5, 5,  "Text String", $text_color);
header('Content-Type: image/jpeg');
//输出png格式图像
imagepng($im);
//销毁图像并释放内存
imagedestroy($im);
```

添加点、线干扰，更复杂的验证码，如：

```php
<?php
//创建图片
$im = imagecreate($x=130, $y=45);
//第一次调用给基于调色板的图像填充背景色
$bg = imagecolorallocate($im, rand(50, 200), rand(0, 155), rand(0, 155));
//字体颜色
$fontColor = imagecolorallocate($im, 255, 255, 255);
//产生随机字符
for($i = 0; $i < 4; $i ++){
    $randNumArr = array(rand(48, 57), rand(65, 90));
    $randNum = $randNumArr[rand(0, 1)];
    $randStr = chr($randNum);
    imagestring($im, 30, 5 + $i * 30, rand(0, $y/2), $randStr, $fontColor);
    $authCode .= $randStr;//记录相应的验证码放session里以便验证
}

//干扰线
for($i = 0; $i < 8; $i ++){
    $lineColor = imagecolorallocate($im, rand(0, 255), rand(0, 255), rand(0, 255));
    imageline($im, rand(0, $x), rand(0, $y), rand(0, $x), rand(0, $y), $lineColor);
}

//干扰点
for($i = 0; $i < 250; $i ++){
    imagesetpixel($im, rand(0, $x), rand(0, $y), $fontColor);
}
header('Content-type: image/png');
imagepng($im);
imagedestroy($im);
```