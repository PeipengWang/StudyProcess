# TableTools
按键：https://datatables.net/extensions/tabletools/button_options#Ajax-button-options
https://datatables.net/reference/button/#tabletools
## 基本

1、tableTools
要通过 DataTables 初始化对象自定义 TableTools 选项，您可以使用此参数。此处可以使用下面“初始化参数”部分中给出的任何选项。
```
$(document).ready( function () {
    $('#example').dataTable( {
        "dom": 'T<"clear">lfrtip',
        "tableTools": {
            "sSwfPath": "/swf/copy_csv_xls_pdf.swf"
        }
    } );
} );
```
2、dom
DataTables 中的参数dom用于定位 DataTables 添加到 DOM 中的表的各种控件（过滤、分页等）。当与 CSS 结合时，这将直接影响显示中元素的位置。可以通过将字符“T”插入到 中来包含 TableTools dom。这就是初始化 TableTools 所需的全部内容 - 一切都是选项和自定义。
```
$(document).ready( function () {
    $('#example').dataTable( {
        "dom": 'T<"clear">lfrtip'
    } );
} );
```
有用到

## 按钮
按钮可以通过配置对象使用buttons，并dom用于指定按钮的放置位置：
## 回调函数
1、fnPreRowSelect
鼠标按下，但 TableTools 尚未选择该行，如果需要，它可用于取消选择
输入参数：
event : 选择该行时发生的点击事件
array - node : 要选择的 TR 元素
```
    $('#example').dataTable( {
        "dom": 'T<"clear">lfrtip',
        "tableTools": {
            "fnPreRowSelect": function ( e, nodes ) {
                if ( e.currentTarget.className.indexOf('no_select') != -1 ) {
                    return false;
                }
                return true;
            }
        }
    } );
```
2、fnRowDeselected
显示详细资料，取消选择行时调用的回调函数（即与 fnRowSelected 相反）。
3、fnRowSelected
选择行时发生的回调函数。这允许在选择任何行（例如选定的行计数器）时发生特定操作。
4、sRowSelect 
是否支持选中多行
TableTools 提供了允许最终用户选择表中的行（通过单击它们）所需的一切。默认情况下禁用此功能，但可以通过将此属性设置为“单”（一次只允许选择表中的一行）或“多”（允许选择任意数量的行）来轻松启用此功能。已选择）。
5、sRowSelector
默认情况下，当行选择处于活动状态时，TableTools 将在单击表中的任何元素时选择一行。将此选择限制为某些列，甚至行中的某些元素可能很有用，这可以通过此选项来完成。它是一个简单的 jQuery 选择器字符串，用于选择将执行选择的元素。
6、sSelectedClass
设置用于指示最终用户已选择行的类。
7、sSwfPath
定义 TableTools 用于复制到剪贴板和本地保存文件操作的 SWF 路径。如果您在这些操作中遇到问题，但在打印方面没有问题，则很可能是问题所在。
8、fnCellRender
用于修改通过fnGetTableData API方法（用于导出数据）从表中读取的数据。这允许在导出数据之前对数据进行预处理 - 例如剥离 HTML 的某些部分或后缀其他数据。
9、fnClick
单击按钮时的回调函数。example1
10、fnComplete
当按钮需要执行的任何操作完成时，此回调函数将被激活。这对于基于 Flash 的按钮特别有用，因为它会在 Flash 执行任何操作（例如保存文件）后激活。
11、fnInit
当按钮被创建并由 TableTools 代码初始化时调用的回调函数。这允许在将按钮插入 DOM 之前对其进行修改（例如添加一个类）。
12、fnMouseout
鼠标离开按钮时的回调函数。
13、fnMouseover
当鼠标悬停在按钮上时的回调函数。
14、fnSelect
在表中选择一行时调用的函数。这允许当表行选择更改状态时在按钮上执行操作（例如添加和删除类）
15、oSelectorOpts
控制源 DataTable 中要使用的数据。DataTables 能够通过其$ API 方法选择要使用的行以及它们的使用顺序。此选项也为 TableTools 导出按钮提供了该功能，因此您只能导出当前数据（例如）。
16、sAction
该操作告诉 TableTools 应将新创建的按钮视为哪种类型的按钮 - 可以使用该按钮执行的操作将取决于此设置。它可以是以下之一：“flash_save”、“flash_copy”、“flash_pdf”、“print”、“text”或“collection”。
17、sButtonClass
设置按钮处于非鼠标悬停状态时的类别。
18、sButtonText
将按钮的文本设置为自定义字符串。
19、sExtends
使用预定义的按钮并根据需要扩展它们通常很有用。此属性应与预定义按钮之一相匹配才能准确提供该功能。
### Ajax请求
1、bFooter
是否在导出的数据中包含页脚。
2、bHeader
在导出的数据中包含或不包含标题。
3、fnAjaxComplete
这是 Ajax 加载完成时调用的方法。它是在 Ajax 按钮的 fnClick 函数中定义的。
4、mColumns
此参数定义应将哪些列用作导出的数据源。该参数可以是值为“all”、“visible”或“hidden”的字符串，也可以是包含要导出的列索引的整数数组。另外，从 TableTools 2.2.3开始，它还可以是一个返回要包含在输出中的列索引数组的函数。该函数采用单个参数，即 DataTables 设置对象，可用于获取 API 实例。
5、sAjaxUrl
Ajax 按钮的默认“fnClick”函数使用此参数来定义 Ajax 请求应发送到的位置。
6、sFieldBoundary
设置 CSV 字段的边界字符。通常您会希望这是一个单引号，但更改它可能会很有用。
7、sFieldSeeperator
定义用作 CSV 字段的字段分隔符的字符。通常这是一个逗号
8、aButtons  下拉框
```
"aButtons": [
            
                {
                    "sExtends":    "collection",
                    "sButtonText": "Save",
                    "aButtons":    [ "csv", "xls", "pdf" ]
                }
            ]
```
### Flash button options
1、bBomInc
定义是否应在文件开头包含字节顺序标记 (BOM)。
2、bFooter
是否在导出的数据中包含页脚。
3、bHeader
导出时包含或不包含表头。
4、bSelectedOnly
当此选项设置为 true 时，从表中收集的数据将仅来自最终用户选择的行（使用 sRowSelect 选项） - 所有其他数据将被丢弃（即在保存/复制中不使用） 。如果未选择任何行，则使用所有数据。
5、mColumns
此参数定义应将哪些列用作导出的数据源。该参数可以是值为“all”、“visible”、“hidden”或“sortable”的字符串，也可以是带有要导出的列索引的整数数组。
6、sCharSet
该参数定义了保存的文件将使用什么字符集。它可以采用“utf8”或“utf16le”的值。通常您会想要使用“utf8”，但“utf16le”对于保存 Excel 读取的文件很有用。
7、sFieldBoundary
设置 CSV 字段的边界字符。通常您会希望这是一个单引号，但更改它可能会很有用。
8、sFieldSeeperator
定义用作 CSV 字段的字段分隔符的字符。通常这是一个逗号（CSV 名称由此而来！）。
9、sFileName
该参数定义要保存的文件的名称。特殊字符“*”将被 HTML 文档的“TITLE”标记（如果存在）替换。否则，您可以使用此参数来定义要使用的文件名。
10、sNewLine
定义用于表示导出数据中的换行符的字符。特殊字符串“auto”可用于自动检测应使用什么换行符（即 Windows 上的 \r\n，其他所有系统上的 \n）。
11、sPdfMessage
自由文本输入，可用于向 PDF 导出添加自定义描述/摘要。
12、sPdfOrientation
设置 PDF 输出的纸张方向。
13、sPdf尺寸
设置 PDF 保存文件中使用的纸张尺寸。
14、sTitle
设置保存文档的标题/文件名。对于 PDF 文件，此标题显示在文档顶部（在 sMessage 文本上方，如果给出）。如果作为空字符串给出，TableTools 将使用 HTML 文档中的 TITLE 标记。
15、sToolTip
定义按钮的“标题”属性，以便浏览器将其显示为标准工具提示显示的一部分（通常是一个黄色的小悬停框）
### Print button options
1、bShowAll
此参数可用于使 TableTools 仅显示当前页面 (false) 或表中的所有记录 (true)。
2、sInfo
当打印显示开始时，TableTools 将向最终用户显示一条信息消息，告诉他们发生了什么以及如何摆脱它。可以使用此参数更改此消息。
3、sMessage
您可以使用此参数让 TableTools 在打印显示的顶部放置信息消息（例如版权消息或标题）。
### Text button options
1、bFooter
是否在导出的数据中包含页脚。
2、bHeader
在导出的数据中包含或不包含标题。
3、mColumns
此参数定义应将哪些列用作导出的数据源。该参数可以是值为“all”、“visible”或“hidden”的字符串，也可以是包含要导出的列索引的整数数组。
## 直接初始化
```
$(document).ready( function () {
    var table = $('#example').dataTable();
    var tableTools = new $.fn.dataTable.TableTools( table, {
        "buttons": [
            "copy",
            "csv",
            "xls",
            "pdf",
            { "type": "print", "buttonText": "Print me!" }
        ]
    } );
    $( tableTools.fnContainer() ).insertAfter('div.info');
} );
```



