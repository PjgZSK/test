### 模板简要
#### 定义
* 在编译期运作  
* 基础功能需编译器支持  
* 是为了实现*将类型作为参数* 这一需求产生的  
#### 细节
* 非确定类型和确定类型都可以作为参数  
    `template<typename T, bool b>` 前者为非确定类型参数，后者为确定类型参数  
* 确定类型参数产生特例化
