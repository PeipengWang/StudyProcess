&&是做与运算，&&又叫短路运算符。 因为当第一个表达式的值为false的时候，则不会再计算第二个表达式；而&则不管第一个表达式是否为真都会执行两个表达式。 
下面测试
```
public class testYu {
    public static void main(String[] args) {
        boolean test1 = false;
        if(test1 && test2()){
            System.out.println("结束");
        }
    }
    public static boolean test2(){
        System.out.println("进入此方法,没有短路");
        return true;
    }
}
```
会发现没有输出："进入此方法,没有短路" ，test2()方法发生短路