
# 关于一天只写出一句sql这种事.... 
## 刚开始是这样的:
SELECT
USERNAME,
MSG_DETAIL AS MSGDETAIL,
COUNT(case when LOGLEVEL='warn' then LOGLEVEL end) AS COUNTSQL,
COUNT(case when LOGLEVEL='Error' then LOGLEVEL end) AS COUNTERROR,
COUNT(MSG_DETAIL) AS TOTALITEM,
AVG(SQL_TAKETIME) AS SQLTAKETIME
FROM t_log
where APINAME='fxd_latest_quotation' AND CURDATE() <= CREATETIME
GROUP BY MSG_DETAIL, USERNAME
ORDER BY SQLTAKETIME DESC LIMIT 0, 20

## 需要分页show出总页数,我想一条sql完成,不然每次请求不同接口都要单独写计算总页数的查询,听起来好low有没有? 
## 机智的我第一反应当然是:count出 group by 后的总条数...wuwuwu,老司机开车了~|~~ 

SELECT
USERNAME AS USERNAME,
MSG_DETAIL AS MSGDETAIL,
COUNT(LOGID) AS TOTALITEM,
COUNTSQL AS COUNTSQL,
COUNTERROR AS COUNTERROR,
SQLTAKETIME AS SQLTAKETIME,

FROM
    (SELECT USERNAME,
    MSG_DETAIL,
    LOGID,
    COUNT(case when LOGLEVEL='warn' then LOGLEVEL end) AS COUNTSQL,
    COUNT(case when LOGLEVEL='Error' then LOGLEVEL end) AS COUNTERROR,
    AVG(SQL_TAKETIME) AS SQLTAKETIME
    FROM t_log
    WHERE APINAME='TF_ContinuousContractInfo' AND CURDATE() <= CREATETIME
    GROUP BY MSG_DETAIL,USERNAME
    ) TL
ORDER BY SQLTAKETIME DESC LIMIT 0,20

## 然后就是加上GROUP BY 数据显示ok,but count的是单条数据的记录===1,一上午过去了,我还在end looper MDZZ!!!,but 这就死心了? 怎么可能---(⊙o⊙)…小司机啊小司机/大兵/大兵 
SELECT
SQL_CALC_FOUND_ROWS
FOUND_ROWS() AS TOTALITEM,
USERNAME,
MSG_DETAIL AS MSGDETAIL,
COUNT(case when LOGLEVEL='warn' then LOGLEVEL end) AS COUNTSQL,
COUNT(case when LOGLEVEL='Error' then LOGLEVEL end) AS COUNTERROR,
AVG(SQL_TAKETIME) AS SQLTAKETIME FROM t_log
WHERE APINAME='fxd_daily_quotation' AND CURDATE() <= CREATETIME
GROUP BY MSG_DETAIL,USERNAME
ORDER BY SQLTAKETIME DESC
LIMIT 0, 20

## 万能的Google,给我跳到了MySQL-官网,哈哈哈,最烦看官网文档了...直晕头,不过硬是看了个七七八八.
##
## 只想大喊一声:Best solution~ 然而放到项目组傻了~在MySQL-编辑器上运行的好好的,通过mybatis一跑 
## 都显示成1了,然而回去mysql编辑器一切恢复正常...MD.初步我将它归为灵异事件.... 
## 满怀心事的吃完饭去了...加班回来再跟你大战三百回合....k. 
## 最后得知:在不加SQL_CALC_FOUND_ROWS的情况下FOUND_ROWS()计算的就是limit中的查询到的数目.加上的话是所有符合条件数据的总数,而且带上一次查询结果的缓存的,前一条语句加上SQL_CALC_FOUND_ROWS查询的结果会在之后的任何一条查询语句中出现...真是坑爹(仅限第一次查询)...so~~~~ 
## ΩΩΩΩ把我坑的好辛苦....继续换!! 


## 加班... 
## 会不会最笨的方法是最有效的呢?哈哈哈哈哈哈哈: 
SELECT
    USERNAME,
    MSG_DETAIL AS MSGDETAIL,
    LOGID,
    ##### 然后我给用mybatis的动态sql给它此处加了判断,只有第一次请求执行这个多重嵌套sql 
    (select count(*) from
        (select * from t_log
            where APINAME='V_Interest_Rate' AND CURDATE() <= CREATETIME
            GROUP BY MSG_DETAIL, USERNAME)
    a) AS TOTALITEM,
    ##### 到此为止可以松一口气了,总算是不要每一次请求,再附加个请求,先拿到总页数,再拿数据(leader让我这样做的,然后劳资硬死磕一天,什么蠢方法,不干 了!)
    COUNT(case when LOGLEVEL='warn' then LOGLEVEL end) AS COUNTSQL,
    COUNT(case when LOGLEVEL='Error' then LOGLEVEL end) AS COUNTERROR,
    AVG(SQL_TAKETIME) AS SQLTAKETIME
FROM t_log
WHERE APINAME='V_Interest_Rate' AND CURDATE() <= CREATETIME
GROUP BY MSG_DETAIL,USERNAME
ORDER BY SQLTAKETIME DESC LIMIT 0,20

## 这次绝对不会再说....Best solution...了,也许还有更好的方法..等来日见证哈哈哈 .Now() === 20:00:00.0000 
## 下班.明天双十一..噢,好像跟我没什么关系~~ 

# 还有要记一下,count就count,别理会什么limit,丝毫不受影响的好吗? 
