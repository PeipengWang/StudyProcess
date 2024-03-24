# BootStrap5
## 项目搭建
1、引入依赖
从官网 getbootstrap.com 下载 Bootstrap 5。
或者Bootstrap 5 CDN
```
<!-- 新 Bootstrap5 核心 CSS 文件 -->
<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/5.1.1/css/bootstrap.min.css">
 
<!-- 最新的 Bootstrap5 核心 JavaScript 文件 -->
<script src="https://cdn.staticfile.org/twitter-bootstrap/5.1.1/js/bootstrap.bundle.min.js"></script>
```
注意：
bootstrap.bundle.min.js 包含了 Bootstrap 的 JavaScript 组件以及必需的 Popper.js 库，它是一个捆绑版的 Bootstrap JavaScript 文件。所以，当你使用 bootstrap.bundle.min.js 时，不需要额外引入 bootstrap.min.js 或 Popper.js，因为它们已经包含在内。
2、移动设备优先
为了让 Bootstrap 开发的网站对移动设备友好，确保适当的绘制和触屏缩放，需要在网页的 head 之中添加 viewport meta 标签，如下所示：
<meta name="viewport" content="width=device-width, initial-scale=1">
width=device-width 表示宽度是设备屏幕的宽度。
initial-scale=1 表示初始的缩放比例。

这个可以自适应设备屏幕，防止屏幕乱抖动

## 容器类
Bootstrap 5 需要一个容器元素来包裹网站的内容。
我们可以使用以下两个容器类：
.container 类用于固定宽度并支持响应式布局的容器。
.container-fluid 类用于 100% 宽度，占据全部视口（viewport）的容器。
注意：超级大屏幕 (≥1400px) 是 Bootstrap 5 新增加的， Bootstrap 4 最大的是特大屏幕 (≥1200px)。
1、容器内边距
默认情况下，容器都有填充左右内边距，顶部和底部没有填充内边距。 Bootstrap 提供了一些间距类用于填充边距。 比如 .pt-5 就是用于填充顶部内边距
```
<div class="container pt-5"></div>
```
## Bootstrap5 网格系统
Bootstrap 提供了一套响应式、移动设备优先的流式网格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多 12 列。
我们也可以根据自己的需要，定义列数
**请确保每一行中列的总和等于或小于 12。**
Bootstrap 5 网格系统有以下 6 个类:
.col- 针对所有设备。
.col-sm- 平板 - 屏幕宽度等于或大于 576px。
.col-md- 桌面显示器 - 屏幕宽度等于或大于 768px。
.col-lg- 大桌面显示器 - 屏幕宽度等于或大于 992px。
.col-xl- 特大桌面显示器 - 屏幕宽度等于或大于 1200px。
.col-xxl- 超大桌面显示器 - 屏幕宽度等于或大于 1400px。

1、实例
```
<!-- 第一个例子：控制列的宽度及在不同的设备上如何显示 -->
<div class="row">
  <div class="col-*-*"></div>
</div>
<div class="row">
  <div class="col-*-*"></div>
  <div class="col-*-*"></div>
  <div class="col-*-*"></div>
</div>
 
<!-- 第二个例子：或让 Bootstrap 者自动处理布局 -->
<div class="row">
  <div class="col"></div>
  <div class="col"></div>
  <div class="col"></div>
</div>
```
第一个例子：创建一行(<div class="row">)。然后， 添加需要的列( .col-*-* 类中设置)。 第一个星号 (*) 表示响应的设备: sm, md, lg 或 xl, 第二个星号 (*) 表示一个数字, 同一行的数字相加为 12。

