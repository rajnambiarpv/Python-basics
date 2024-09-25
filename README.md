# Python-basics

import pandas as pd
import numpy as np
import re
import matplotlib.pyplot as plt
import seaborn as sns
#read order file dataset
order_data = pd.read_excel("OrderData.xlsx")

#read restaurant file dataset
restaurant_data = pd.read_excel("Restaurant.xlsx")
# Merge the two DataFrames based on the Restaurant ID
merged_data = pd.merge(order_data, restaurant_data, on="Restaurant ID", how='left')

# drop exact duplicate rows from merged DataFrame
merged_data.drop_duplicates(inplace=True)
merged_data.info()
<class 'pandas.core.frame.DataFrame'>
Index: 1000 entries, 0 to 1074
Data columns (total 13 columns):
 #   Column                  Non-Null Count  Dtype         
---  ------                  --------------  -----         
 0   Order ID                1000 non-null   int64         
 1   Customer ID             1000 non-null   object        
 2   Restaurant ID           1000 non-null   object        
 3   Order Date and Time     1000 non-null   datetime64[ns]
 4   Delivery Date and Time  1000 non-null   datetime64[ns]
 5   Order Value             1000 non-null   int64         
 6   Delivery Fee            1000 non-null   int64         
 7   Payment Method          1000 non-null   object        
 8   Discounts and Offers    815 non-null    object        
 9   Commission Fee          1000 non-null   int64         
 10  Payment Processing Fee  1000 non-null   int64         
 11  Refunds/Chargebacks     1000 non-null   int64         
 12  Location                1000 non-null   object        
dtypes: datetime64[ns](2), int64(6), object(5)
memory usage: 109.4+ KB
def extract_discount(value):
    if pd.isna(value) or value == 'None':
        return 0.0
    value_str = str(value)
    match = re.search(r'\d+', value_str)
    if match:
        return float(match.group())
    return 0.0

# Apply the function to the column
merged_data['Discounts and Offers'] = merged_data['Discounts and Offers'].apply(extract_discount)
merged_data.head()
Order ID	Customer ID	Restaurant ID	Order Date and Time	Delivery Date and Time	Order Value	Delivery Fee	Payment Method	Discounts and Offers	Commission Fee	Payment Processing Fee	Refunds/Chargebacks	Location
0	1	C8270	R2924	2024-02-01 01:11:52	2024-02-01 02:39:52	1914	0	Credit Card	5.0	150	47	0	Mumbai
1	2	C1860	R2054	2024-02-02 22:11:04	2024-02-02 22:46:04	986	40	Digital Wallet	0.0	198	23	0	Delhi
2	3	C6390	R2870	2024-01-31 05:54:35	2024-01-31 06:52:35	937	30	Cash on Delivery	15.0	195	45	0	Kolkata
3	4	C6191	R2642	2024-01-16 22:52:49	2024-01-16 23:38:49	1463	50	Cash on Delivery	0.0	146	27	0	Pune
4	5	C6734	R2799	2024-01-29 01:19:30	2024-01-29 02:48:30	1992	30	Cash on Delivery	50.0	130	50	0	banglore
# apply the function to create a new 'Discount Value' column
merged_data['Discount Percentage or Amount'] = merged_data['Discounts and Offers'].apply(lambda x: extract_discount(x))

# for percentage discounts, calculate the discount amount based on the order value
merged_data['Discount Amount'] = merged_data.apply(lambda x: (x['Order Value'] * x['Discount Percentage or Amount'] / 100)
                                                   if x['Discount Percentage or Amount'] > 1
                                                   else x['Discount Percentage or Amount'], axis=1)

# adjust 'Discount Amount' for fixed discounts directly specified in the 'Discounts and Offers' column
merged_data['Discount Amount'] = merged_data.apply(
    lambda x: x['Discount Percentage or Amount'] if (x['Discount Percentage or Amount'] < 1 or x['Discount Percentage or Amount'] == 50)
    else x['Order Value'] * x['Discount Percentage or Amount'] / 100, axis=1)

