# Optimizing Inventory and Delivery in a Retail Supply Chain
# Dataset: UCI Online Retail Dataset
```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```
# -----------------------------------------
# 📥 Load the Data
# -----------------------------------------
```
file_path = r"D:\Projects\Optimizing Inventory and Delivery in a Retail Supply Chain\Online Retail.xlsx"
df = pd.read_excel(file_path)
```
# -----------------------------------------
# 🧹 Initial Exploration & Cleaning
# -----------------------------------------
```
print(df.head())
print(df.info())
print(df.describe())
print(df.isnull().sum())
```
# Remove missing CustomerIDs and invalid rows
```
df = df[df['CustomerID'].notna()]
df = df[df['Quantity'] > 0]
df = df[df['UnitPrice'] > 0]
```
# -----------------------------------------
# 🧾 Inventory Movement Analysis
# -----------------------------------------
```
df['InvoiceDate'] = pd.to_datetime(df['InvoiceDate'])
df['Date'] = df['InvoiceDate'].dt.date
df['Week'] = df['InvoiceDate'].dt.to_period('W').dt.start_time
```
# Quantity Sold per Day

```daily_sales = df.groupby(['StockCode', 'Date'])['Quantity'].sum().reset_index()```

# Quantity Sold per Week
```
weekly_sales = df.groupby(['StockCode', 'Week'])['Quantity'].sum().reset_index()
```
# Total Quantity Sold by Product
```
product_sales = df.groupby(['StockCode'])['Quantity'].sum().reset_index()
product_sales = product_sales.sort_values(by='Quantity', ascending=False)
```
# Top 10 Moving Items
```
top_movers = product_sales.head(10)
```

# Bottom 10 Moving Items
```
bottom_movers = product_sales.tail(10)
```
# -----------------------------------------
# 📊 Plot Top and Bottom Movers
# -----------------------------------------
```
plt.figure(figsize=(10, 6))
plt.bar(top_movers['StockCode'].astype(str), top_movers['Quantity'], color='green')
plt.title('Top 10 Moving Products')
plt.xlabel('StockCode')
plt.ylabel('Total Quantity Sold')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 6))
plt.bar(bottom_movers['StockCode'].astype(str), bottom_movers['Quantity'], color='red')
plt.title('Bottom 10 Moving Products')
plt.xlabel('StockCode')
plt.ylabel('Total Quantity Sold')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```
# -----------------------------------------
# 🏭 Simulate Warehouses
# -----------------------------------------
```
warehouses = ['W1', 'W2', 'W3']
unique_products = df['StockCode'].unique()
product_to_warehouse = {product: np.random.choice(warehouses) for product in unique_products}
df['Warehouse'] = df['StockCode'].map(product_to_warehouse)
```

# Simulate Delivery Lag (1–5 days)
```
df['DeliveryDays'] = np.random.randint(1, 6, size=len(df))
df['DeliveryDate'] = df['InvoiceDate'] + pd.to_timedelta(df['DeliveryDays'], unit='D')
```
# -----------------------------------------
# 📈 Key Metrics Calculation
# -----------------------------------------
```
df['Revenue'] = df['Quantity'] * df['UnitPrice']
```

# Product-Level Revenue
```
product_sales = df.groupby('StockCode')['Revenue'].sum().sort_values(ascending=False).reset_index()
```

# Country-wise Demand
```country_demand = df.groupby('Country')['Quantity'].sum().sort_values(ascending=False).reset_index()```

# Returns Analysis
```
returns = df[df['Quantity'] < 0]
returns_by_product = returns.groupby('StockCode')['Quantity'].sum().reset_index()
```
# -----------------------------------------
# 🧮 Warehouse Performance Metrics
# -----------------------------------------
```
warehouse_revenue = df.groupby('Warehouse')['Revenue'].sum().reset_index()
warehouse_delivery_time = df.groupby('Warehouse')['DeliveryDays'].mean().reset_index()
```
# -----------------------------------------
# 📊 Revenue by Warehouse
# -----------------------------------------
```
plt.figure(figsize=(8, 5))
plt.bar(warehouse_revenue['Warehouse'], warehouse_revenue['Revenue'], color='skyblue')
plt.title('Total Revenue by Warehouse')
plt.xlabel('Warehouse')
plt.ylabel('Revenue')
plt.tight_layout()
plt.show()
```
# -----------------------------------------
# 🔝 Top 5 Products per Warehouse
# -----------------------------------------
```
top_products = df.groupby(['Warehouse', 'StockCode'])['Revenue'].sum().reset_index()
top5_per_warehouse = top_products.groupby('Warehouse').apply(lambda x: x.nlargest(5, 'Revenue')).reset_index(drop=True)

for warehouse in top5_per_warehouse['Warehouse'].unique():
    data = top5_per_warehouse[top5_per_warehouse['Warehouse'] == warehouse]
    plt.figure(figsize=(8, 5))
    plt.bar(data['StockCode'].astype(str), data['Revenue'], color='orange')
    plt.title(f'Top 5 Products in {warehouse}')
    plt.xlabel('StockCode')
    plt.ylabel('Revenue')
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()
```
# -----------------------------------------
# 🚚 Delivery Speed Comparison
# -----------------------------------------
```
plt.figure(figsize=(8, 5))
plt.bar(warehouse_delivery_time['Warehouse'], warehouse_delivery_time['DeliveryDays'], color='green')
plt.title('Average Delivery Time by Warehouse')
plt.xlabel('Warehouse')
plt.ylabel('Average Delivery Days')
plt.tight_layout()
plt.show()
```
# -----------------------------------------
# 💾 Export Results to Excel
# -----------------------------------------
```
with pd.ExcelWriter('Retail_SupplyChain_Report.xlsx') as writer:
    warehouse_revenue.to_excel(writer, sheet_name='Warehouse Revenue', index=False)
    top5_per_warehouse.to_excel(writer, sheet_name='Top Products per Warehouse', index=False)
    warehouse_delivery_time.to_excel(writer, sheet_name='Delivery Speed', index=False)
    product_sales.to_excel(writer, sheet_name='Product Sales', index=False)
    country_demand.to_excel(writer, sheet_name='Country Demand', index=False)
    returns_by_product.to_excel(writer, sheet_name='Returns', index=False)

print("✅ Report exported successfully!")
```