第二个例子: 不在每个 col 上添加数字，让 bootstrap 自动处理布局，同一行的每个列宽度相等： 两个 "col" ，每个就为 50% 的宽度。三个 "col"每个就为 33.33% 的宽度，四个 "col"每个就为 25% 的宽度，以此类推。同样，你可以使用 .col-sm|md|lg|xl 来设置列的响应规则。
2、嵌套列
```
<div class="row">
  <div class="col-8">
    .col-8
    <div class="row">
      <div class="col-6">.col-6</div>
      <div class="col-6">.col-6</div>
    </div>
  </div>
  <div class="col-4">.col-4</div>
</div>
```
3、偏移列
偏移列通过 offset-*-* 类来设置。第一个星号( * )可以是 sm、md、lg、xl，表示屏幕设备类型，第二个星号( * )可以是 1 到 11 的数字。
为了在大屏幕显示器上使用偏移，请使用 .offset-md-* 类。这些类会把一个列的左外边距（margin）增加 * 列，其中 * 范围是从 1 到 11。
例如：.offset-md-4 是把.col-md-4 往右移了四列格。
## 表格
Bootstrap5 通过 .table 类来设置基础表格的样式，实例如下:
```
<table class="table">
    <thead>
      <tr>
        <th>Firstname</th>
        <th>Lastname</th>
        <th>Email</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>John</td>
        <td>Doe</td>
        <td>john@example.com</td>
      </tr>
      <tr>
        <td>Mary</td>
        <td>Moe</td>
        <td>mary@example.com</td>
      </tr>
      <tr>
        <td>July</td>
        <td>Dooley</td>
        <td>july@example.com</td>
      </tr>
    </tbody>
</table>
```
1、条纹表格
通过添加 .table-striped 类，您将在 <tbody> 内的行上看到条纹
```
<table class="table table-striped">
```
2、带边框表格
.table-bordered 类可以为表格添加边框
3、鼠标悬停状态表格
.table-hover 类可以为表格的每一行添加鼠标悬停效果（灰色背景）
复制数据时用
4、黑色背景表格
.table-dark 类可以为表格添加黑色背景
5、响应式表格
.table-responsive 类用于创建响应式表格：在屏幕宽度小于 992px 时会创建水平滚动条，如果可视区域宽度大于 992px 则显示不同效果（没有滚动条）
## Bootstrap5 信息提示框
提示框可以使用 .alert 类, 后面加上 .alert-success, .alert-info, .alert-warning, .alert-danger, .alert-primary, .alert-secondary, .alert-light 或 .alert-dark 类来实现:
<div class="alert alert-success">
  <strong>成功!</strong> 指定操作成功提示信息。
</div>
<div class="alert alert-success">
  <strong>成功!</strong> 你应该认真阅读 <a href="#" class="alert-link">这条信息</a>。
</div>
<div class="alert alert-success alert-dismissible">
  <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
  <strong>成功!</strong> 指定操作成功提示信息。
</div>
## 按钮
```
<button type="button" class="btn">基本按钮</button>
<button type="button" class="btn btn-primary">主要按钮</button>
<button type="button" class="btn btn-secondary">次要按钮</button>
<button type="button" class="btn btn-success">成功</button>
<button type="button" class="btn btn-info">信息</button>
<button type="button" class="btn btn-warning">警告</button>
<button type="button" class="btn btn-danger">危险</button>
<button type="button" class="btn btn-dark">黑色</button>
<button type="button" class="btn btn-light">浅色</button>
<button type="button" class="btn btn-link">链接</button>
<input type="button" class="btn btn-info" value="输入框按钮">
<input type="submit" class="btn btn-info" value="提交按钮">
<input type="reset" class="btn btn-info" value="重置按钮">
```
1、不同大小的按钮
Bootstrap 5 可以设置按钮的大小，使用 .btn-lg 类设置大按钮，使用 .btn-sm 类设置小按钮
2、块级按钮
通过添加 .btn-block 类可以设置块级按钮，.d-grid 类设置在父级元素中
3、激活和禁用的按钮
按钮可设置为激活或者禁止点击的状态。
.active 类可以设置按钮是可用的， disabled 属性可以设置按钮是不可点击的。 注意 <a> 元素不支持 disabled 属性，你可以通过添加 .disabled 类来禁止链接的点击。
4、加载按钮
我们也可以设置一个正在加载的按钮。
5、Bootstrap5 按钮组
```
<div class="btn-group">
  <button type="button" class="btn btn-primary">Apple</button>
  <button type="button" class="btn btn-primary">Samsung</button>
  <button type="button" class="btn btn-primary">Sony</button>
</div>
```
6、下拉按钮选择
```
<button type="button" class="btn btn-primary dropdown-toggle" data-bs-toggle="dropdown">Sony</button>
        <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Tablet</a></li>
            <li><a class="dropdown-item" href="#">Smartphone</a></li>
        </ul>
```
## Bootstrap5 徽章（Badges）
实际是给文字加一个柔和的背景
```
<h1>测试标题 <span class="badge bg-secondary">New</span></h1>
  <span class="badge bg-primary">主要</span>
  <span class="badge bg-secondary">次要</span>
  <span class="badge bg-success">成功</span>
  <span class="badge bg-danger">危险</span>
  <span class="badge bg-warning">警告</span>
  <span class="badge bg-info">信息</span>
  <span class="badge bg-light">浅色</span>
  <span class="badge bg-dark">深色</span>
```
药丸形状徽章
使用 .rounded-pill 类来设置药丸形状徽章
```
<span class="badge rounded-pill bg-default">默认</span>
<span class="badge rounded-pill bg-primary">主要</span>
<span class="badge rounded-pill bg-success">成功</span>
<span class="badge rounded-pill bg-info">信息</span>
<span class="badge rounded-pill bg-warning">警告</span>
<span class="badge rounded-pill bg-danger">危险</span>
```
## 进度条
```
<div class="progress">
  <div class="progress-bar" style="width:70%"></div>
</div>
```
1、混合彩色进度条
```
<div class="progress">
  <div class="progress-bar bg-success" style="width:40%">
    Free Space
  </div>
  <div class="progress-bar bg-warning" style="width:10%">
    Warning
  </div>
  <div class="progress-bar bg-danger" style="width:20%">
    Danger
  </div>
</div>
```
2、动画进度条
使用 .progress-bar-animated 类可以为进度条添加动画