"sDom": 'T<"clear">lfrtip<"clear spacer">T'   上下都有按键

## 实例接口
https://datatables.net/extensions/tabletools/api
1、fnCalcColRatios
获取表显示中使用的列比例的字符串（以逗号分隔）
2、fnDeselect
取消选择单个行
3、fnGetSelected
获取最终用户选择的 TR 节点的数组。
4、fnGetSelectedData
从 DataTables 中获取所选行的数据源对象/数组（与表中每行的 fnGetSelected 后跟 fnGetData 相同）。
5、fnGetSelectedIndexes
获取DataTable中已选择的行的索引。
6、fnGetTableData
根据参数“mColumns”、“sFieldBoundary”和“sFieldSeperator”从表中获取格式化为字符串的数据。然后可以将其发送到 Flash 以进行文件保存，通过 Ajax 或您想要执行的任何其他操作发送到服务器。
7、fnGetTitle
获取页面标题。
8、fnInfo
向最终用户显示信息元素（如复制到剪贴板和打印视图选项所使用的那样，告诉用户正在发生什么）
9、fnIsSelected
使用 TableTools 检查用户是否已选择给定的 TR 行。
10、fnPrint
以编程方式启用或禁用打印视图
11、fnResizeButtons
重新计算要应用于用于复制/保存操作的 Flash 影片的高度和宽度。当 TableTools 在隐藏元素（例如选项卡）中初始化时（其中高度和宽度最初无法计算），这非常有用。
12、fnResizeRequired
查明是否需要调整按钮大小 (fnResizeButtons)。当 TableTools 在隐藏元素（例如选项卡）中初始化时，可能会发生这种情况。
13、fnSelect
选择单独的行
14、fnSelectAll
选择表中的所有行
```
var oTT = TableTools.fnGetInstance( 'example1' );
oTT.fnSelectAll();
```
15、fnSelectNone
取消选择表中的所有行。
16、fnSetText
设置 Flash 影片剪辑的文本。此方法必须用于设置使用 Flash 按钮复制到剪贴板或文件保存时使用的文本。
17、fnGetInstance
此方法可用于获取任何给定表的 TableTools 主实例。
```
var TableTools1 = TableTools.fnGetInstance( 'example1' ); 
/* 对“example1”表的 TableTools 实例执行一些操作... */ 

```
18、fnGetMasters
获取页面上所有 TableTool 主实例的数组
## 插件
copy_to_div
此按钮会将表内容（以格式化文本形式，而不是 HTML）复制到具有给定 ID 的元素（使用“sDiv”参数）。
```
$.fn.dataTable.TableTools.buttons.copy_to_div = $.extend(
    true,
    {},
    $.fn.dataTable.TableTools.buttonBase,
    {
        "sNewLine":    ">br<",
        "sButtonText": "Copy to element",
        "target":      "",
        "fnClick": function( button, conf ) {
            $(conf.target).html( this.fnGetTableData(conf) );
        }
    }
);
 
 
/* Example usage */
$(document).ready( function () {
    $('#example').dataTable( {
        dom': 'T<"clear">lfrtip',
        tableTools': {
            "aButtons": [ {
                "sExtends":    "copy_to_div",
                "sButtonText": "Copy to div",
                "target":      "#copy",
            } ]
        }
    } );
} );
```
download
该按钮将创建一个表单，该表单在激活按钮时提交到服务器。可以将其配置为使用 GET 或 POST（使用 `sType` 选项），并且请求 URL 由 `sUrl` 选项定义。这非常适合向服务器发出请求以触发下载，例如使用服务器端处理创建 PDF。

