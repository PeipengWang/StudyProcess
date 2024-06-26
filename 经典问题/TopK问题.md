# TopK问题
## 从N个无序数中寻找Top-k个最小数
### 海量数据
　针对海量数据的top k问题，这里实现了一种时间复杂度为O(Nlogk)的有效算法：初始时一次性从文件中读取k个数据，并建立一个有k个数的最大堆，代表目前选出的最小的k个数。然后从文件中一个一个的读取剩余数据，如果读取的数据比堆顶元素小，则把堆顶元素替换成当前的数，然后从堆顶向下重新进行堆调整；否则不进行任何操作，继续读取下一个数据。直到文件中的所有数据读取完毕，堆中的k个数就是海量数据中最小的k个数（如果是找最大的k个数，则使用最小堆）

　　对于从海量数据(N)中找出TOP K，这种算法仅需一次性将k个数装入内存，其余数据从文件一个一个读即可，不用将海量数据全都一次性读进内存，所以它是针对海量数据TOP K问题最为有效的算法
　　基本思路如下：定义一个小顶堆，后续放入数据跟，最小值做对比，小于则放入，大于则取出，在入堆
　　问题：由于java的PriorityQueue是没有容量限制的，因此需要加一个方法，来检查是否到了这个k容量。
```
  public static void main(String[] args) {
        int k = 2;
        int n = 1000;
        final PriorityQueue<Integer> results = new PriorityQueue<>(k, (integer, t1) -> integer < t1 ? integer : t1);
        //逐步放入数据，只会保留前两个最大的数据
        add(results, 1, k);
        add(results, 2, k);
        add(results, 3, k);
        add(results, 10, k);  
        System.out.println(results.size());
        System.out.println(results.poll());
        System.out.println(results.poll());
    }
    private static void add(PriorityQueue<Integer> priorityQueue, Integer num, int limit){
        if(priorityQueue.size() < limit){
            priorityQueue.add(num);
        }else if(priorityQueue.size() > 0 && priorityQueue.peek() < num){
            priorityQueue.poll();
            priorityQueue.add(num);
        }

    }
```
### 非海量数据

一般采用BFPRT算法查找出第k个小的数

或者直接排序，根据索引找

## 统计最高10条字符串的出现频率
### 有一千万条查询串，不重复的不超过三百万，统计最热门的10条查询串（内存1G. 字符串长 0-255）
不重复的三百万条字符串的所占最大空间为：256x300x104/1024/1024/1024=0.715G，建立一个map一个一个读取一千万条查询串，统计出现次数，最后将存下三百万条不重复的串和对应的出现次数，此时再从map里取10条查询串以出现次数建立一个大小为10的最小堆，再遍历map中剩下的查询串，依次与最小堆堆顶元素的出现次数作比较，若次数大于堆顶元素，则把堆顶元素替换成当前的串，然后从堆顶向下重新进行堆调整；否则不进行任何操作，继续读取下一个查询串，直到文件中的所有数据读取完毕，堆中的10条串就是最热门的10条查询串，时间复杂度 O(Nlog10)
### 给定a、b两个文件，各存放50亿个url，每个url各占64字节，内存限制是4G，让你找出a、b文件共同的url？
方案1：

　　可以估计每个文件的大小约为50x108x64/1000/1000/1000=320G（1G约等于10亿字节），远远大于内存限制的4G。所以不可能将其完全加载到内存中处理，考虑采取分治法。

　　遍历文件a，对每个url求取哈希值再对1000取余clip_image002，然后根据所取得的值（0~999）将url分别存储到1000个小文件（记为clip_image004）中。这样每个小文件的大约为320M。遍历文件b，采取和a相同的哈希算法将url分别存储到1000各小文件（记为clip_image006）。这样处理后，所有可能相同的url都在对应的小文件（clip_image008）中，不对应的小文件不可能有相同的url。然后我们只要求出1000对小文件中相同的url即可。

　　求每对小文件中相同的url时，可以把其中一个小文件的url存储到set中。然后遍历另一个小文件的每个url，看其是否在刚才构建的set中，如果是，那么就是共同的url，存到文件里面，上述过程重复1000遍即可。

### 有10个文件，每个文件1G，每个文件的每一行存放的都是用户的query，每个文件的query都可能重复。要求你按照query的频度排序
方案1：

　　若内存不够存储所有不重复的query，则按顺序读取10个文件，按照hash(query)%10的结果将query写入到另外10个文件（记为clip_image010）中。这样新生成的文件每个的大小大约也1G（假设hash函数是随机的），相同query进入了同一个小文件。

　　找一台内存在2G左右的机器，依次对clip_image010[1]用hash_map(query, query_count)来统计每个query出现的次数。利用快速/堆/归并排序按照出现次数进行排序。将排序好的query和对应的query_cout输出到文件中。这样得到了10个排好序的文件（记为clip_image012）。

　　对clip_image012[1]这10个文件进行归并排序（内排序与外排序相结合）。

方案2：

　　一般query的总量是有限的，只是重复的次数比较多而已，可能对于所有的query，一次性就可以加入到内存了。这样，我们就可以采用trie树/hash_map等直接来统计每个query出现的次数，然后按出现次数做快速/堆/归并排序就可以了。

方案3：

　　与方案1类似，但在做完hash，分成多个文件后，可以交给多个文件来处理，采用分布式的架构来处理（比如MapReduce），最后再进行合并。

### 有一个1G大小的一个文件，里面每一行是一个词，词的大小不超过16字节，内存限制大小是1M。返回频数最高的100个词

　　顺序读文件中，对于每个词x，取clip_image014，然后按照该值存到5000个小文件（记为clip_image016）中，相同的词进入同一个文件。这样每个文件大概是200k左右。如果其中的有的文件超过了1M大小，还可以按照类似的方法继续往下分，知道分解得到的小文件的大小都不超过1M。对每个小文件，采用trie树或hash_map统计每个文件中出现的词以及相应的频率，再用容量为100的最小堆并取出出现频率最大的100个词，并把100词及相应的频率存入文件，这样又得到了5000个文件。最后用容量为100的最小堆，依次遍历每个文件，获得最终频数最高的100个词。

### 海量日志数据，提取出某日访问百度次数最多的那个IP
　　首先是这一天，并且是访问百度的日志中的IP取出来，逐个写入到一个大文件中。注意到IP是32位的，最多有clip_image018个IP。同样可以采用先哈希再取模的方法，比如模1000，把整个大文件映射为1000个小文件，再找出每个小文中出现频率最大的IP（可以采用hash_map进行频率统计，然后再找出频率最大的几个）及相应的频率。然后再在这1000个最大的IP中，找出那个频率最大的IP，即为所求。


### 海量数据分布在100台电脑中，想个办法统计出这批数据的TOP10
　　在每台电脑上求出TOP10，可以采用包含10个元素的堆完成（TOP10小，用最大堆，TOP10大，用最小堆）。比如求TOP10大，我们首先取前10个元素调整成最小堆，如果发现，然后扫描后面的数据，并与堆顶元素比较，如果比堆顶元素大，那么用该元素替换堆顶，然后再调整为最小堆。最后堆中的元素就是TOP10大。
　　






参考：https://www.cnblogs.com/Joezzz/p/10283102.html