可以将1和2结合在一起组成一个混合彩色动画进度条
## 加载效果
Bootstrap5 加载效果
要创建加载中效果可以使用 .spinner-border 类:
## 卡片
我们可以通过 Bootstrap5 的 .card 与 .card-body 类来创建一个简单的卡片，卡片可以包含头部、内容、底部以及各种颜色设置
```
<div class="card">
  <div class="card-body">简单的卡片</div>
</div>
```
## 菜单
不同颜色导航栏
可以使用以下类来创建不同颜色导航栏：.bg-primary, .bg-success, .bg-info, .bg-warning, .bg-danger, .bg-secondary, .bg-dark 和 .bg-light)。
提示: 对于暗色背景 .navbar-dark 需要设置文本颜色为浅色的，对于浅色背景 .navbar-light 需要设置文本颜色为深色的。

### head导航
```
<ul class="nav">
  <li class="nav-item">
    <a class="nav-link" href="#">Link</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" href="#">Link</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" href="#">Link</a>
  </li>
  <li class="nav-item">
    <a class="nav-link disabled" href="#">Disabled</a>
  </li>
</ul>
```
.nav-pills 类可以将导航项设置成胶囊形状。
active时会展示位button类型
```
  <ul class="nav nav-pills">
        <li class="nav-item ">
          <a class="nav-link active"  href="#">Link</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Link</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="#">Link</a>
        </li>
        <li class="nav-item">
          <a class="nav-link disabled" href="#">Disabled</a>
        </li>
      </ul>
```