如果 DataTables 中启用了 Ajax 数据，则插件将提交与 DataTables 用于其自己的 Ajax 请求的相同参数（例如服务器端处理参数）。提交的数据在 HTTP 变量“json”中可用，并且是数据的 JSON 编码表示形式。如果需要，可以指定“fnData”选项以向数据对象添加其他参数。

请注意，此插件需要 DataTables 1.10 或更高版本。
```
$.fn.dataTable.TableTools.buttons.download = $.extend(
    true,
    {},
    $.fn.dataTable.TableTools.buttonBase,
    {
        "sButtonText": "Download",
        "sUrl":      "",
        "sType":     "POST",
        "fnData":    false,
        "fnClick": function( button, config ) {
            var dt = new $.fn.dataTable.Api( this.s.dt );
            var data = dt.ajax.params() || {};
 
            // Optional static additional parameters
            // data.customParameter = ...;
 
            if ( config.fnData ) {
                config.fnData( data );
            }
 
            var iframe = $('<iframe/>', {
                    id: "RemotingIFrame"
                }).css( {
                    border: 'none',
                    width: 0,
                    height: 0
                } )
                .appendTo( 'body' );
                 
            var contentWindow = iframe[0].contentWindow;
            contentWindow.document.open();
            contentWindow.document.close();
             
            var form = contentWindow.document.createElement( 'form' );
            form.setAttribute( 'method', config.sType );
            form.setAttribute( 'action', config.sUrl );
 
            var input = contentWindow.document.createElement( 'input' );
            input.name = 'json';
            input.value = JSON.stringify( data );
             
            form.appendChild( input );
            contentWindow.document.body.appendChild( form );
            form.submit();
        }
    }
);
 
// Example initialisation
$(document).ready( function () {
    $('#example').dataTable( {
        "sDom": 'T<"clear">lfrtip',
        "oTableTools": {
            "aButtons": [ {
                "sExtends":    "download",
                "sButtonText": "Download XLS",
                "sUrl":        "/generate_xls.php"
            } ]
        }
    } );
} );
```