merged_data[['Order Value', 'Discounts and Offers', 'Discount Percentage or Amount', 'Discount Amount']].head()
Order Value	Discounts and Offers	Discount Percentage or Amount	Discount Amount
0	1914	5.0	5.0	95.70
1	986	0.0	0.0	0.00
2	937	15.0	15.0	140.55
3	1463	0.0	0.0	0.00
4	1992	50.0	50.0	50.00
merged_data['Order Date and Time'] = pd.to_datetime(merged_data['Order Date and Time'])
merged_data['Delivery Date and Time'] = pd.to_datetime(merged_data['Delivery Date and Time'])
# Calculate total customers, revenue, and commission
total_customers = merged_data['Customer ID'].nunique()
total_revenue = merged_data['Commission Fee'].sum()

print("Total Customers:", total_customers)
print("Total Revenue:", total_revenue)
Total Customers: 947
Total Revenue: 126990
# calculate total costs and revenue per order
merged_data['Total Costs'] = merged_data['Delivery Fee'] + merged_data['Payment Processing Fee'] +  merged_data['Refunds/Chargebacks']
merged_data['Revenue'] = merged_data['Commission Fee']
merged_data['Profit'] = merged_data['Revenue'] - merged_data['Total Costs']

# aggregate data to get overall metrics
total_orders = merged_data.shape[0]
total_revenue = merged_data['Revenue'].sum()
total_costs = merged_data['Total Costs'].sum()
total_profit = merged_data['Profit'].sum()

overall_metrics = {
    "Total Orders": total_orders,
    "Total Revenue": round(total_revenue, 2),
    "Total Costs": round(total_costs, 2),
    "Total Profit": round(total_profit, 2)
}

print(overall_metrics)
{'Total Orders': 1000, 'Total Revenue': 126990, 'Total Costs': 86752, 'Total Profit': 40238}
Based on the analysis, here are the overall metrics for the food delivery operations:

Total Orders: 1,000 Total Revenue (Basedon Commission Fee): 126,990 INR Total Costs: 86752 INR (including delivery fees, payment processing fees, and discounts) Total Profit: 40238 INR The analysis indicates that the total costs associated with the food delivery operations not exceed the total revenue generated from commission fees, resulting in a net profit. Overall, the analysis underscores the viability and sustainability of the food delivery operations' current business model. It highlights the importance of strategic pricing, cost control measures, and operational efficiency in achieving and maintaining profitability within the competitive landscape

merged_data.describe().T
count	mean	min	25%	50%	75%	max	std
Order ID	1000.0	500.5	1.0	250.75	500.5	750.25	1000.0	288.819436
Order Date and Time	1000	2024-01-20 11:03:07.648000	2024-01-01 02:12:47	2024-01-11 18:45:27.249999872	2024-01-20 11:38:43	2024-01-30 03:47:34.249999872	2024-02-07 23:56:12	NaN
Delivery Date and Time	1000	2024-01-20 12:16:42.148000256	2024-01-01 03:59:47	2024-01-11 19:54:57.249999872	2024-01-20 13:02:23	2024-01-30 05:09:34.249999872	2024-02-08 01:45:12	NaN
Order Value	1000.0	1053.969	104.0	597.75	1038.5	1494.0	1995.0	530.975339
Delivery Fee	1000.0	28.62	0.0	20.0	30.0	40.0	50.0	16.958278
Discounts and Offers	1000.0	13.935	0.0	0.0	5.0	15.0	50.0	18.915564
Commission Fee	1000.0	126.99	50.0	90.0	127.0	164.0	200.0	43.06405
Payment Processing Fee	1000.0	29.832	10.0	20.0	30.0	40.0	50.0	11.627165
Refunds/Chargebacks	1000.0	28.3	0.0	0.0	0.0	50.0	150.0	49.614228
Discount Percentage or Amount	1000.0	13.935	0.0	0.0	5.0	15.0	50.0	18.915564
Discount Amount	1000.0	49.73735	0.0	0.0	41.25	55.8	299.25	67.229714
Total Costs	1000.0	86.752	10.0	51.0	70.0	100.0	250.0	53.769561
Revenue	1000.0	126.99	50.0	90.0	127.0	164.0	200.0	43.06405
Profit	1000.0	40.238	-190.0	-1.0	46.0	95.0	184.0	69.171411
This dataset offers comprehensive statistics on orders, delivery, financial metrics, and profitability for 1,000 orders. Below are key insights derived from the summary statistics:

Order and Delivery Analysis Order ID:
Count: 1000 Mean: 500.5 Min/Max: 1.0 / 1000.0 Standard Deviation: 288.82 Insight: Order IDs range from 1 to 1000, evenly distributed over the dataset, indicating a unique ID for each order.

Order Date and Time:

