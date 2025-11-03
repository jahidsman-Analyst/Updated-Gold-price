import requests
import pandas as pd
import numpy as np

# Fetch USD to INR exchange rate
url_usdinr = "https://api.exchangerate-api.com/v4/latest/USD"
response_usdinr = requests.get(url_usdinr)

inr_rate = 1.0  # Default if fail
if response_usdinr.status_code == 200:
    usdinr_data = response_usdinr.json()
    inr_rate = usdinr_data['rates']['INR']
else:
    print("Error fetching USD/INR rate:", response_usdinr.status_code)

print(f"USD to INR Rate: 1 USD = ₹{inr_rate:.2f}")

# CoinGecko API for top 10 crypto in USD
url_crypto = "https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=10&page=1&sparkline=false"

response_crypto = requests.get(url_crypto)

coins_data = []

if response_crypto.status_code == 200:
    data = response_crypto.json()
    
    # Extract and convert crypto to INR
    for coin in data:
        price_inr = coin['current_price'] * inr_rate
        coins_data.append({
            'Name': coin['name'],
            'Symbol': coin['symbol'].upper(),
            'Price (INR)': f"₹{price_inr:.2f}",
            '24h Change (%)': f"{coin['price_change_percentage_24h']:.2f}%" if coin['price_change_percentage_24h'] is not None else "0.00%"
        })
else:
    print("Error fetching crypto data:", response_crypto.status_code)

# CoinGecko API for PAX Gold in INR
url_paxg = "https://api.coingecko.com/api/v3/simple/price?ids=pax-gold&vs_currencies=inr"
response_paxg = requests.get(url_paxg)

if response_paxg.status_code == 200:
    paxg_data = response_paxg.json()
    paxg_price_inr = paxg_data['pax-gold']['inr']
    
    # Fetch 24h change for PAXG (in USD, but % same)
    url_change = "https://api.coingecko.com/api/v3/coins/pax-gold/market_chart?vs_currency=usd&days=1&interval=daily"
    response_change = requests.get(url_change)
    paxg_change = 0.0
    if response_change.status_code == 200:
        change_data = response_change.json()
        if len(change_data['prices']) > 1:
            prev_price = change_data['prices'][0][1]
            curr_price = change_data['prices'][-1][1]
            paxg_change = ((curr_price - prev_price) / prev_price) * 100
    
    # Calculation: 10g rate
    oz_to_g = 31.1035  # Troy oz to grams
    price_per_g = paxg_price_inr / oz_to_g
    price_10g = price_per_g * 10
    
    # Add Gold (10g) row
    coins_data.append({
        'Name': 'Gold (10g)',
        'Symbol': '10g',
        'Price (INR)': f"₹{price_10g:.2f}",
        '24h Change (%)': f"{paxg_change:.2f}%"
    })
else:
    print("Error fetching PAXG data:", response_paxg.status_code)

df = pd.DataFrame(coins_data)

print(df.to_string(index=False))