### 下拉导航
```
<li class="nav-item dropdown">
    <a class="nav-link dropdown-toggle" data-bs-toggle="dropdown" href="#">Dropdown</a>
    <div class="dropdown-menu">
      <a class="dropdown-item" href="#">Link 1</a>
      <a class="dropdown-item" href="#">Link 2</a>
      <a class="dropdown-item" href="#">Link 3</a>
    </div>
  </li>
```
### 动态胶囊选项
这个好用
```
<ul class="nav nav-pills">
  <li class="nav-item">
    <a class="nav-link active" data-bs-toggle="pill" href="#home">Home</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" data-bs-toggle="pill" href="#menu1">Menu 1</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" data-bs-toggle="pill" href="#menu2">Menu 2</a>
  </li>
</ul>
 
<!-- Tab panes -->
<div class="tab-content">
  <div class="tab-pane active container" id="home">...</div>
  <div class="tab-pane container" id="menu1">...</div>
  <div class="tab-pane container" id="menu2">...</div>
</div>
```
## Bootstrap5 轮播
```
<!-- 轮播 -->
<div id="demo" class="carousel slide" data-bs-ride="carousel">
 
   <!-- 指示符 -->
  <div class="carousel-indicators">
    <button type="button" data-bs-target="#demo" data-bs-slide-to="0" class="active"></button>
    <button type="button" data-bs-target="#demo" data-bs-slide-to="1"></button>
    <button type="button" data-bs-target="#demo" data-bs-slide-to="2"></button>
  </div>
  
  <!-- 轮播图片 -->
  <div class="carousel-inner">
    <div class="carousel-item active">
      <img decoding="async" src="https://static.runoob.com/images/mix/img_fjords_wide.jpg" class="d-block" style="width:100%">
    </div>
    <div class="carousel-item">
      <img decoding="async" src="https://static.runoob.com/images/mix/img_nature_wide.jpg" class="d-block" style="width:100%">
    </div>
    <div class="carousel-item">
      <img decoding="async" src="https://static.runoob.com/images/mix/img_mountains_wide.jpg" class="d-block" style="width:100%">
    </div>
  </div>
  
  <!-- 左右切换按钮 -->
  <button class="carousel-control-prev" type="button" data-bs-target="#demo" data-bs-slide="prev">
    <span class="carousel-control-prev-icon"></span>
  </button>
  <button class="carousel-control-next" type="button" data-bs-target="#demo" data-bs-slide="next">
    <span class="carousel-control-next-icon"></span>
  </button>
</div>
```
### 垂直导航（侧边栏导航）
```
<div class="container mt-3">
  <h2>垂直导航</h2>
  <p>.flex-column 类用于创建垂直导航：</p>
  <ul class="nav flex-column">
    <li class="nav-item">
      <a class="nav-link" href="#">Link</a>
    </li>
    <li class="nav-item">
      <a class="nav-link" href="#">Link</a>
    </li>
    <li class="nav-item">
      <a class="nav-link" href="#">Link</a>
    </li>
    <li class="nav-item">
      <a class="nav-link disabled" href="#">Disabled</a>
    </li>
  </ul>
</div>
```
1、创建滑动导航
我们可以通过 JavaScript 来设置是否在 .offcanvas 类后面添加 .show 类，从而控制侧边栏的显示与隐藏：

.offcanvas 隐藏内容 (默认)
.offcanvas.show 显示内容
2、侧边栏的方向
可以通过以下四个类来控制侧边栏的方向：
.offcanvas-start 显示在左侧，如上实例。
.offcanvas-end 显示在右侧
.offcanvas-top 显示在顶部
.offcanvas-bottom 显示在底部
3、设置背景及背景是否可滚动
我们可以在弹出侧边栏的时候设置 <body> 元素是否可以滚动，也可以设置是否显示一个背景画布。 使用 data-bs-scroll 属性来设置 <body> 元素是否可滚动，data-bs-backdrop 来切换是否显示背景画布。


## 模态框
模态框（Modal）是覆盖在父窗体上的子窗体。通常，目的是显示来自一个单独的源的内容，可以在不离开父窗体的情况下有一些互动。子窗体可提供信息交互等。
```
<button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#myModal">
  打开模态框
</button>
 
 
<!-- 模态框 -->
<div class="modal" id="myModal">
  <div class="modal-dialog">
    <div class="modal-content">
 
      <!-- 模态框头部 -->
      <div class="modal-header">
        <h4 class="modal-title">模态框标题</h4>
        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
      </div>
 
      <!-- 模态框内容 -->
      <div class="modal-body">
        模态框内容..
      </div>
 
      <!-- 模态框底部 -->
      <div class="modal-footer">
        <button type="button" class="btn btn-danger" data-bs-dismiss="modal">关闭</button>
      </div>
 
    </div>
  </div>
</div>
```
1、添加动画效果
```
<!-- 添加动画效果 -->
<div class="modal fade"></div>
 
<!-- 不使用动画效果 -->
<div class="modal"></div>
```
2、模态框尺寸
我们可以通过添加 .modal-sm 类来创建一个小模态框，.modal-lg 类可以创建一个大模态框。
尺寸类放在 <div>元素的 .modal-dialog 类后 

```
<div class="modal-dialog modal-sm">  小模态框
<div class="modal-dialog modal-lg"> 大模态框


```

