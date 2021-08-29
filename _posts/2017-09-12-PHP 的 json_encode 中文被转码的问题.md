---
layout: post
categories: PHP
tags: [PHP]
---

在 php5.2 中做 `json_encode` 的时候。中文会被 `unicode` 编码，php5.3 加入了 `options` 参数，5.4 以后才加入 `JSON_UNESCAPED_UNICODE` 这个参数，不需要做 `escape` 和 `unicode` 处理。 所以在5.4之前都需要对中文做个处理。 

## 5.4 里面的处理

```
json_encode($str, JSON_UNESCAPED_UNICODE);
```

## 5.4 之前的处理

方法1：在实际应用中有个问题，部分字符会掉，不知为何，如字符串："日期11.2" 会被变成 "日期.2" 

```
function encode_json($str){  
    $code = json_encode($str);  
    return preg_replace("#\\\u([0-9a-f]+)#ie", "iconv('UCS-2', 'UTF-8', pack('H4', '\\1'))", $code);  
}  
```

方法2：先对需要处理的做 `urlencode` 处理，然后 `json_encode`，最后做 `urldecode` 处理 

```
function encode_json($str) {  
    return urldecode(json_encode(url_encode($str)));      
}  
  
function url_encode($str) {  
    if(is_array($str)) {  
        foreach($str as $key=>$value) {  
            $str[urlencode($key)] = url_encode($value);  
        }  
    } else {  
        $str = urlencode($str);  
    }  
      
    return $str;  
}
```