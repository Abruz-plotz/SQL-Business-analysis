# SQL Project

## Dataset

```sql
select * from customer_churn;
```
#### Output 🥇

###### Using Python :-
![Result](https://raw.githubusercontent.com/Abruz-plotz/SQL-Business-analysis/main/Screenshots%20for%20GithubSQL/First.png)


###### Full output link:-

<div style="margin: 10px 0;">
  <a href="https://github.com/Abruz-plotz/SQL-Business-analysis/blob/main/Customer_analysis_SQL.csv" 
     style="
       display: inline-block;
       background-color: #4CAF50;
       color: white;
       padding: 10px 20px;
       text-decoration: none;
       border-radius: 5px;
       font-weight: bold;
     ">
    📁 Download Full Output (CSV)
  </a>
</div>

### Data Cleaning:

```sql
set sql_safe_updates = 0;

update customer_churn
join (select round(avg(WarehouseToHome),0) as avg_WarehouseToHome from customer_churn)
a on 1=1 set customer_churn.WarehouseToHome = a.avg_WarehouseToHome
where customer_churn.WarehouseToHome is null;

update customer_churn
join (select round(avg(HourSpendOnApp),0) as avg_HourSpendOnApp  from customer_churn)
a on 1=1 set customer_churn.HourSpendOnApp = a.avg_HourSpendOnApp
where customer_churn.HourSpendOnApp is null;
    
update customer_churn
join (select round(avg(OrderAmountHikeFromlastYear),0) as avg_OrderAmountHikeFromlastYear from customer_churn)
a on 1=1 set customer_churn.OrderAmountHikeFromlastYear=a.avg_OrderAmountHikeFromlastYear
where customer_churn.OrderAmountHikeFromlastYear is null;
    
update customer_churn
join (select round(avg(DaySinceLastOrder),0) as avg_DaySinceLastOrder from customer_churn)
a on 1=1 set customer_churn.DaySinceLastOrder=a.avg_DaySinceLastOrder
where customer_churn.DaySinceLastOrder is null;
    
update customer_churn
set Tenure = (select Tenure from
(select Tenure,count(Tenure) as value_occurrence from customer_churn group by
Tenure order by value_occurrence desc limit 1) as mode_Tenure)
where Tenure is null;   
    
update customer_churn
set CouponUsed = (select CouponUsed from
(select CouponUsed,count(CouponUsed) as value_occurrence from customer_churn group by
CouponUsed order by value_occurrence desc limit 1) as mode_CouponUsed)
where CouponUsed is null;   
     
update customer_churn
set OrderCount = (select OrderCount from
(select  OrderCount,count(OrderCount) AS value_occurrence from customer_churn group by
OrderCount order by value_occurrence desc limit 1) as mode_OrderCount)
where OrderCount is null;

Select * from customer_churn where WarehouseToHome > 100;
delete from customer_churn where WarehouseToHome > 100;
```
#### Output
![Result](https://raw.githubusercontent.com/Abruz-plotz/SQL-Business-analysis/main/Screenshots%20for%20GithubSQL/Dist%20100+%20delete.png)


### Dealing with Inconsistencies

```sql   
   update customer_churn 
   set PreferredPaymentMode = case
   when PreferredPaymentMode = 'cc' then 'Credit Card'
   when PreferredPaymentMode = 'cod' then 'Cash on Delivery'
   else PreferredPaymentMode
   end,
   PreferredLoginDevice = case
   when PreferredLoginDevice = 'Phone' then 'Mobile Phone'
   else PreferredLoginDevice
   end,
   PreferedOrderCat = case 
   when PreferedOrderCat = 'Mobile' then 'Mobile Phone'
   else PreferedOrderCat 
   end;
   
Select * from customer_churn;
```
#### Output
![Result](https://raw.githubusercontent.com/Abruz-plotz/SQL-Business-analysis/main/Screenshots%20for%20GithubSQL/After%20Incon%20remove.png)   


### Data Transformation: 
###### Column Renaming and Creating New Columns

```sql 
alter table customer_churn
rename column PreferedOrderCat to PreferredOrderCat,
rename column HourSpendOnApp to HoursSpentOnApp,
add column ComplaintReceived varchar(50),
add column ChurnStatus varchar(50);

update customer_churn 
set ComplaintReceived  = case
when Complain = 1 then 'Yes'
else 'No'
end,
ChurnStatus = case
when churn = 1 then 'Churned'
else 'Active'
end;

-- Column Dropping

alter table customer_churn
drop column churn,
drop column complain;

select * from customer_churn;
```
![Result](https://raw.githubusercontent.com/Abruz-plotz/SQL-Business-analysis/main/Screenshots%20for%20GithubSQL/After%20Incon%20remove.png) 


### Data Exploration and Analysis
  <!-- -- Question D -->

```sql 
select  sum(case when churnStatus = 'Churned' then 1 else 0 end) as Count_of_Churned,
		 sum(case when churnStatus = 'Active' then 1 else 0 end) as Count_of_Active 
from customer_churn;

select avg(Tenure),sum(CashbackAmount) as Total_Cashback,count(CashbackAmount) as Number_of_customers
from customer_churn
where churnStatus = 'Churned';
```
![Result](https://raw.githubusercontent.com/Abruz-plotz/SQL-Business-analysis/main/Screenshots%20for%20GithubSQL/count.png)


```sql 
Select 
cast((Select count(*) from customer_churn
where churnStatus = 'Churned' and ComplaintReceived = 'Yes') as DECIMAL(10, 2)) * 100 / 
(select count(*) from customer_churn
where churnStatus = 'Churned') as Percentage_of_Churned_Customers_complained;
```
**Analysis No.4 :-** City tier with the highest number of churned customers whose preferred order category is "Laptop & Accessory".

```sql
  SELECT CityTier,
    COUNT(CustomerID) as Churned_Customer_count FROM customer_churn
where churnStatus = 'Churned' and PreferredOrderCat='Laptop & Accessory'
  GROUP BY CityTier
  ORDER BY Churned_Customer_count DESC limit 1;
```


**Analysis No.5 :-** The most preferred payment mode among active customers.

```sql  
  select PreferredPaymentMode as PreferredPaymentMode_AmongActive, count(CustomerID) AS No_of_Customers_Used from customer_churn
  where churnStatus = 'Active'
  group by PreferredPaymentMode
  order by No_of_Customers_Used DESC limit 1;
```

**Analysis No.6 :-** Total order amount hike from last year for customers who are single and prefer mobile phones for ordering.

```sql
Select sum(OrderAmountHikeFromlastYear) as Total_Order_Amount_Hike from customer_churn
where MaritalStatus='Single' and PreferredOrderCat='Mobile Phone';
```
![Result](https://raw.githubusercontent.com/Abruz-plotz/SQL-Business-analysis/main/Screenshots%20for%20GithubSQL/Ans_6.png)

##### Analysis No:- 7

```sql  
Select avg(NumberOfDeviceRegistered) from customer_churn
where PreferredPaymentMode = 'UPI';
```  
  Select CityTier,
    COUNT(CustomerID) as Customer_count from customer_churn
     GROUP BY CityTier
     ORDER BY Customer_count DESC limit 1;
     
  Select Gender, 
  sum(CouponUsed) as Utilized_coupon_count from customer_churn
  group by Gender
  ORDER BY Utilized_coupon_count DESC limit 1;
  
  Select PreferredOrderCat,count(CustomerID) as No_of_Customers,
         Sum(HoursSpentOnApp) as Max_hours_Spent from customer_churn
          group by PreferredOrderCat
          order by PreferredOrderCat,No_of_Customers,Max_hours_Spent;
          
	Select Sum(OrderCount) as Total_Order_Count,count(OrderCount) as No_Of_Customers from customer_churn
    where PreferredPaymentMode = 'Credit Card' 
    and SatisfactionScore =(Select max(SatisfactionScore) from customer_churn);
    
    Select avg(SatisfactionScore) as Average_Satisfaction_Score
    from customer_churn where ComplaintReceived = 'Yes';
    
    Select CustomerID,PreferredOrderCat,CouponUsed from  customer_churn
    where CouponUsed > 5; 
  
  Select PreferredOrderCat as Top_3_category,(select avg(CashbackAmount)) as Average_Cashback from customer_churn
  group by PreferredOrderCat
  order by Average_Cashback DESC limit 3;
  
 /* select CustomerID from customer_churn
  group by CustomerID
  having sum(OrderCount) > 500; */
  
  
    
 Select PreferredPaymentMode
        from customer_churn
  group by  PreferredPaymentMode
    having avg(Tenure) > 10
	   and sum(OrderCount) > 500;        
    
Select  CustomerID,WarehouseToHome,
case when WarehouseToHome <= 5 then 'Very Close Distance'
     when WarehouseToHome > 5 and WarehouseToHome <= 10 then 'Close Distance'
     when WarehouseToHome > 10 and WarehouseToHome <= 15 then 'Moderate Distance'
	 when WarehouseToHome > 15 then 'Far Distance'
     end
     as Comment_Distance
     from customer_churn;
     
    Select  CustomerID,PreferredOrderCat,OrderAmountHikeFromlastYear,DaySinceLastOrder,CityTier,MaritalStatus,OrderCount
    from customer_churn
    group by  CustomerID
    having MaritalStatus='Married' 
    and CityTier = 1 and OrderCount > (select avg(OrderCount) from customer_churn);-- 2.5533;

Create table customer_returns( ReturnID  INT PRIMARY KEY,CustomerID INT, 
                              ReturnDate date, RefundAmount int);
                              
  Insert into customer_returns(ReturnID,CustomerID,ReturnDate,RefundAmount)
                     values(1001,50022,'2023-01-01',2130),
						   (1002,50316,'2023-01-23',2000),
                           (1003,51099,'2023-02-14',2290),
                           (1004,52321,'2023-03-08',2510),
						   (1005,52928,'2023-03-20',3000),
                           (1006,53749,'2023-04-17',1740),
                           (1007,54206,'2023-04-21',3250),
						   (1008,54838,'2023-04-30',1990);
 
 select * from customer_returns;
 
 Select customer_returns.*,customer_churn.churnStatus,ComplaintReceived
         from customer_churn join customer_returns
               on customer_churn.CustomerID = customer_returns.CustomerID
               where customer_churn.churnStatus = 'Churned' and customer_churn.ComplaintReceived = 'Yes';      
