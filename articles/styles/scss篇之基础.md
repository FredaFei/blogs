##### 变量声明
格式

```
    $[变量名]: [value];
    //exp:
    $border-color:red; 
```
##### 变量引用

```
    .box{border:1px solid $border-color;}
```
##### CSS嵌套和父选择器的标识符&

```
    .box{
        width:100px;
        &-active{
           border-color: blue;
        }
        >.row{padding:10px;}
        .text{font-size: 16px;}
    }
```
编译后
```
    .box {
      width: 100px;
    }
    .box-active {
      border-color: blue;
    }
    .box > .row {
      padding: 10px;
    }
    .box .text {
      font-size: 16px;
    }
```

#### 导入SASS文件
格式
```
    @import “[文件]”
    //exp:
    @import “colors”
```