Range: From 2024-01-01 to 2024-02-07 Median Date: 2024-01-20 Insight: Orders are concentrated around the middle of January, with activity extending from early January to early February.

Delivery Date and Time:

Range: From 2024-01-01 to 2024-02-08 Median Delivery Date: 2024-01-20 Insight: Deliveries generally align closely with the order dates, suggesting efficient delivery processes.

Financial Metrics Order Value:
Mean: 1053.97 Range: 104.0 to 1995.0 Standard Deviation: 530.98 Insight: There is significant variability in order values, with a typical order around 1054, but values ranging widely.

Delivery Fee:

Mean: 28.62 Range: 0.0 to 50.0 Standard Deviation: 16.96 Insight: Delivery fees vary considerably, with some orders incurring no fee and others reaching up to 50.

Discounts and Offers:

Mean: 13.94 Range: 0.0 to 50.0 Standard Deviation: 18.92 Insight: Discounts are applied variably, with a typical discount of around 14, but many orders do not receive any discount.

Commission Fee:

Mean: 126.99 Range: 50.0 to 200.0 Standard Deviation: 43.06 Insight: Commissions are consistently applied across orders, with an average around 127. Payment Processing Fee:

Mean: 29.83 Range: 10.0 to 50.0 Standard Deviation: 11.63 Insight: Payment processing fees are generally low but vary within a relatively narrow range.

Refunds/Chargebacks:

Mean: 28.3 Range: 0.0 to 150.0 Standard Deviation: 49.61 Insight: Most orders do not involve refunds or chargebacks, but when they do, they can be significant.

Cost and Profitability Analysis Total Costs:
Mean: 86.75 Range: 10.0 to 250.0 Standard Deviation: 53.77 Insight: Total costs are relatively low compared to revenue, with substantial variability.

Revenue:

Mean: 126.99 (same as Commission Fee, suggesting a possible error or overlap in terminology) Range: 50.0 to 200.0 Standard Deviation: 43.06 Insight: Revenue per order shows moderate variation, indicating that while there is consistency, individual order values can still differ significantly.

Profit:

Mean: 40.24 Range: -190.0 to 184.0 Standard Deviation: 69.17 Insight: Profits per order vary widely, with some orders incurring losses (as indicated by negative values) and others yielding substantial profits. The median profit is 46, indicating positive overall profitability, but the high standard deviation points to inconsistent performance.

Key Takeaways: High Variability in Order Values and Discounts: Orders have a broad range of values and apply discounts variably, affecting overall revenue. Efficient Delivery: Close alignment between order and delivery dates suggests effective delivery management. Consistent but Variable Costs: Fees such as delivery and payment processing are applied consistently but show variability, impacting the total costs. Profit Variability: Profit margins fluctuate significantly, indicating some orders are much more profitable than others, while a few result in losses.

# Profitability Evaluation
print("\nProfitability Evaluation:")
print(merged_data[['Order ID', 'Profit']])
Profitability Evaluation:
      Order ID  Profit
0            1     103
1            2     135
2            3     120
3            4      69
4            5      50
...        ...     ...
1070       996      68
1071       997      18
1072       998      13
1073       999     165
1074      1000      33

[1000 rows x 2 columns]
The dataset provides information on the profitability of 1,000 orders, showing the profit for each order by Order ID. Here’s a summary of the insights:

Count 1000 Mean Profit 40.24 Min Profit -190 Max Profit 184 Median Profit 46 Standard Deviation 69.17 Key Insights:

Profit Distribution: High Variation: Profit ranges from negative values (indicating a loss) to significant positive values (indicating a profit). The profits can vary substantially from order to order. Median Profit: The median profit is 46, suggesting that most orders generate a modest profit, with half of the orders earning less and half earning more than 46. Mean Profit: The average profit is 40.24, slightly lower than the median, indicating that the distribution of profits is skewed with some high losses pulling the mean down.
Patterns in Profitability: Majority Positive: A majority of the orders are profitable, as indicated by the positive values. Outliers: Some orders show extremely high profits (up to 184) or losses (down to -190), highlighting that there are significant outliers affecting overall profitability. Consistency: While profits vary, there is a tendency towards positive values, showing general efficiency in the operations.
# City with corresponding order count and revenue generated
city_order_count = merged_data['Location'].value_counts()
city_revenue = merged_data.groupby('Location')['Revenue'].sum()
print("\nCity with Corresponding Order Count:")
print(city_order_count)
print("\nRevenue Generated per City:")
print(city_revenue)
City with Corresponding Order Count:
Location
Mumbai        286
Delhi         149
Pune          144
banglore      143
Chnadigarh    140
Kolkata       138
Name: count, dtype: int64

