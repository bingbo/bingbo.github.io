## fopen打开文件，fread全部读取内容

```php
header("content-type:text/html;charset=utf-8");  
//文件路径  
$file_path="text.txt";  
//判断是否有这个文件  
if(file_exists($file_path)){  
   if($fp=fopen($file_path,"a+")){  
      //读取文件  
      $conn=fread($fp,filesize($file_path));  
      //替换字符串  
      $conn=str_replace("\r\n","<br/>",$conn);  
      echo $conn."<br/>";  
   }else{  
      echo "文件打不开";  
   }  
}else{  
   echo "没有这个文件";  
}  
fclose($fp); 
```

## file_get_contents全部读取内容

```php
header("content-type:text/html;charset=utf-8");  
//文件路径  
$file_path="text.txt";  
$conn=file_get_contents($file_path);  
$conn=str_replace("\r\n","<br/>",file_get_contents($file_path));  
echo $conn;  
fclose($fp);  
```

## fopen打开文件，fread循环读取内容

```php
header("content-type:text/html;charset=utf-8");  
//文件路径  
$file_path="text.txt";  
//判断文件是否存在  
if(file_exists($file_path)){  
   //判断文件是否能打开  
   if($fp=fopen($file_path,"a+")){  
      $buffer=1024;  
      //边读边判断是否到了文件末尾  
      $str="";  
      while(!feof($fp)){  
         $str.=fread($fp,$buffer);  
      }  
   }else{  
      echo "文件不能打开";  
   }  
}else{  
   echo "没有这个文件";  
}  
//替换字符  
$str=str_replace("\r\n","<br>",$str);  
echo $str;  
```