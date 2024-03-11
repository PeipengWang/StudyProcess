# if_else
## 上节课回顾
include<studio.h> -- 包含头
sacnf() -- 输入语句
printf()-- 输出语句
## 为什么要学判断语句
人工智能发展，判断逻辑，判断思维，机器的判断思维是机器智能化的从0到1（补充）
顺序结构，判断语句，循环语句是计算机语言的三种结构，无论是C语言还是任意一种计算机语言，所以说你学好了C语言判断语句，就学会了任何一种语言的判断语句。最后等到一定程度，语言就是一种工具（补充）
在 C 语言中，嵌套 if-else 语句是合法的，这意味着您可以在一个 if 或 else if 语句内使用另一个 if 或 else if 语句。
在C语言中，有三种条件判断结构：if语句、if-else语句和switch语句。
## 接下来来了解什么是判断语句，他的写法是怎么样的
if  -- 如果
else  -- 否则
转化为文字
	如果...就...
	if(条件){条件成立后我们要做的事}
## 转化为流程图
![](_v_images/20230511224253348_23758.png =448x)


##  单分支if语句
```
	如果...就...
	if(条件){条件成立后我们要做的事}
```
## if-else语句

```
	如果...就...否则...
	if(条件){条件成立后我们要做的事}else{条件不成立我们要做的事}

需求:判断用户年龄    如果年龄满18我们就输出  欢迎光临  否则输出  谢绝入内
```
```
#include<stdio.h>
	int main() {
		//需求:判断用户年龄    
		//如果年龄满18我们就输出  欢迎光临  
		//否则输出  谢绝入内
		printf("请输入年龄:");	  //给用户一个提示
		int age;
		scanf_s("%d", &age);	 //获取用户在控制输入的值,并给age赋值
		if (age >= 18) {
			printf("欢迎光临");
		}
		else {
			printf("谢绝入内");
		}
		return 0;
	}

```
## if-esle if语句
```
	如果...就...否则  如果(可能有多个)...就 ...否则...
	 if(条件1){条件1成立后我们要做的事}
	 else if(条件2){条件2成立后我们要做的事}
			.可以有多个
	 else(前面所有条件都不满足我们要做的事)
需求:判断春夏秋冬
```
```
#include<stdio.h>
	int main() {
		//需求:判断春夏秋冬
		// 1 2 3春
		// 4 5 6夏
		// 7 8 9秋
		// 10 11 12冬
		// 要求 : 用户输入的形式
		// 至少三种方式实现
		printf("请输入月份:");
		int month;
		scanf_s("%d",&month);
		if (month <= 3) {
			printf("春天在这里");
		}
		else if (month >= 4 && month <= 6) {
			printf("最爱的夏天来了");
		}
		else if (month >= 7 && month <= 9) {
			printf("落叶秋");
		}
		else if (month >= 10 && month <= 12) {
			printf("冻感超人");
		}
		else {
			printf("月份错误");
		}
		return 0;
	}
```
## if嵌套
需求:如果分数大于60输出及格
	 否则如果小于60输出挂科
	 否则输出 谢天谢地

```
#include<stdio.h>
int main() {
	//需求:如果分数大于60输出及格
	//否则如果小于60输出挂科
	//否则输出 谢天谢地
	int score;
	scanf_s("%d",&score);
	if (score > 60 && score <= 100) {
		printf("及格");
	}
	else if (score < 60 && score >= 0) {
		printf("挂科");
	}
	else {
		if (score > 100 || score < 0) {
			printf("成绩错误");
		}
		else {
			printf("谢天谢地");
		}
	}
	printf("==============================");
	if (score > 60) {
		if (score <= 100) {
			printf("及格");
		}
		else {
			printf("成绩错误");
		}
	}
	else if (score < 60) {
		if (score >= 0) {
			printf("挂科");
		}
		else {
			printf("成绩错误");
		}
	}
	else{
		printf("谢天谢地");
	}
	return 0;
}
```
## switch case
```
switch (值) {//整型 字符 Enum枚举
	case 值:语句; break;
	...
	...
	...
	default:语句; break;
}
需求:判断春夏秋冬
	1 2 3春
	4 5 6夏
	7 8 9秋
	10 11 12冬
```
```#include<stdio.h>
	int main() {
		printf("请输入月份:");
		int month;
		scanf_s("%d",&month);
		switch (month) {//整型 字符 Enum枚举
			case 1:
			case 2:
			case 3:printf("春"); break;
			case 4:
			case 5:
			case 6:printf("夏"); break;
			case 7:
			case 8:
			case 9:printf("秋"); break;
			case 10:
			case 11:
			case 12:printf("冬"); break;
			default:printf("没有当前月"); break;
		}
		return 0;
	}
```