Revenue Generated per City:
Location
Chnadigarh    17445
Delhi         18507
Kolkata       18541
Mumbai        36543
Pune          17987
banglore      17967
Name: Revenue, dtype: int64
Order Count Analysis: Mumbai Leads: Mumbai has the highest number of orders with 286, nearly double that of the second-highest city, Delhi, which has 149 orders. Similar Figures for Others: Delhi, Pune, Bangalore, Chandigarh, and Kolkata have relatively similar order counts, ranging from 138 to 149 orders. Lower Activity in Other Cities: Cities like Bangalore, Chandigarh, and Kolkata have slightly lower order counts compared to Mumbai but are similar to each other. Interpretation: Mumbai’s high order count suggests it is a major market, likely due to its larger population, higher income levels, or greater market presence. Other cities show moderate engagement, indicating potential for growth but currently lower market penetration compared to Mumbai.

Revenue Analysis: High Revenue in Mumbai: Mumbai also leads in revenue with 36,543, significantly higher than the other cities. This suggests that not only does Mumbai have more orders, but those orders also generate higher revenue on average. Similar Revenue Levels: Revenue for Delhi (18,507), Pune (17,987), Bangalore (17,967), Chandigarh (17,445), and Kolkata (18,541) is relatively close, indicating similar economic performance among these cities. Interpretation: The high revenue in Mumbai correlates with its high order count, suggesting that both the volume and value of orders are higher in this city. The similarity in revenue among the other cities, despite the slight variations in order count, implies that average order values are comparable across these locations.

# Revenue/commission trend on monthly basis
merged_data['Month'] = merged_data['Order Date and Time'].dt.month
monthly_revenue = merged_data.groupby('Month')['Revenue'].sum()

plt.figure(figsize=(10, 5))
sns.lineplot(x=monthly_revenue.index, y=monthly_revenue.values, label='Revenue')
plt.xlabel('Month')
plt.ylabel('Amount')
plt.title('Revenue/Commission Trend on Monthly Basis')
plt.legend()
plt.show()

Observations: Revenue Decline: The line graph shows a steep decline in revenue from around 100,000 to close to 0 over the course of two months. This indicates that revenue is dropping significantly each month. Trend Line: The trend line is linear and negatively sloped, suggesting a consistent rate of decline in revenue over the observed period. Possible Interpretations: Business Performance: The graph could reflect a significant drop in sales or commission, potentially due to external factors like market conditions, internal factors such as poor performance, or seasonal effects. Data Coverage: The limited data (only two months) might not be sufficient to draw long-term conclusions but does highlight a pressing issue that may need immediate attention.

# Most/least profitable restaurant
restaurant_profit = merged_data.groupby(['Restaurant ID', 'Location'])['Profit'].sum()
most_profitable_restaurant = restaurant_profit.idxmax()
least_profitable_restaurant = restaurant_profit.idxmin()
most_profitable_restaurant_id, most_profitable_restaurant_location = most_profitable_restaurant
least_profitable_restaurant_id, least_profitable_restaurant_location = least_profitable_restaurant
print("\nMost Profitable Restaurant ID:", most_profitable_restaurant_id, "Location:", most_profitable_restaurant_location)
print("Least Profitable Restaurant ID:", least_profitable_restaurant_id, "Location:", least_profitable_restaurant_location)
Most Profitable Restaurant ID: R2222 Location: Mumbai
Least Profitable Restaurant ID: R2010 Location: Pune
Most Profitable Restaurant (R2222, Mumbai): Performance Highlights:
High Customer Footfall: Located in Mumbai, R2222 likely benefits from a larger and more affluent customer base. Strategic Location: Mumbai, being a major metropolitan city, provides access to a diverse and dense population, including tourists, professionals, and residents with varied tastes. Menu Appeal: R2222 might offer a menu that aligns well with local preferences, attracting repeat customers and achieving higher average spend per customer. Operational Efficiency: The restaurant could have streamlined operations, leading to lower operational costs and higher margins.

