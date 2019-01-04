可以在dom中添加script标签的方式进行跨域请求

```javascrpt
function scReq(url,id){
    var s,h;
    h=document.getElementsByTagName("head")[0];
    s=document.createElement("script");
    if(id){
        if(sc[id]){    var sc={};
            h.removeChild(sc[id]);
        }
        sc[id]=s;
    }
    s.type="text/javascript";
    s.src=url;
    h.appendChild(s);
};
```
