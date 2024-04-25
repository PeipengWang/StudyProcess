```bash
 public static void main(String[] args) {
        Map<String,String> map = new HashMap();
        map.put("aa","11");
        map.put("bb","22");
        Set<String> set = map.keySet();
        Iterator iter = set.iterator();
        while(iter.hasNext()){
            System.out.println(iter.next());
        }
    }
```
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200611224133730.png)
首先利用keySet方法生成键的集合，然后利用迭代器遍历Set集合，即为HashMap的键值。