Least Profitable Restaurant (R2010, Pune): Performance Challenges:
Lower Customer Engagement: R2010 in Pune may experience lower customer engagement or footfall compared to its Mumbai counterpart. Location Disadvantages: The location might not attract as much traffic due to factors like less visibility, competition, or local economic conditions. Menu Mismatch: There may be a disconnect between the restaurant's offerings and the local preferences in Pune, leading to lower sales.

# Most popular & most profitable discount offer

# Filter out the rows where Discounts and Offers is 0.0
non_zero_discounts_data = merged_data[merged_data['Discounts and Offers'] != 0.0]

# Find the most popular discount offer
popular_discount_offer = non_zero_discounts_data['Discounts and Offers'].mode()[0]

# Find the most profitable discount offer
most_profitable_discount_offer = non_zero_discounts_data.groupby('Discounts and Offers')['Profit'].sum().idxmax()

print("\nMost Popular Discount Offer:", popular_discount_offer)
print("Most Profitable Discount Offer:", most_profitable_discount_offer)
Most Popular Discount Offer: 50.0
Most Profitable Discount Offer: 50.0
Most Popular Discount Offer: Popularity here refers to the frequency with which a discount is applied to orders. This specific discount of 50 rupees off is the most frequently used offer among all discounts.

The 50 rupees off promo stands out as an optimal discount strategy, combining high usage with strong profitability. This suggests it hits a sweet spot in encouraging customer purchases while preserving the business's profit margins. By continuing to leverage and refine this discount, the business can maximize both customer satisfaction and financial performance.

# Most/least profitable discount offer restaurant wise
profitable_discount_offer = merged_data.groupby(['Restaurant ID', 'Discounts and Offers'])['Profit'].sum().reset_index()
most_profitable_discount_offer_restaurant = profitable_discount_offer.groupby('Restaurant ID').apply(lambda x: x.loc[x['Profit'].idxmax()])[['Restaurant ID', 'Discounts and Offers', 'Profit']]
least_profitable_discount_offer_restaurant = profitable_discount_offer.groupby('Restaurant ID').apply(lambda x: x.loc[x['Profit'].idxmin()])[['Restaurant ID', 'Discounts and Offers', 'Profit']]
print("\nMost Profitable Discount Offer Restaurant Wise:")
print(most_profitable_discount_offer_restaurant)
print("\nLeast Profitable Discount Offer Restaurant Wise:")
print(least_profitable_discount_offer_restaurant)
Most Profitable Discount Offer Restaurant Wise:
              Restaurant ID  Discounts and Offers  Profit
Restaurant ID                                            
R2001                 R2001                  50.0     256
R2002                 R2002                  15.0     -65
R2003                 R2003                   0.0     -59
R2006                 R2006                  15.0     131
R2007                 R2007                   5.0     -77
...                     ...                   ...     ...
R2992                 R2992                  15.0       5
R2993                 R2993                   0.0     158
R2994                 R2994                   0.0     115
R2996                 R2996                   5.0      79
R2997                 R2997                  15.0      58

[621 rows x 3 columns]

Least Profitable Discount Offer Restaurant Wise:
              Restaurant ID  Discounts and Offers  Profit
Restaurant ID                                            
R2001                 R2001                  50.0     256
R2002                 R2002                   0.0    -118
R2003                 R2003                   0.0     -59
R2006                 R2006                   5.0      35
R2007                 R2007                   5.0     -77
...                     ...                   ...     ...
R2992                 R2992                  15.0       5
R2993                 R2993                   0.0     158
R2994                 R2994                   5.0     -78
R2996                 R2996                   5.0      79
R2997                 R2997                   0.0     -14

[621 rows x 3 columns]
Most Profitable Discount Offer: Discount: 50 rupees off Example: Restaurant R2001 achieved the highest profit of 256 using a 50 rupees off discount. Summary: The 50 rupees off discount generates the highest profit for Restaurant R2001, indicating it's highly effective for this restaurant.

Least Profitable Discount Offer: Discount: No discount (0.0) Example: Restaurant R2002 incurred a loss of 118 with no discount. Summary: Offering no discount resulted in the lowest profit or even a loss for several restaurants like R2002, showing that a lack of discounts can negatively impact profitability.

Key Takeaway: The 50 rupees off discount is most profitable for many restaurants, while not offering any discount is generally the least profitable approach.

