##### Mixin(混合器)声明
格式
```
    @mixin [混合器名]{...}
    @mixin size($[变量名]:[默认值],$[变量名]]){...}
    //exp:
    @mixin center{
      display: block;
      margin-left: auto;
      margin-right: auto;
    }
    @mixin size($width,$height:auto){
      width: $width;
      height: $height;
    }
```
##### 使用混合器
格式
```
    @include [混合器名];
    //exp:
    .box{
        @include center;
    }
    .content{
        @include center;
        @include size(300px);
    }
```
编译结果
```
    .box {
      display: block;
      margin-left: auto;
      margin-right: auto;
    }
    
    .content {
      width: 300px;
      height: auto;
      display: block;
      margin-left: auto;
      margin-right: auto;
    }
```
##### Placeholder声明
格式
```
    %[Placeholder名]{...}
    //exp:
    %center{
      display: block;
      margin-left: auto;
      margin-right: auto;
    }
```

##### 使用Placeholder
格式
```
    @extend %[Placeholder名];
    //exp:
    .box{
        @extend center;
    }
    .content{
        @extend center;
        @include size(300px);
    }
```
编译结果
```
    .box,.content {
      display: block;
      margin-left: auto;
      margin-right: auto;
    }
    
    .content {
      width: 300px;
      height: auto;
    }
```

##### @mixin VS %placeholder

+ 需要变量根据上下文生成特定的CSS，使用@mixin，反之使用%placeholder
+ 不需要变量根据上下文生成特定的CSS，优先使用%placeholder，因为Sass每次复制@mixin，会造成重复的CSS代码；

##### 函数声明
格式
```
    @function [函数名]($[变量名]:[默认值],$[变量名]]){...}
    //exp:
    $prefix: 'm';
    @function component($name,$modifier:null){
      @if $modifier == null {
        @return ".#{$prefix}-#{$name}"
      }@else{
        @return ".#{$prefix}-#{$name}-#{$modifier}"
      }
    }
```
##### 函数使用
格式
```
    @function [函数名]($[变量名]:[默认值],$[变量名]]){...}
    //exp:
    #{component(button)} {
      border: 1px solid red;
      &-icon {
        color: #555;
      }
    }
```
编译后
```
    .m-component-button {
      border: 1px solid red;
    }
    .m-component-button-icon {
      color: #555;
    }
```