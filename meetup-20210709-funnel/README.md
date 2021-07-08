---
marp: true
theme: uncover
paginate: true
---

<style>
.row {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  width: 100%;
}

.column {
  display: flex;
  flex-direction: column;
  flex-basis: 100%;
  flex: 1;
}
</style>

## Window Funnel Analytics

- [Introduction](#2)
- [Implementaiton](#6)
- [Works to do in datafuse](#11)

---
### Intrduction
![height:500px](assets/funnel1.png)

---

### Intrduction

- 漏斗分析: 漏斗分析是对用户行为在特定转换路径的转化率分析
- 基于用户行为的分析, 窗口分析
- 分组计算量大
- 计算灵活, 路径自由, 支持下钻分析
- 数据分析驱动优化产品设计

---


### Example
![height:500px](assets/funnel2.png)



---
### Features

- 用户行为独立, 个体之间不影响
- 计算单一用户的最大Level深度
- 高效的匹配算法
- 状态合并


---
### Implementaiton

- Group By -> sort to sorted events
- Window Sequence Matching algorithm
- Merge states


---
### Define the UDAF

```
windowFunnel(timestamp)(window,  condition1, condition2, ....)
```
- timestamp: timestamp integer column
- window: window between the first timestamp and last timestamp
- conditions: boolean values


---
### Example

``` sql
SELECT
    level,
    count() AS c
FROM
(
    SELECT
        user_id,
        windowFunnel(6048000000000000)(timestamp, eventID = 1003,
	 eventID = 1009, eventID = 1007, eventID = 1010) AS level
    FROM trend
    WHERE (event_date >= '2019-01-01') AND
    	  (event_date <= '2019-02-02')
    GROUP BY user_id
)
GROUP BY level
ORDER BY level ASC;
```


---
###  Window Sequence Matching algorithm

![w:800px](assets/funnel3.png)




---
###  Performance tunning

- Sharing data by uid
	- distributed_group_by_no_merge
	- disable data shuffle

- Indexes
	- pre filter
	- sort key
	- bitmap
- Compressions


---
### Works to do in Datafuse

- Generic Aggregate function to have generic states
	- Like: `using TimestampEvent = std::pair<T, UInt8>;`

- Improve the group by performance
	- Varint hash method for group by queries
	- Arena memory pool (woring on)