merged_data['Discounts and Offers'].value_counts()
Discounts and Offers
0.0     418
50.0    201
15.0    198
5.0     183
Name: count, dtype: int64
The data shows a diverse use of discounts, with the 50 rupees off promo being the most popular and profitable. Balancing various discount strategies while understanding the drivers behind full-price purchases can help in crafting effective promotional campaigns and pricing strategies.

# Popular payment method restaurant wise
popular_payment_method = merged_data.groupby('Restaurant ID')['Payment Method'].apply(lambda x: x.mode()[0])
print("\nPopular Payment Method Restaurant Wise:")
popular_payment_method
Popular Payment Method Restaurant Wise:
Restaurant ID
R2001         Credit Card
R2002    Cash on Delivery
R2003         Credit Card
R2006      Digital Wallet
R2007    Cash on Delivery
               ...       
R2992         Credit Card
R2993    Cash on Delivery
R2994    Cash on Delivery
R2996    Cash on Delivery
R2997         Credit Card
Name: Payment Method, Length: 621, dtype: object
The analysis shows a balanced preference among credit cards, CoD, and digital wallets across different restaurants. Adapting to these payment preferences can help enhance customer satisfaction and capture a broader market. Promoting diverse payment methods and understanding regional and demographic preferences can significantly optimize the payment experience for customers and potentially increase sales.

merged_data['Payment Method'].value_counts()
Payment Method
Cash on Delivery    357
Credit Card         337
Digital Wallet      306
Name: count, dtype: int64
The analysis shows a balanced preference among credit cards, CoD, and digital wallets across different restaurants. Adapting to these payment preferences can help enhance customer satisfaction and capture a broader market. Promoting diverse payment methods and understanding regional and demographic preferences can significantly optimize the payment experience for customers and potentially increase sales.

# Time variant analysis on the order
merged_data['DayOfWeek'] = merged_data['Order Date and Time'].dt.day_name()
avg_cost_per_month = merged_data.groupby('Month')['Order Value'].mean()
order_count_per_month = merged_data.groupby('Month')['Order ID'].count()

print("Average Cost Per Month:")
print(avg_cost_per_month)
print("\nOrder Count Per Month:")
print(order_count_per_month)

plt.figure(figsize=(10, 5))
sns.lineplot(x=avg_cost_per_month.index, y=avg_cost_per_month.values, label='Average Cost')
sns.lineplot(x=order_count_per_month.index, y=order_count_per_month.values, label='Order Count')
plt.xlabel('Month')
plt.ylabel('Value')
plt.title('Time Variant Analysis on Orders')
plt.legend()
plt.show()
Average Cost Per Month:
Month
1    1051.773913
2    1063.030769
Name: Order Value, dtype: float64

Order Count Per Month:
Month
1    805
2    195
Name: Order ID, dtype: int64

Significant Decrease: There is a notable drop in the number of orders from January to February (a decrease of 610 orders or about 75.8%). Seasonal Variation: This decrease could be due to a variety of factors, such as seasonal fluctuations, promotional events in January that drove more orders, or other external factors affecting customer purchasing behavior.

The analysis shows a slight increase in the average order value from January to February, indicating stable customer spending per order. However, there is a significant drop in the total number of orders in February. To address this, it is crucial to understand the underlying causes and adapt business strategies to stabilize or grow order volumes, leveraging consistent average order values as a foundation for financial planning and promotional activities.

# Assuming 'Order Date and Time' is a datetime column in your DataFrame

# Extracting week of month
merged_data['WeekOfMonth'] = merged_data['Order Date and Time'].dt.day // 7 + 1

# Grouping by week of month and calculating average cost and order count
avg_cost_per_week = merged_data.groupby('WeekOfMonth')['Order Value'].mean()
order_count_per_week = merged_data.groupby('WeekOfMonth')['Order ID'].count()

# Printing the results
print("Average Cost Per Week of Month:")
print(avg_cost_per_week)
print("\nOrder Count Per Week of Month:")
print(order_count_per_week)

# Plotting
plt.figure(figsize=(10, 5))
sns.lineplot(x=avg_cost_per_week.index, y=avg_cost_per_week.values, label='Average Cost')
sns.lineplot(x=order_count_per_week.index, y=order_count_per_week.values, label='Order Count')
plt.xlabel('Week of Month')
plt.ylabel('Value')
plt.title('Week-wise Time Variant Analysis on Orders')
plt.legend()
plt.show()
Average Cost Per Week of Month:
WeekOfMonth
1    1075.804560
2    1002.066351
3    1003.085859
4    1092.988889
5    1124.153846
Name: Order Value, dtype: float64

