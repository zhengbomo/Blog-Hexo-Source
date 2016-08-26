---
title: ppdai-analyze
date: 2016-07-29 09:48:44
updated: 2016-07-29 09:48:44
categories:
tags:
---

用户数2808

## 学历比例
学历      数量    百分比
NULL	    2164   77.0655
大专	     209    7.4430
高中	     150	  5.3419
中专	     111    3.9530
本科	     81	    2.8846
初中及以下	 64	    2.2792
未知	     26	    0.9259
研究生及以上	3	   0.1068

```sql
select wenhua, count(*) count, count(*) / 2808 * 100 as percent from userinfo
group by wenhua
order by percent desc
```


## 借款
研究生及以上	337005
本科	       44194
大专	       38057
高中	       26200
初中及以下    8694
中专	       5578
未知	       1169

```sql
select wenhua, round(avg(total_borrow_amount), 0) amount from userinfo
where `total_borrow_amount` is not null
group by wenhua
order by amount desc
```



## 结婚与借款
已婚	41,697
未婚	7,902
离婚	4,963
其他	600
丧偶	0

```sql
select jiehun, FORMAT(avg(total_borrow_amount), 0) total_borrow from userinfo
where jiehun is not null
group by jiehun
order by total_borrow
```

## 年龄段
年龄段 人数  借款数
31-40	190	  69499
21-30	395	  11049
41-50	49	  2253
11-20	7	    168
51-60	3	    0

```sql
select nnd '年龄段', count(*) as '人数', round(avg(`total_borrow_amount`), 0) '借款数' from
(
select
case
when age>=1 and age<=10 then '1-10'
when age>=11 and age<=20 then '11-20'
when age>=21 and age<=30 then '21-30'
when age>=31 and age<=40 then '31-40'
when age>=41 and age<=50 then '41-50'
when age>=51 and age<=60 then '51-60'
when age>=60 then '60+'
end
as nnd, username, total_borrow_amount from userinfo
where age is not null
) userinfo

group by nnd
```

## 性别
男	34,068
女	18,409

```sql
select gender, FORMAT(avg(total_borrow_amount), 0) total_borrow from userinfo
where gender is not null
group by gender
order by total_borrow
```


## 预期还款
性别  数量  借款平均数
男    31	   183878
女    19	   9568

```sql
select gender, round(avg(total_borrow_amount), 0) total_borrow, count(*) count from userinfo
where gender is not null and yuqi_days is not null and yuqi_days > 0
group by gender
order by total_borrow
```

## 提前还款
性别  数量  借款平均数
男    167 	69004
女    103   5967

```sql
select gender, round(avg(total_borrow_amount), 0) total_borrow, count(*) count from userinfo
where gender is not null and yuqi_days is not null and tiqian_days < 0
group by gender
order by total_borrow
```


## 更新rankcode和rank
```sql
update userinfo u set u.rankcode = (
	select l.rankcode from loans l
	where l.`user_url` = u.`user_url` limit 1
)
update userinfo u set u.rank = (
	select l.`rank` from loans l
	where l.`user_url` = u.`user_url` limit 1
)
```

## rankcode
D	    826
C	    644
AA	  442
AAA	  389
E	    222
B	    208
F	    72
A	    5

```sql
select rankcode, count(*) count from userinfo
where rankcode is not null
group by rankcode
order by count desc
```

等级  数量  平均提前还款天数
C	    644	 -2.073505804590557
D	    826	 -0.5588983077979839
B	    208	 -0.36880341745339906
E	    222	  0
F	    72	  0
AA	  442	  0
A	    5	    0
AAA	  389	  0

```sql
select rankcode, count(*) count, avg(`tiqian_days`) tiqian_days from userinfo
where rankcode is not null
group by rankcode
order by tiqian_days
```
