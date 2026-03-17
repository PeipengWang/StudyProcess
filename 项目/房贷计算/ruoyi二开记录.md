左侧菜单加载流程：

```
登录
   ↓
调用 /getRouters
   ↓
后端 SysMenuController 查询数据库 sys_menu
   ↓
返回菜单树 JSON
   ↓
Vuex permission.js 存储 sidebarRoutes
   ↓
Sidebar.vue 渲染菜单
```

核心数据表：

```
sys_menu
```



1. 基础房贷测算功能

- 支持贷款类型选择：商贷/公积金/组合贷
- 支持还款方式：等额本息/等额本金
- 输入参数：贷款总额、LPR+基点年利率、贷款年限
- 输出汇总数据：还款总额、总利息、首月还款额
- 输出完整还款计划表：期数、还款日、当月利息、当月本金、剩余本金