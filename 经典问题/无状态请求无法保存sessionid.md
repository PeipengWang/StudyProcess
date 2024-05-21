项目入库日志操作中，加入了sessionid的字段，但是如果请求使用RESETful无状态请求写日志，导致无法获取sessionid，从而引起入库失败。  

原因分析：resetful接口如果在自己的方法中不加入HttpRequest 或者 HttpSession这类接口，获取的请求是无状态的，这时候，如果调用写日志的方法会导致获取session失败，从而执行失败。  

解决方法有两个：  
方案一：影响接口包括其中以下两个      
进一步加上判空操作，减少对业务的影响。  

方案二：  
判断是否为无状态请求：
ActionContext ctx = ActionContext.getContext();  
if (ctx != null)  
如果此状态为null，则可以确定是无状态的。  
