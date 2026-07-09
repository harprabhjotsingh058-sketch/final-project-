"""
Mohali Weather Dashboard
Run with: streamlit run weather_dashboard.py
Requires: pip install streamlit pandas numpy seaborn matplotlib
"""

import streamlit as st
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# ---------------------------------------------------------
# Page config
# ---------------------------------------------------------
st.set_page_config(
    page_title="Mohali Weather Dashboard",
    page_icon="🌦️",
    layout="wide",
)

sns.set_theme(style="whitegrid")

# ---------------------------------------------------------
# Data (live snapshot fetched for Mohali, Punjab, India)
# ---------------------------------------------------------
CURRENT = {
    "location": "Mohali, Punjab, India",
    "temperature_f": 82.0,
    "condition": "Light rain (drizzle)",
    "is_day": True,
}

daily_data = [
    {"date": "2026-07-09", "day": "Thursday", "high_f": 82.2, "precip_chance": 95, "precip_type": "rain"},
    {"date": "2026-07-10", "day": "Friday",   "high_f": 91.0, "precip_chance": 35, "precip_type": "rain"},
    {"date": "2026-07-11", "day": "Saturday", "high_f": 91.7, "precip_chance": 55, "precip_type": "rain"},
    {"date": "2026-07-12", "day": "Sunday",   "high_f": 95.0, "precip_chance": 35, "precip_type": "rain"},
    {"date": "2026-07-13", "day": "Monday",   "high_f": 96.9, "precip_chance": 35, "precip_type": "rain"},
]

df = pd.DataFrame(daily_data)
df["date"] = pd.to_datetime(df["date"])
df["high_c"] = (df["high_f"] - 32) * 5.0 / 9.0  # numpy-friendly vectorized conversion
df["high_c"] = np.round(df["high_c"], 1)

# ---------------------------------------------------------
# Header
# ---------------------------------------------------------
st.title("🌦️ Mohali Weather Dashboard")
st.caption(f"Live forecast snapshot for {CURRENT['location']}")

# ---------------------------------------------------------
# Current conditions - metric cards
# ---------------------------------------------------------
col1, col2, col3 = st.columns(3)
col1.metric("Current Temperature", f"{CURRENT['temperature_f']}°F")
col2.metric("Condition", CURRENT["condition"])
col3.metric("Today's High", f"{df.iloc[0]['high_f']}°F", f"{df.iloc[0]['precip_chance']}% rain")

st.divider()

# ---------------------------------------------------------
# Forecast table
# ---------------------------------------------------------
st.subheader("5-Day Forecast Data")
st.dataframe(
    df[["day", "date", "high_f", "high_c", "precip_chance", "precip_type"]].rename(
        columns={
            "day": "Day",
            "date": "Date",
            "high_f": "High (°F)",
            "high_c": "High (°C)",
            "precip_chance": "Rain Chance (%)",
            "precip_type": "Type",
        }
    ),
    use_container_width=True,
    hide_index=True,
)

st.divider()

# ---------------------------------------------------------
# Charts
# ---------------------------------------------------------
chart_col1, chart_col2 = st.columns(2)

with chart_col1:
    st.subheader("High Temperature Trend")
    fig1, ax1 = plt.subplots(figsize=(6, 4))
    sns.lineplot(data=df, x="day", y="high_f", marker="o", linewidth=2.5, color="#D9534F", ax=ax1)
    ax1.set_ylabel("High Temp (°F)")
    ax1.set_xlabel("")
    ax1.set_title("5-Day High Temperature (°F)")
    for i, row in df.iterrows():
        ax1.annotate(f"{row['high_f']:.0f}°F", (i, row["high_f"]), textcoords="offset points", xytext=(0, 8), ha="center")
    plt.tight_layout()
    st.pyplot(fig1)

with chart_col2:
    st.subheader("Rain Probability")
    fig2, ax2 = plt.subplots(figsize=(6, 4))
    colors = sns.color_palette("Blues", n_colors=len(df))
    order = np.argsort(df["precip_chance"].values)
    palette = np.array(colors)[np.argsort(order)]
    sns.barplot(data=df, x="day", y="precip_chance", palette=list(palette), ax=ax2)
    ax2.set_ylabel("Rain Chance (%)")
    ax2.set_xlabel("")
    ax2.set_title("5-Day Precipitation Chance (%)")
    ax2.set_ylim(0, 100)
    plt.tight_layout()
    st.pyplot(fig2)

st.divider()

# ---------------------------------------------------------
# Combined chart - temp vs rain (dual axis)
# ---------------------------------------------------------
st.subheader("Temperature vs. Rain Chance (Combined View)")
fig3, ax3 = plt.subplots(figsize=(10, 4.5))
ax3.plot(df["day"], df["high_f"], color="#D9534F", marker="o", linewidth=2.5, label="High Temp (°F)")
ax3.set_ylabel("High Temp (°F)", color="#D9534F")
ax3.tick_params(axis="y", labelcolor="#D9534F")

ax4 = ax3.twinx()
ax4.bar(df["day"], df["precip_chance"], alpha=0.3, color="#4A90D9", label="Rain Chance (%)")
ax4.set_ylabel("Rain Chance (%)", color="#4A90D9")
ax4.tick_params(axis="y", labelcolor="#4A90D9")

fig3.suptitle("Mohali: High Temperature & Rain Probability")
plt.tight_layout()
st.pyplot(fig3)

# ---------------------------------------------------------
# Summary stats using numpy
# ---------------------------------------------------------
st.divider()
st.subheader("Quick Stats")
stat1, stat2, stat3, stat4 = st.columns(4)
stat1.metric("Avg High (°F)", f"{np.mean(df['high_f']):.1f}")
stat2.metric("Max High (°F)", f"{np.max(df['high_f']):.1f}")
stat3.metric("Avg Rain Chance", f"{np.mean(df['precip_chance']):.0f}%")
stat4.metric("Temp Range (°F)", f"{np.ptp(df['high_f']):.1f}")

st.caption("Data snapshot fetched for Mohali, Punjab, India — values reflect the forecast at time of retrieval.")
