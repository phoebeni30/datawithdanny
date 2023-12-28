# datawithdanny
Datawithdanny_course_project

## E-WALLET PRODUCT

# INPUT DESCRIPTION
File fact.csv: transactional info.
File dim_merchant.xlsx: dimension table about merchant info.
File dim_store.xlsx: dimension table about store info

# BUSINESS REQUIREMENT
For each user, please show:
1: The first 2 appID user payments and the date that users used those services?
2: The last storeID user payments and the date that users used that service?
3. How many distinct merchantID & channels that users pay for?
4. How much SaleVolume did each users spend in the last 30 days?
5. Find the merchantName that has the highest applied voucher transactions.
6. Find the Province that has the highest sale volume in the last 45 days.


# Solution visualization

      DROP TABLE IF EXISTS temptable_partI;
      with appid_row_number as (
          select *
          from  (select f.userID, f.appID, f.TransactionDate,
                      ROW_NUMBER() over(partition by f.userID order by f.appID, f.TransactionDate) as row_number
              from dbo.fact f) t
          where t.row_number <=2
      )
      ,
      appid_infor as (
          select r.userID,
                  max(case when r.row_number = 1 then r.appID end) as FirstAppID,
                  isnull(max(case when r.row_number = 2 then r.appID end),0) as SecondAppID,
                  max(case when r.row_number = 1 then r.TransactionDate end) as FirstAppIDDate,
                  isnull(max(case when r.row_number = 2 then r.TransactionDate end),0) as SecondAppIDDate
          from appid_row_number r 
          group by r.userID
      )
      ,
      laststoreid_infor as (
          select t.userID,
                  t.storeID as LastStoreID,
                  t.TransactionDate as LastStoreIDDate
          from (
              select f.userID, 
                      f.storeID,
                      f.TransactionDate,
                      ROW_NUMBER() OVER(partition by userID order by TransactionDate desc) as store_row_number
              from dbo.fact f 
          ) t 
          where t.store_row_number = 1
      )
      ,
      count_merchantID_channel_infor as (
          select  f.userID,
                  count(distinct m.merchantID) as Count_merchantID,
                  count(distinct f.Channel) as Count_Channels
          from dbo.fact f 
          inner join dbo.dim_merchant m
          on f.appID = m.appid
          group by f.userID
      )
      ,
      Salesvolume_infor as (
          select trans_table.userID,
                  sum(case when f.TransactionDate >= trans_table.Last30days_date and f.TransactionDate <= trans_table.TransactionDate 
                      then f.SalesAmount else 0
                      end) 
                      as SalesVolume
          from
              (select t.userID,
                      t.TransactionDate,
                      DATEADD(DAY,-30,t.TransactionDate) as Last30days_date
              from (
                  select f.userID, 
                          f.storeID,
                          cast(f.TransactionDate as datetime2) as TransactionDate ,
                          ROW_NUMBER() OVER(partition by userID order by TransactionDate desc) as TransactionDate_row_number
                  from dbo.fact f 
              ) t 
              where t.TransactionDate_row_number = 1)  as trans_table
          inner join dbo.fact f 
          on f.userID = trans_table.userID
          group by trans_table.userID
      )
      ,
      merchantname_yes as (
          select mr.userID,
                  mr.merchantName,
                  mr.count_merchantname
          from 
              (select mt.userID,
                      mt.merchantName,
                      mt.count_merchantname,
                      ROW_NUMBER() OVER(partition by mt.userID order by mt.count_merchantname desc) as merchant_row_number 
              from
                  (select f.userID,
                          m.merchantID,
                          m.merchantName,
                          count(m.merchantID) as count_merchantname
                  from dbo.dim_merchant m
                  inner join (select *
                              from dbo.fact
                              where VoucherStatus = 'Yes' ) f
                  on f.appID = m.appid
                  group by f.userID, m.merchantID, m.merchantName) mt) mr
          where mr.merchant_row_number = 1
      )
      ,
      merchantname_infor as(
          select f.userID,
                  ISNULL(y.merchantName,0) as merchantName
          from (
              select distinct f.userID 
              from dbo.fact f ) f
          left join merchantname_yes y on f.userID = y.userID
      )
      ,
      province_table as (
          select salevolume45days_table.userID,
                  salevolume45days_table.storeID,
                  salevolume45days_table.SalesVolume,
                  s.Province
          from
              (select trans_table.userID,
                  trans_table.storeID,
                  sum(case when f.TransactionDate >= trans_table.Last45days_date and f.TransactionDate <= trans_table.TransactionDate 
                          then f.SalesAmount else 0
                          end) 
                          as SalesVolume
              from
                  (select t.userID,
                          t.storeID,
                          t.TransactionDate,
                          DATEADD(DAY,-45,t.TransactionDate) as Last45days_date
                  from (
                      select f.userID, 
                              f.storeID,
                              cast(f.TransactionDate as datetime2) as TransactionDate ,
                              ROW_NUMBER() OVER(partition by userID order by TransactionDate desc) as TransactionDate_row_number
                      from dbo.fact f 
                  ) t 
                  where t.TransactionDate_row_number = 1)  as trans_table
              inner join dbo.fact f 
              on f.userID = trans_table.userID
              group by trans_table.userID, trans_table.storeID) salevolume45days_table   --calculate sale volumes/ each customer in the last 45 days
          inner join dbo.dim_store s 
          on s.storeID = salevolume45days_table.storeID
      )
      ,
      province_infor as (
          select province_sv.userID,
                  province_sv.storeID,
                  province_sv.SalesVolume,
                  province_sv.Province
          from (
                  select pt.userID,
                          pt.storeID,
                          pt.SalesVolume,
                          pt.Province,
                          ROW_NUMBER() OVER(partition by pt.userID order by pt.SalesVolume desc) as salevolume_row_number
                  from province_table pt 
          ) province_sv
          where province_sv.salevolume_row_number = 1
      )
      select ai.userID,
              ai.FirstAppID,
              ai.FirstAppIDDate,
              ai.SecondAppID,
              ai.SecondAppIDDate,
              si.LastStoreID,
              si.LastStoreIDDate,
              mci.Count_merchantID,
              mci.Count_Channels,
              svi.SalesVolume,
              mni.merchantName,
              pi.Province
      into temptable_partI
      from appid_infor ai 
      inner join laststoreid_infor si on ai.userID = si.userID
      inner join count_merchantID_channel_infor mci on ai.userID = mci.userID
      inner join Salesvolume_infor svi on ai.userID = svi.userID
      inner join merchantname_infor mni on ai.userID = mni.userID
      inner join province_infor pi on ai.userID = pi.userID
      
--Check table
select * from temptable_partI
order by userID asc
