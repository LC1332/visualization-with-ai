# Plotly交互式可视化

需要说明的是，这套课程会使用“想到哪里，写到哪里”的方式。因为这本身是我（李鲁鲁）给自己的探索课程。

这里我们注意到datawhale已经有一个 [wow-plotly](https://github.com/datawhalechina/wow-plotly) 的项目。这个项目介绍了plotly的基础代码。但是这个repo不能满足我的几个问题

- 为什么我们要用plotly， plt不行吗？
- plotly里面一些很重要的hover之类的特性，比如我在知乎笔记中用到的这些能够体现plotly特性的并没有体现 （ https://zhuanlan.zhihu.com/p/646948797 ）
- 作为一个数据工作者，我并不想太去记忆plotly的底层代码。而想用prompt的方式来关联我的数据。

# 为什么要用plotly

我最早是在探究colab的时候发现的plotly这个库。和Matplotlib一样，plotly的图片可以直接在colab中进行显示。但是plotly有更多鼠标交互，移动到数据点上能够显示额外信息之类的功能。这些是plt所没有的。你可以用下面这个prompt，来获取更多plotly和plt之间的差别

```
我正在准备一门数据可视化的课程，第一节课会涉及plotly

我知道plotly相比matplotlib有下面特点

- 都可以在notebook 中显示
- 有交互鼠标hover显示等功能

思考一下告诉我plotly相比plt还有什么好处
```

可以发现，plotly和页面是有更好的兼容性的，这也是为什么我们在后面使用交互式数据可视化的时候，更多选用plotly的原因。

# 使用colab进行开发

由于colab无缝集成了一个（弱化的）gemini ai编程。这使得我们的课程的虚拟运行环境+ AI容易被控制。所以第一节课我会使用colab来进行讲解。

如果你不想用colab，直接使用notebook打开对应的ipynb文件就可以。但是使用colab的话你可以把prompt逐个输入，跟进整个课程的用prompt生成代码的过程。



```
player_dataframes[player_name] 存储了玩家每天的交易数据
生成 player_dataframes 的过程见buy_code中的代码

其中total_assets列记录了玩家每天的总资产 date列表示日期

我想在notebook中实现一个带状图可视化比较每个人的总资产收益率( 资产/初始值 - 1 )

- 每个人使用不同的颜色
- 每个时刻，收益率最高的那个人会显示在y轴最上方
- 也就是带状图会随着每天收益率排序的不同上下穿插
- 统计全局的最小收益率（会是一个负值） 使用 (收益率 + C). 作为带状图的纵向宽度，C = - 最小收益率
- 鼠标hover在x某个值时，会显示当前时刻每个人的收益率

<buy_code>
import random

# --- Data Preparation: Convert all_stock_data into a more accessible format ---
# Extract dates (assuming all stocks share the same date list)
dates = all_stock_data[0]['dates']
num_days = len(dates)
stock_names = [stock['name'] for stock in all_stock_data]

# Create a list of dictionaries, where each dict represents stock prices for a given day.
# This allows for easy lookup of all stock prices on a specific day.
daily_stock_prices = []
for day_idx in range(num_days):
    day_prices = {}
    for stock_info in all_stock_data:
        day_prices[stock_info['name']] = stock_info['prices'][day_idx]
    daily_stock_prices.append(day_prices)

# --- Simulation Parameters ---
# Percentage of total assets to sell/buy each day
transaction_percentage = 0.2

# --- Simulate Daily Trading ---
for day_idx in range(num_days):
    current_date = dates[day_idx]
    current_prices = daily_stock_prices[day_idx]

    for player_name, player_state in players_data.items():
        # --- 1. Calculate Player's Total Assets (before any transactions for the day) ---
        current_total_assets = player_state['cash']
        current_holdings_values = {} # To store the market value of each stock holding
        
        # Sum up the value of current stock holdings
        for stock_name, holding in player_state['holdings'].items():
            if holding['shares'] > 0:
                stock_current_value = holding['shares'] * current_prices[stock_name]
                current_total_assets += stock_current_value
                current_holdings_values[stock_name] = stock_current_value
        
        # --- 2. Implement Selling Logic ---
        # If the player has any stocks, they will try to sell
        if current_holdings_values: # Check if there are any stocks to potentially sell
            # Select a random stock from the player's current holdings
            stock_to_sell_name = random.choice(list(current_holdings_values.keys()))
            
            sell_amount_target = transaction_percentage * current_total_assets
            stock_value_in_holding = current_holdings_values[stock_to_sell_name]
            
            # Actual sell amount is the minimum of the target or the total value of the holding
            actual_sell_amount = min(sell_amount_target, stock_value_in_holding)

            if actual_sell_amount > 0 and current_prices[stock_to_sell_name] > 0:
                shares_to_sell = actual_sell_amount / current_prices[stock_to_sell_name]
                player_state['cash'] += actual_sell_amount
                player_state['holdings'][stock_to_sell_name]['shares'] -= shares_to_sell
                
                # Remove stock from holdings if shares become negligible
                if player_state['holdings'][stock_to_sell_name]['shares'] < 1e-9: # Use a small epsilon for float comparison
                    del player_state['holdings'][stock_to_sell_name]

        # --- 3. Implement Buying Logic ---
        # Recalculate total assets after selling (this affects the buy amount calculation)
        current_total_assets_after_sell = player_state['cash']
        for stock_name, holding in player_state['holdings'].items():
            if holding['shares'] > 0:
                current_total_assets_after_sell += holding['shares'] * current_prices[stock_name]

        if player_state['cash'] > 0: # Only buy if there's cash available
            # Select a random stock to buy from all available stocks
            stock_to_buy_name = random.choice(stock_names)
            
            buy_amount_target = transaction_percentage * current_total_assets_after_sell
            # Actual buy amount is the minimum of the target or all available cash
            actual_buy_amount = min(buy_amount_target, player_state['cash'])

            if actual_buy_amount > 0 and current_prices[stock_to_buy_name] > 0:
                shares_to_buy = actual_buy_amount / current_prices[stock_to_buy_name]
                player_state['cash'] -= actual_buy_amount
                
                # Add shares to holding, creating a new entry if the stock wasn't held before
                if stock_to_buy_name not in player_state['holdings']:
                    player_state['holdings'][stock_to_buy_name] = {'shares': 0} # Initialize shares
                player_state['holdings'][stock_to_buy_name]['shares'] += shares_to_buy

        # --- 4. Calculate Daily Total Assets and Store Records ---
        daily_record = {'date': current_date}
        daily_record['cash'] = player_state['cash']
        
        # Calculate total assets at the end of the day after all transactions
        total_assets_end_of_day = player_state['cash']
        for stock_name in stock_names: # Include all stock columns for consistency
            holding_shares = player_state['holdings'].get(stock_name, {'shares': 0})['shares']
            stock_value = holding_shares * current_prices[stock_name]
            daily_record[f'{stock_name}_value'] = stock_value
            total_assets_end_of_day += stock_value
        
        daily_record['total_assets'] = total_assets_end_of_day
        player_state['daily_records'].append(daily_record)

# --- 5. Create Pandas DataFrames for each player ---
player_dataframes = {}
for player_name, player_state in players_data.items():
    df = pd.DataFrame(player_state['daily_records'])
    player_dataframes[player_name] = df

# --- 6. Display Example Output ---
print(f"Generated trading data for {len(player_names)} players over {num_days} days.")
print("\n--- Example DataFrame for '阿土伯' (first 5 rows) ---")
print(player_dataframes['阿土伯'].head())

print("\n--- Final Assets for all players ---")
for player_name, df in player_dataframes.items():
    print(f"{player_name}: Final Assets = {df['total_assets'].iloc[-1]:.2f}")

</buy_code>
```