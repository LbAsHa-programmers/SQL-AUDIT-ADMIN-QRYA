SELECT t.[text]
FROM sys.dm_exec_cached_plans AS p
CROSS APPLY sys.dm_exec_sql_text(p.plan_handle) AS t
WHERE t.[text] LIKE N'%BPCNUM%';


/*####################################################################################*/


SELECT t.[text], s.last_execution_time
FROM sys.dm_exec_cached_plans AS p
INNER JOIN sys.dm_exec_query_stats AS s
   ON p.plan_handle = s.plan_handle
CROSS APPLY sys.dm_exec_sql_text(p.plan_handle) AS t
WHERE t.[text] LIKE N'%%'
ORDER BY s.last_execution_time DESC;



/*####################################################################################*/


select  top(100)
        creation_time,
        last_execution_time,
        execution_count,
        total_worker_time/1000 as CPU,
        convert(money, (total_worker_time))/(execution_count*1000)as [AvgCPUTime],
        qs.total_elapsed_time/1000 as TotDuration,
        convert(money, (qs.total_elapsed_time))/(execution_count*1000)as [AvgDur],
        total_logical_reads as [Reads],
        total_logical_writes as [Writes],
        total_logical_reads+total_logical_writes as [AggIO],
        convert(money, (total_logical_reads+total_logical_writes)/(execution_count + 0.0)) as [AvgIO],
        [sql_handle],
        plan_handle,
        statement_start_offset,
        statement_end_offset,
        plan_generation_num,
        total_physical_reads,
        convert(money, total_physical_reads/(execution_count + 0.0)) as [AvgIOPhysicalReads],
        convert(money, total_logical_reads/(execution_count + 0.0)) as [AvgIOLogicalReads],
        convert(money, total_logical_writes/(execution_count + 0.0)) as [AvgIOLogicalWrites],
        query_hash,
        query_plan_hash,
        total_rows,
        case 
            when sql_handle IS NULL then ' '
            else(substring(st.text,(qs.statement_start_offset+2)/2,(
                case
                    when qs.statement_end_offset =-1 then len(convert(nvarchar(MAX),st.text))*2      
                    else qs.statement_end_offset    
                end - qs.statement_start_offset)/2  ))
        end as query_text,
        db_name(st.dbid) as database_name,
        object_schema_name(st.objectid, st.dbid)+'.'+object_name(st.objectid, st.dbid) as [object_name],
        sp.[query_plan]
from sys.dm_exec_query_stats as qs with(readuncommitted)
cross apply sys.dm_exec_sql_text(qs.[sql_handle]) as st
cross apply sys.dm_exec_query_plan(qs.[plan_handle]) as sp
WHERE st.[text] LIKE '%BPCUSTOMER%'
and CONVERT(smalldatetime, last_execution_time) = CONVERT(smalldatetime, GETDATE())
ORDER BY last_execution_time DESC;