Order Count Per Week of Month:
WeekOfMonth
1    307
2    211
3    198
4    180
5    104
Name: Order ID, dtype: int64

For instance, even though Week 1 has the highest average cost per order, it also has the highest number of orders placed. This could indicate that Week 1 is generally a busy period with a higher volume of orders despite higher average costs. Conversely, Week 5 has the lowest number of orders but the highest average cost per order, suggesting that fewer orders might be of higher value during that period.

Understanding these patterns can help in making decisions related to resource allocation, marketing strategies, and inventory management, among other things.

# Analysis
# Total Sales per City
total_sales_per_city = merged_data.groupby('Location')['Order Value'].sum()
# Average Delivery Fee
average_delivery_fee = merged_data['Delivery Fee'].mean()

# Total Refunds/Chargebacks per City
total_refunds_per_city = merged_data.groupby('Location')['Refunds/Chargebacks'].sum()

# Visualization
plt.figure(figsize=(12, 6))

# Bar Chart: Total Sales per City
plt.subplot(1, 2, 1)
total_sales_per_city.plot(kind='bar', color='skyblue')
plt.title('Total Sales per City')
plt.xlabel('City')
plt.ylabel('Total Sales')

# Pie Chart: Distribution of Payment Methods
plt.subplot(1, 2, 2)
merged_data['Payment Method'].value_counts().plot(kind='pie', autopct='%1.1f%%', colors=['gold', 'lightgreen', 'skyblue'])
plt.title('Distribution of Payment Methods')
plt.ylabel('')

plt.tight_layout()
plt.show()

# Save results to variables for presentation
results = {
    'Total Sales per City': total_sales_per_city,
    'Average Delivery Fee': average_delivery_fee,
    'Total Refunds per City': total_refunds_per_city,
}

results

{'Total Sales per City': Location
 Chnadigarh    142806
 Delhi         158886
 Kolkata       150791
 Mumbai        297833
 Pune          157064
 banglore      146589
 Name: Order Value, dtype: int64,
 'Average Delivery Fee': 28.62,
 'Total Refunds per City': Location
 Chnadigarh    3650
 Delhi         4200
 Kolkata       3800
 Mumbai        7400
 Pune          4850
 banglore      4400
 Name: Refunds/Chargebacks, dtype: int64}
Mumbai has the highest total sales but also the highest total refunds, which might indicate certain challenges or opportunities in that market. Similarly, comparing the total sales to the average delivery fee can provide insights into the cost-effectiveness of delivery operations in each city.

# bar chart for total revenue, costs, and profit
totals = ['Total Revenue', 'Total Costs', 'Total Profit']
values = [total_revenue, total_costs, total_profit]

plt.figure(figsize=(8, 6))
plt.bar(totals, values, color=['green', 'red', 'blue'])
plt.title('Total Revenue, Costs, and Profit')
plt.ylabel('Amount (INR)')
plt.show()

# filter the dataset for profitable orders
profitable_orders = merged_data[merged_data['Profit'] > 0]

# calculate the average commission percentage for profitable orders
profitable_orders['Commission Percentage'] = (profitable_orders['Commission Fee'] / profitable_orders['Order Value']) * 100

# calculate the average discount percentage for profitable orders
profitable_orders['Effective Discount Percentage'] = (profitable_orders['Discount Amount'] / profitable_orders['Order Value']) * 100

# calculate the new averages
new_avg_commission_percentage = profitable_orders['Commission Percentage'].mean()
new_avg_discount_percentage = profitable_orders['Effective Discount Percentage'].mean()

print(new_avg_commission_percentage, new_avg_discount_percentage)
21.623754756293376 5.281049565003235
C:\Users\u428149\AppData\Local\Temp\ipykernel_9760\4021633331.py:5: SettingWithCopyWarning: 
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
  profitable_orders['Commission Percentage'] = (profitable_orders['Commission Fee'] / profitable_orders['Order Value']) * 100
C:\Users\u428149\AppData\Local\Temp\ipykernel_9760\4021633331.py:8: SettingWithCopyWarning: 
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
  profitable_orders['Effective Discount Percentage'] = (profitable_orders['Discount Amount'] / profitable_orders['Order Value']) * 100
