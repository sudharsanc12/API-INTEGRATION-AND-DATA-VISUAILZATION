import streamlit as st
import requests
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime

# Streamlit setup
st.set_page_config(page_title="Stock Market Dashboard", layout="wide")
st.title("📈 Stock Market Analysis Dashboard")
st.markdown("Powered by Alpha Vantage (Free API)")

# Input your API key here
API_KEY = "your_api_key_here"

# Stock options
stock_dict = {
    "Apple (AAPL)": "AAPL",
    "Microsoft (MSFT)": "MSFT",
    "Google (GOOGL)": "GOOGL",
    "Amazon (AMZN)": "AMZN",
    "Tesla (TSLA)": "TSLA"
}

selected_stocks = st.multiselect("Select stocks to analyze", list(stock_dict.keys()), default=["Apple (AAPL)", "Microsoft (MSFT)"])

# Fetch function
@st.cache_data(ttl=3600)
def fetch_stock_data(symbol):
    url = f"https://www.alphavantage.co/query"
    params = {
        "function": "TIME_SERIES_INTRADAY",
        "symbol": symbol,
        "interval": "60min",
        "apikey": API_KEY
    }
    response = requests.get(url, params=params)
    data = response.json()

    if "Time Series (60min)" not in data:
        return pd.DataFrame()

    raw = data["Time Series (60min)"]
    df = pd.DataFrame(raw).T
    df.columns = ["Open", "High", "Low", "Close", "Volume"]
    df.index = pd.to_datetime(df.index)
    df = df.astype(float)
    df.sort_index(inplace=True)
    return df

# Collect data for selected stocks
stock_dfs = {}
for name in selected_stocks:
    symbol = stock_dict[name]
    df = fetch_stock_data(symbol)
    if df.empty:
        st.warning(f"⚠️ No data for {symbol}")
    else:
        stock_dfs[symbol] = df

# Stop if no data
if not stock_dfs:
    st.error("🚫 No stock data available. Try again or check API key.")
    st.stop()

# Show data table
st.subheader("📊 Latest Prices (Hourly)")
for symbol, df in stock_dfs.items():
    st.markdown(f"**{symbol}**")
    st.dataframe(df.tail())

# Line chart
st.subheader("📉 Closing Price Over Time")
plt.figure(figsize=(12, 5))
for symbol, df in stock_dfs.items():
    sns.lineplot(data=df["Close"], label=symbol)
plt.title("Hourly Closing Prices")
plt.xlabel("Time")
plt.ylabel("Price (USD)")
plt.xticks(rotation=45)
plt.legend()
st.pyplot(plt)

# Box plot
st.subheader("📦 Price Distribution")
closing_prices = pd.DataFrame({symbol: df["Close"] for symbol, df in stock_dfs.items()})
plt.figure(figsize=(8, 5))
sns.boxplot(data=closing_prices)
plt.title("Stock Price Distribution (Hourly)")
plt.ylabel("Price (USD)")
st.pyplot(plt)

st.success("✅ Dashboard loaded successfully.")
