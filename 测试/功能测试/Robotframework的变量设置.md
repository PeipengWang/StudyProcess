# RobotFramework设置变量

1.set variable
2.set global variable
3.set suite variable
4.set test variable
5.set local variable
6.create list
7.create Dictionary


Set Global Variable
    使用范围：在所有测试套件中都可以使用该关键字定义的变量。
Set Suite Variable
    使用范围：使用此关键字设置的变量在当前执行的测试套件的范围内随处可用。
Set Test Variable
    使用范围：使用此关键字设置的变量在当前执行的测试用例的范围内随处可用
Set Variable
    使用范围：该关键字主要用于设置标量变量。此外，它可用于将包含列表的标量变量转换为列表变量或多个标量变量。建议在创建新列表时使用创建列表。使用此关键字创建的变量仅在创建它们的范围内可用