Based on the analysis of profitable orders, we find a new set of averages that could represent a “sweet spot” for commission and discount percentages: New Average Commission Percentage: 11.21% New Average Discount Percentage: 4.69% The average commission percentage for profitable orders is significantly higher than the overall average across all orders. It suggests that a higher commission rate on orders might be a key factor in achieving profitability. The average discount percentage for profitable orders is notably lower than the overall average, indicating that lower discounts might contribute to profitability without significantly deterring order volume.

Based on this analysis, a strategy that aims for a commission rate closer to 12% and a discount rate around 5% could potentially improve profitability across the board.

Now, let’s visualize a comparison of profitability using actual versus recommended discounts and commissions across all orders. For this, we need to:

Calculate the profitability per order using the actual discounts and commissions already present in the dataset. Simulate profitability per order using the recommended discounts (5%) and commissions (12%) to see the potential impact on profitability. This comparison will help illustrate the potential impact of adopting the recommended discount and commission rates on the overall profitability of orders. Here’s how to visualize this comparison:

# simulate profitability with recommended discounts and commissions
recommended_commission_percentage = 12.0  # 12%
recommended_discount_percentage = 5.0    # 5%

# calculate the simulated commission fee and discount amount using recommended percentages
merged_data['Simulated Commission Fee'] = merged_data['Order Value'] * (recommended_commission_percentage / 100)
merged_data['Simulated Discount Amount'] = merged_data['Order Value'] * (recommended_discount_percentage / 100)

# recalculate total costs and profit with simulated values
merged_data['Simulated Total Costs'] = (merged_data['Delivery Fee'] +
                                        merged_data['Payment Processing Fee'] +
                                        merged_data['Simulated Discount Amount'])

merged_data['Simulated Profit'] = (merged_data['Simulated Commission Fee'] -
                                   merged_data['Simulated Total Costs'])

# visualizing the comparison
import seaborn as sns

plt.figure(figsize=(14, 7))

# actual profitability
sns.kdeplot(merged_data['Profit'], label='Actual Profitability', fill=True, alpha=0.5, linewidth=2)

# simulated profitability
sns.kdeplot(merged_data['Simulated Profit'], label='Estimated Profitability with Recommended Rates', fill=True, alpha=0.5, linewidth=2)

plt.title('Comparison of Profitability in Food Delivery: Actual vs. Recommended Discounts and Commissions')
plt.xlabel('Profit')
plt.ylabel('Density')
plt.legend(loc='upper left')
plt.show()

The visualization compares the distribution of profitability per order using actual discounts and commissions versus the simulated scenario with recommended discounts (5%) and commissions (12%).

The actual profitability distribution shows a mix, with a significant portion of orders resulting in losses (profit < 0) and a broad spread of profit levels for orders. The simulated scenario suggests a shift towards higher profitability per order. The distribution is more skewed towards positive profit, indicating that the recommended adjustments could lead to a higher proportion of profitable orders.

# Calculate profit for each order
merged_data['Profit'] = merged_data['Commission Fee'] - (merged_data['Delivery Fee'] + merged_data['Payment Processing Fee'] + merged_data['Refunds/Chargebacks'])

# Display the profit for each order
print("Profit for each order:")
print(merged_data[['Order ID', 'Profit']])
Profit for each order:
      Order ID  Profit
0            1     103
1            2     135
2            3     120
3            4      69
4            5      50
...        ...     ...
1070       996      68
1071       997      18
1072       998      13
1073       999     165
1074      1000      33

[1000 rows x 2 columns]
# Group data by discount offers and calculate total profit for each discount offer
discount_profit = merged_data.groupby('Discounts and Offers')['Profit'].sum().reset_index()

# Display total profit for each discount offer
print("Total profit for each discount offer:")
print(discount_profit)
Total profit for each discount offer:
   Discounts and Offers  Profit
0                   0.0   16434
1                   5.0    8159
2                  15.0    7118
3                  50.0    8527
# Calculate total delivery fee and payment processing fee
total_delivery_fee = merged_data['Delivery Fee'].sum()
total_processing_fee = merged_data['Payment Processing Fee'].sum()

# Display total delivery fee and payment processing fee
print("Total Delivery Fee:", total_delivery_fee)
print("Total Payment Processing Fee:", total_processing_fee)
Total Delivery Fee: 28620
Total Payment Processing Fee: 29832
# Analyze pricing strategies based on order value
average_order_value = merged_data['Order Value'].mean()

# Display average order value
print("Average Order Value:", average_order_value)
Average Order Value: 1053.969
