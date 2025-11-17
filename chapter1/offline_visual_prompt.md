all_stock_data是一个list of dict
每个元素有三个字段name(string), dates(list), prices(list)
code

我希望实现一个plotly的折线图绘制每个股票的价格
- 横轴是时间，纵轴是价格

<code>
import numpy as np
import pandas as pd
from datetime import datetime, timedelta

# Stock names
stock_names = [
    '大庆石油', '可口可乐', '迪士尼', 
    '摩托罗拉', '王府井百货', '顶新食品'
]

# Simulation parameters
initial_price = 100
num_days = 80

# List to hold all stock data
all_stock_data = []

# Generate data for each stock
for name in stock_names:
    prices = [initial_price]
    dates = [datetime.now() - timedelta(days=num_days - 1)] # Start date

    for i in range(1, num_days):
        # Random walk: add a standard normal random variable
        new_price = prices[-1] + np.random.randn()
        prices.append(max(0, new_price)) # Ensure price doesn't go below 0
        dates.append(dates[-1] + timedelta(days=1))
    
    # Format dates as strings for readability
    formatted_dates = [d.strftime('%Y-%m-%d') for d in dates]

    stock_info = {
        'name': name,
        'dates': formatted_dates,
        'prices': prices
    }
    all_stock_data.append(stock_info)

# Display the first stock's data as an example
print(f"Generated data for {len(all_stock_data)} stocks.")
print("Example data for the first stock:")
print(all_stock_data[0])
</code>