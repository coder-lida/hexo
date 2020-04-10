title: SpringMVC请求参数获取的几种方法
tags:
  - Spring
categories:
  - Dev
  - Frame
date: 2019-08-15 11:08:00
cover: true

---
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/spring.jpg)
<!-- more -->

SpringMVC请求参数获取的几种方法

## 通过@PathVariabl获取路径中的参数

```
    @RequestMapping(value="user/{id}/{name}",method=RequestMethod.GET)
    public String printMessage1(@PathVariable String id,@PathVariable String name, ModelMap model) {
        
        System.out.println(id);
        System.out.println(name);
        model.addAttribute("message", "111111");
        return "users";
    }
```

例如，访问user/123/lei路径时，执行以上方法，其中，参数id=123，name=lei

## @ModelAttribute获取POST请求的FORM表单数据

表单如下

```
 <form method="post" action="hao.do">
    a: <input id="a" type="text"   name="a"/>
    b: <input id="b" type="text"   name="b"/>
    <input type="submit" value="Submit" />
 </form>
```

Java  Pojo如下

```
    public class Pojo{
        private String a;
        private int b;
    }
```

Java Controller如下

```
@RequestMapping(method = RequestMethod.POST) 
public String processSubmit(@ModelAttribute("pojo") Pojo pojo) { 
    
    return "helloWorld"; 
}
```

## @RequestBody获取POST请求的FORM表单数据

`@RequestBody`接收的是一个Json对象的字符串，而不是一个`Json`对象。然而在`ajax`请求往往传的都是`Json`对象，后来发现用 `JSON.stringify(data)`的方式就能将对象变成字符串。同时`ajax`请求的时候也要指定`dataType: "json",contentType:"application/json"`这样就可以轻易的将一个对象或者`List`传到`Java`端，使用`@RequestBody`即可绑定对象或者`List`.

js代码

```
<script type="text/javascript">  
    $(document).ready(function(){  
        var saveDataAry=[];  
        var data1={"userName":"test","address":"gz"};  
        var data2={"userName":"ququ","address":"gr"};  
        saveDataAry.push(data1);  
        saveDataAry.push(data2);         
        $.ajax({ 
            type:"POST", 
            url:"user/saveUser", 
            dataType:"json",      
            contentType:"application/json",               
            data:JSON.stringify(saveData), 
            success:function(data){ 

            } 
         }); 
    }); 

```

java代码

```
    @RequestMapping(value = "saveUser", method = {RequestMethod.POST }}) 
    @ResponseBody  
    public void saveUser(@RequestBody List<User> users) { 
         userService.batchSave(users); 
    }
```
`@ModelAttribute`和`@RequestBody`注解不同之处在于`@ModelAttribute`注解可以在前端直接获取返回值

```
@Controller
public class Hello2ModelController extends BaseController {

    @RequestMapping(value = "/helloWorld2")  
    public String helloWorld(@ModelAttribute("myUser") User user) {
        user.setName("老王");
       return "helloWorld";  
    }  
}
```

`model`中`key`为`myUser` ,前台可以直接通过`${myUser.xx}`获取`user`相应属性

## 直接用HttpServletRequest获取

```
@RequestMapping(method = RequestMethod.GET) 
public String get(HttpServletRequest request, HttpServletResponse response) { 
   System.out.println(request.getParameter("a")); 
    return "helloWorld"; 
}
```

## 用注解@RequestParam绑定请求参数

用注解`@RequestParam`绑定请求参数a到变量a

当请求参数a不存在时会有异常发生,可以通过设置属性`required=false`解决,

例如: `@RequestParam(value="a", required=false)`

Controller如下

```
@RequestMapping(value = "/requestParam", method = RequestMethod.GET) 
public String setupForm(@RequestParam("a") String a, ModelMap model) { 
   System.out.println(a); 
return "helloWorld";
}

```