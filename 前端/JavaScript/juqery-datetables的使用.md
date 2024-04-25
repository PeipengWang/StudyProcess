# jquery-dataTables
## 介绍
jQuery DataTables 是一个功能丰富且灵活的 jQuery 插件，用于创建交互式的数据表格，提供搜索、排序、分页、筛选等功能，以便更好地展示和管理数据。
通过使用 jQuery DataTables，你可以轻松地将数据集转换为易于浏览和交互的表格，而无需手动编写大量的 HTML 和 JavaScript 代码。它具有广泛的配置选项和扩展，使你能够适应不同的数据源和显示需求。
以下是一些 jQuery DataTables 的主要特点：
数据处理： 可以从多种数据源（如 JSON、AJAX、HTML 表格等）获取数据并展示。
分页： 支持自动分页，将大量数据拆分成易于浏览的页面。
排序： 可以对各列进行升序或降序排序，帮助用户快速找到所需信息。
搜索： 提供内置搜索功能，允许用户通过关键字搜索数据。
筛选： 允许用户根据列值进行筛选，以缩小显示范围。
分组： 可以将数据分组显示，方便查看相关数据。
自定义： 提供多种配置选项和扩展，允许你根据需求自定义表格的样式和行为。
插件： 有许多官方和第三方扩展，可以为表格添加额外的功能。
要使用 jQuery DataTables，你需要在你的项目中引入 jQuery 库和 DataTables 插件的相关文件，然后在 HTML 中创建一个表格，并通过 JavaScript 初始化 DataTables。
官方网站：https://datatables.net/。
官方源码地址：https://github.com/DataTables
一般会去官方地址根据需求来下载，如果简单本地使用的话直接引用就行
下载地址：https://datatables.net/download/
可以根据选择自由下载
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/2741c0c363274b1e829d980934668b97.png)

## 入门使用
分三步：
1、新建一个html文件，head引入依赖
```html
  <script src="js/jquery.dataTables.js"></script>
  <link rel="stylesheet" type="text/css" href="css/jquery.dataTables.css"/>
```
2、新建一个表格
```html
<table id="myTable" class="display">
  <thead>
  <tr>
    <th>Column 1</th>
    <th>Column 2</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Row 1 Data 1</td>
    <td>Row 1 Data 2</td>
  </tr>
  <tr>
    <td>Row 2 Data 1</td>
    <td>Row 2 Data 2</td>
  </tr>
  </tbody>
</table>
```
效果如下：

3、用jquery-dataTbales美化
```js
  $(document).ready( function () {
    $('#myTable').DataTable();
  } );
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/c385099e1aa1464b80526e9c531de617.png)

此时效果为
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/510ef2d6abea4343a0ef5994656c26bc.png)

可以清楚的看到能够快速对我们的表格数据进行了美化，不用再重复写css和js代码了。
## 通过获得数据源来生成表格
我们可以根据数据直接生成表格数据
在DataTable中定义了三种数据源
数组 -[]
对象 -{}
实例-new MyClass()
接下来将会展示这三种数据源直接生成表格的形式，在这之前需要简化一下html元素
```html
<table id="myTable" class="display">
  <thead>
  <tr>
    <th>Column 1</th>
    <th>Column 2</th>
    <th>Column 3</th>
    <th>Column 4</th>
    <th>Column 5</th>
    <th>Column 6</th>
  </tr>
  </thead>
  </tbody>
</table>
```
直接删掉数据，定义好标题就可以，这意味着这个元素可以定义六列，当然还有更简便的方法，接下来会提到
利用dataTables填充数据
1、数组
```js
  var data = [
    [
      "Tiger Nixon",
      "System Architect",
      "Edinburgh",
      "5421",
      "2011/04/25",
      "$3,120"
    ],
    [
      "Garrett Winters",
      "Director",
      "Edinburgh",
      "8422",
      "2011/07/25",
      "$5,300"
    ]
  ]
  $('#myTable').DataTable( {
    data: data
  } );
```
此时表格展示如下：

2、对象
对象非常适合直观使用，其方式与数组略有不同。如果您通过API主动处理数据，则对象可以使获取特定数据变得非常容易，因为您只需要使用属性名称，而不需要记住该数据所在的数组索引
对象还可以包含比数据表显示所需的更多信息，这对于数据操作非常有用（例如，包括最终用户不可见的数据库主键）。
在这里，上面的html头部也不需要定义了，只需要一个table的id即可，因为我们会在数据中定义要展示的数据,（虽然可以删除，但是这块的头部也会丢失，这里主要关注数据的展示，但这是需要注意的）
```html
<table id="myTable" class="display">

</table>
```
定义好要展示的数据
```js
  var data= [
    {
      "name":       "Tiger Nixon",
      "position":   "System Architect",
      "salary":     "$3,120",
      "start_date": "2011/04/25",
      "office":     "Edinburgh",
      "extn":       "5421"
    },
    {
      "name":       "Garrett Winters",
      "position":   "Director",
      "salary":     "$5,300",
      "start_date": "2011/07/25",
      "office":     "Edinburgh",
      "extn":       "8422"
    }
  ]
```
选择要展示的数据和类属性
```js
  $('#myTable').DataTable( {
    data: data,
    columns: [
      { data: 'name' },
      { data: 'position' },
      { data: 'salary' },
      { data: 'office' }
    ]
  } );
```
这里上面的数据有6列，但是我么可以通过定义columns的方式仅仅展示其中4列
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/4da871b32bc14c26ad65b5f67492dbdd.png)

3、实例
让 DataTables 显示来自 Javascript 对象实例，首先需要定义一个实例对象
```js
function Employee ( name, position, salary, office ) {
    this.name = name;
    this.position = position;
    this.salary = salary;
    this._office = office;
 
    this.office = function () {
        return this._office;
    }
};
 
```
直接展示即可
```js
$('#myTable').DataTable( {
    data: [
        new Employee( "Tiger Nixon", "System Architect", "$3,120", "Edinburgh" ),
        new Employee( "Garrett Winters", "Director", "$5,300", "Edinburgh" )
    ],
    columns: [
        { data: 'name' },
        { data: 'salary' },
        { data: 'office' },
        { data: 'position' }
    ]
} );
```
这里仅仅介绍基本的数据源展示，还有对于表格的数据渲染，复杂数据的展示详见官方文档
