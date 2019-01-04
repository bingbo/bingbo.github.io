用PHP来导出excel表格这里有三种不同的方法

### 用页面html等来控制输出表格样式

__服务端数据处理部分__

```php
header("Content-type: application/vnd.ms-excel; charset=utf-8");
header("Content-Disposition: attachment; filename=test.xls");
header("Expires: Mon, 26 Jul 1997 05:00:00 GMT");
header("Cache-Control: no-cache");
header("Pragma:no-cache");

$this->smarty->assign('data',$result);
$this->smarty->display('extract/board/excel.tpl');
```

__页面渲染部分__

```html
<html xmlns:o="urn:schemas-microsoft-com:office:office" 
xmlns:x="urn:schemas-microsoft-com:office:excel" 
xmlns="http://www.w3.org/TR/REC-html40"> 
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 
<head> 
<meta http-equiv="Content-type" content="text/html;charset=UTF-8" /> 
<style id="Classeur1_16681_Styles"></style> 
</head> 
<body> 
<div id="Classeur1_16681" align=center x:publishsource="Excel"> 
<table x:str border=1 cellpadding=0 cellspacing=0 width=100% style="border-collapse: collapse"> 
<tr>
{%foreach $data.attributes as $key => $value%}
<td>{%$value.name%}</td>
{%/foreach%}
</tr>
{%foreach $data.taskData.info as $index => $item%}
<tr>
{%foreach $data.attributes as $akey => $attr%}
<td class=xl2216681 nowrap>{%$item.info[{%$attr.ename%}]%}</td>
{%/foreach%}
</tr>
{%/foreach%}
</table> 
</div> 
</body> 
</html>
```

> 通过页面的形式去展现生成excel表格可以避免生成的表格内容为乱码的情况

### 直接用原生php echo输出生成excel

```php
<?php
    header('Content-Type: application/vnd.ms-excel');
    header('Content-Disposition: attachment;filename="orderinfo.xls"');
    header('Cache-Control: max-age=0');

    //输出excel列名信息
    $head = array(
        "name",
        "password",
        "sex",
        "age",
        "phone",
        "姓名",
        "......"
    );

    foreach($head as $k => $h){
        //统一转换为GBK编码，防止中文乱码
        $head[$k] = iconv('utf-8', 'gbk', $h);
    }
    echo implode("\t", $head);
    echo "\n";
    while($cnt < 100){
        $cnt ++;

        for($j = 0; $j < 6; $j ++){
            $row[$j] = $j;
        }
        echo implode("\t", $row);
        echo "\n";
    }
```

### 用php fputcsv生成excel

```php
<?php
    header('Content-Type: application/vnd.ms-excel');
    header('Content-Disposition: attachment;filename="orderinfo.xls"');
    header('Cache-Control: max-age=0');

    //输出到浏览器
    $fp = fopen('php://output', 'a');

    //输出excel列名信息
    $head = array(
        "name",
        "password",
        "sex",
        "age",
        "phone",
        "......"
    );

    foreach($head as $k => $h){
        //csv的excel支持GBK编码，一定要转换，否则乱码
        $head[$k] = iconv('utf-8', 'gbk', $h);
    }
    //将数据通过fputcsv写到文件句柄
    fputcsv($fp, $head, "\t");
    //计数器
    $cnt = 0;

    //每隔100000行，刷新一下输出buffer,不要太大，也不要太小
    $limit = 100000;
    while(true){
        if($cnt > 10000000) break;
        $cnt ++;
        if($limit == $cnt){
            //刷新一下输出buffer,防止由于数据过多造成问题
            ob_flush();
            flush();
            $cnt = 0;
        }

        for($j = 0; $j < 6; $j ++){
            $row[$j] = $j;
        }
        fputcsv($fp, $row, "\t");
    }
```

> 这里用fputcsv输出内容，但保存为orderinfo.xls名，这样每个元素会保存在一个单独的单元格里，如果保存名为*.csv的格式，则每行所有元素会保存在一个单元格里且以,或制表符进行分隔
