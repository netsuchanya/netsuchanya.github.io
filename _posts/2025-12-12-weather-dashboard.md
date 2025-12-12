---
title: "Project: Weather Data Visualization Dashboard"
date: 2025-12-12
permalink: /posts/weather-viz-dashboard/
tags:
  - data-visualization
  - weather-analysis
  - python
  - matplotlib
  - forecasting
---

# 🌦️ Project: Weather Data Visualization & Forecasting Dashboard

Effective weather analysis requires looking at data through different lenses: the long-term trends for planning and the short-term details for immediate action. This project utilizes Python to visualize weather forecast data, split into two distinct modules: a **7-Day Strategic Outlook** and a **3-Hour Tactical View**.

---

## 📅 Part 1: The 7-Day Forecast (Strategic View)

The first module focuses on the week ahead. The goal here is to identify general trends in temperature and precipitation to aid in weekly planning.

### 1. Temperature Trend (Line Chart)
This chart provides a quick visual summary of the temperature trajectory over the next 7 days, allowing us to spot warming or cooling trends immediately.
<img src="/images/7daymax.png" alt="7-Day Temperature Trend Line Chart" width="100%">

### 2. Regional Heatmap (Spatial Analysis)
A heatmap visualization that displays temperature distribution across the target region. This helps identify "hotspots" or cooler zones geographically.
<img src="/images/tempmax.png" alt="Regional Temperature Heatmap" width="100%">

### 3. Daily Rainfall Intensity (Bar Chart)
This bar chart quantifies the expected precipitation for each day, highlighting wet vs. dry periods clearly.
<img src="/images/rainfall.png" alt="Daily Rainfall Bar Chart" width="100%">

### 4. Min/Max Temperature Range
A visualization of the daily temperature amplitude. This plot shows the gap between the daily low (minimum) and daily high (maximum), which is crucial for understanding daily volatility.
<img src="/images/minmax.png" alt="Min/Max Temperature Range Plot" width="100%">

### Python Code: 7-Day Visualization
Below is the code used to generate the four charts above.

```python
# [PASTE YOUR CODE FOR THE 7-DAY FORECAST HERE]
import requests
import pandas as pd
import matplotlib.pyplot as plt
import geopandas as gpd
import numpy as np

# ==============================================================================
# 1. ตั้งค่าพิกัดจังหวัด (ตัวแทนภาคต่างๆ)
# ==============================================================================
provinces_coords = [
    {"name": "Bangkok", "lat": 13.75, "lon": 100.50},
    {"name": "Chiang Mai", "lat": 18.79, "lon": 98.98},
    {"name": "Chiang Rai", "lat": 19.91, "lon": 99.84},
    {"name": "Phuket", "lat": 7.88, "lon": 98.39},
    {"name": "Songkhla", "lat": 7.20, "lon": 100.60},
    {"name": "Khon Kaen", "lat": 16.43, "lon": 102.82},
    {"name": "Nakhon Ratchasima", "lat": 14.97, "lon": 102.10},
    {"name": "Ubon Ratchathani", "lat": 15.24, "lon": 104.85},
    {"name": "Chonburi", "lat": 13.36, "lon": 100.98},
    {"name": "Kanchanaburi", "lat": 14.02, "lon": 99.53},
    {"name": "Nakhon Sawan", "lat": 15.70, "lon": 100.07},
    {"name": "Prachuap Khiri Khan", "lat": 11.80, "lon": 99.79},
]

# ==============================================================================
# 2. ฟังก์ชันดึงข้อมูล (Fetch Data)
# ==============================================================================
def get_weather_dashboard():
    all_days_data = [] # เก็บข้อมูลรายวัน 7 วัน
    today_data = []    # เก็บข้อมูลเฉพาะวันนี้
    
    print(f"📡 กำลังโหลดข้อมูลพยากรณ์อากาศจาก Open-Meteo...")
    
    for p in provinces_coords:
        # ดึง 3 ค่า: อุณหภูมิสูงสุด, อุณหภูมิต่ำสุด, ปริมาณฝน
        url = f"https://api.open-meteo.com/v1/forecast?latitude={p['lat']}&longitude={p['lon']}&daily=temperature_2m_max,temperature_2m_min,precipitation_sum&timezone=Asia%2FBangkok"
        
        try:
            r = requests.get(url, timeout=5)
            if r.status_code == 200:
                data = r.json()
                daily = data['daily']
                
                # เก็บข้อมูลรายวัน (7 วัน)
                for i in range(len(daily['time'])):
                    all_days_data.append({
                        'Province': p['name'],
                        'Date': daily['time'][i],
                        'TempMax': daily['temperature_2m_max'][i],
                        'TempMin': daily['temperature_2m_min'][i], # เพิ่มค่า Min
                        'Rain': daily['precipitation_sum'][i]
                    })
                
                # เก็บข้อมูลเฉพาะวันนี้ (Index 0)
                today_data.append({
                    'name': p['name'], 
                    'TempMax': daily['temperature_2m_max'][0],
                    'TempMin': daily['temperature_2m_min'][0],
                    'Rain': daily['precipitation_sum'][0]
                })
                print(f"   ✅ {p['name']} OK")
        except Exception as e:
            print(f"   ❌ Error {p['name']}: {e}")

    return pd.DataFrame(all_days_data), pd.DataFrame(today_data)

# ==============================================================================
# 3. เริ่มการแสดงผล (Visualizations)
# ==============================================================================
df_7days, df_today = get_weather_dashboard()

if not df_7days.empty:
    
    # -----------------------------------------------------------
    # 📊 1. กราฟเส้น (Line Chart) - แนวโน้มอุณหภูมิ 7 วัน
    # -----------------------------------------------------------
    plt.figure(figsize=(10, 5))
    selected = ['Bangkok', 'Chiang Mai', 'Phuket', 'Khon Kaen'] # เลือกจังหวัดเด่นๆ
    
    for prov in selected:
        subset = df_7days[df_7days['Province'] == prov]
        plt.plot(subset['Date'], subset['TempMax'], marker='o', label=prov, linewidth=2)
    
    plt.title("1. 7-Day Max Temperature Forecast", fontsize=14)
    plt.ylabel("Temp (°C)")
    plt.legend()
    plt.grid(True, linestyle='--', alpha=0.5)
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

    # -----------------------------------------------------------
    # 🗺️ 2. แผนที่ความร้อน (Heatmap Map)
    # -----------------------------------------------------------
    print("\nกำลังวาดแผนที่...")
    try:
        map_url = "https://raw.githubusercontent.com/apisit/thailand.json/master/thailand.json"
        gdf = gpd.read_file(map_url)
        gdf_merged = gdf.merge(df_today, on='name', how='left')

        fig, ax = plt.subplots(1, 1, figsize=(8, 12))
        gdf.plot(ax=ax, color='#eeeeee', edgecolor='darkgrey')
        gdf_merged.dropna().plot(column='TempMax', cmap='hot_r', linewidth=0.8, ax=ax, edgecolor='grey', legend=True,
                                 legend_kwds={'label': "Max Temp Today (°C)", 'shrink': 0.5})
        
        for idx, row in gdf_merged.dropna().iterrows():
            ax.text(row.geometry.centroid.x, row.geometry.centroid.y, f"{row['TempMax']}°", fontsize=8, ha='center')

        ax.set_title("2. Thailand Temperature Map (Today)", fontsize=14)
        ax.set_axis_off()
        plt.show()
    except Exception as e:
        print(f"Map Error: {e}")

    # -----------------------------------------------------------
    # 🌧️ 3. [ของใหม่] กราฟแท่งปริมาณฝน (Rainfall Bar Chart)
    # -----------------------------------------------------------
    plt.figure(figsize=(10, 6))
    # เรียงลำดับจังหวัดตามปริมาณฝน
    df_rain_sorted = df_today.sort_values(by='Rain', ascending=False)
    
    bars = plt.bar(df_rain_sorted['name'], df_rain_sorted['Rain'], color='dodgerblue', alpha=0.8)
    
    plt.title("3. Rainfall Forecast Today (mm)", fontsize=14)
    plt.ylabel("Rainfall (mm)")
    plt.xticks(rotation=45)
    plt.grid(axis='y', linestyle='--', alpha=0.5)
    
    # ใส่ตัวเลขบนแท่งกราฟ
    for bar in bars:
        yval = bar.get_height()
        if yval > 0:
            plt.text(bar.get_x() + bar.get_width()/2, yval + 0.1, f"{yval}", ha='center', fontsize=9)
        else:
            plt.text(bar.get_x() + bar.get_width()/2, 0.1, "Dry", ha='center', fontsize=8, color='grey')
            
    plt.tight_layout()
    plt.show()

    # -----------------------------------------------------------
    # 🌡️ 4. [ของใหม่] กราฟเปรียบเทียบ ต่ำสุด-สูงสุด (Min-Max Comparison)
    # -----------------------------------------------------------
    plt.figure(figsize=(12, 6))
    
    x = np.arange(len(df_today))  # ตำแหน่งแกน X
    width = 0.35  # ความกว้างแท่งกราฟ

    # สร้างกราฟแท่งคู่
    plt.bar(x - width/2, df_today['TempMax'], width, label='Max Temp', color='tomato')
    plt.bar(x + width/2, df_today['TempMin'], width, label='Min Temp', color='skyblue')

    plt.title("4. Min vs Max Temperature Today", fontsize=14)
    plt.ylabel("Temperature (°C)")
    plt.xticks(x, df_today['name'], rotation=45)
    plt.legend()
    plt.grid(axis='y', linestyle='--', alpha=0.5)
    
    plt.tight_layout()
    plt.show()

    print("\n✅ แสดงผลครบทั้ง 4 กราฟ")

else:
    print("ไม่พบข้อมูล กรุณาเช็คอินเทอร์เน็ต")
````
## ⏱️ Part 2: The 3-Hour Forecast (Tactical View)
The second module zooms in for a high-resolution analysis. It processes data in 3-hour intervals to provide precise, near-term diagnostics.

### 1. 12-Hour Trend (Short-Term Line Chart)

A high-frequency line chart showing temperature fluctuations over the immediate 12-hour window. Essential for intra-day decisions. 
<img src="/images/hourlytemp.png" alt="12-Hour Short Term Trend" width="100%">

### 2. +3 Hour Forecast Map

A spatial projection of weather conditions exactly 3 hours from the current timestamp. This snapshot is vital for immediate situational awareness. 
<img src="/images/3hr_map.png" alt="Plus 3 Hour Forecast Map" width="100%">

### 3. Rainfall Accumulation

Unlike the daily bar chart, this graph tracks cumulative rainfall over short intervals, helping to predict potential flash flooding or water accumulation issues. 
<img src="/images/rainfall3hours.png" alt="Rainfall Accumulation Graph" width="100%">

### 4. "Feels Like" vs. Actual Temperature

This comparison plot highlights the difference between the recorded air temperature and the "Feels Like" index (accounting for humidity and wind). This is the most important metric for human comfort and safety. 
<img src="/images/feellike3hours.png" alt="Feels Like vs Actual Temp Graph" width="100%">

### Python Code: 3-Hour Visualization

Below is the code used to generate the short-term diagnostic charts.

````
import requests
import pandas as pd
import matplotlib.pyplot as plt
import geopandas as gpd
import numpy as np
from datetime import datetime, timedelta

# ==============================================================================
# 1. ตั้งค่าพิกัดจังหวัด
# ==============================================================================
provinces_coords = [
    {"name": "Bangkok", "lat": 13.75, "lon": 100.50},
    {"name": "Chiang Mai", "lat": 18.79, "lon": 98.98},
    {"name": "Phuket", "lat": 7.88, "lon": 98.39},
    {"name": "Khon Kaen", "lat": 16.43, "lon": 102.82},
    {"name": "Songkhla", "lat": 7.20, "lon": 100.60},
    {"name": "Nakhon Ratchasima", "lat": 14.97, "lon": 102.10},
    {"name": "Chonburi", "lat": 13.36, "lon": 100.98},
    {"name": "Kanchanaburi", "lat": 14.02, "lon": 99.53},
    {"name": "Ubon Ratchathani", "lat": 15.24, "lon": 104.85},
    {"name": "Nakhon Sawan", "lat": 15.70, "lon": 100.07},
]

# ==============================================================================
# 2. ฟังก์ชันดึงข้อมูลรายชั่วโมง (Hourly Forecast)
# ==============================================================================
def get_3hour_forecast():
    summary_data = [] # เก็บข้อมูลสรุปรายจังหวัด (ณ เวลา +3 ชม.)
    trend_data = {}   # เก็บข้อมูลกราฟเส้นรายชั่วโมง
    
    # หานาฬิกาปัจจุบัน (ชั่วโมง)
    current_hour = datetime.now().hour
    
    print(f"📡 กำลังโหลดข้อมูลรายชั่วโมง (ฐานเวลา: {current_hour}:00 น.)...")
    
    for p in provinces_coords:
        # hourly parameters: อุณหภูมิ, ความรู้สึกจริง, ฝน
        url = f"https://api.open-meteo.com/v1/forecast?latitude={p['lat']}&longitude={p['lon']}&hourly=temperature_2m,apparent_temperature,precipitation&timezone=Asia%2FBangkok&forecast_days=1"
        
        try:
            r = requests.get(url, timeout=5)
            if r.status_code == 200:
                data = r.json()
                hourly = data['hourly']
                
                # --- Logic การตัดข้อมูล ---
                # ข้อมูล hourly จะเริ่มที่ 00:00 ของวันนี้
                # เราต้องดึงข้อมูลที่ index = current_hour (ตอนนี้) ถึงอนาคต
                
                # 1. ข้อมูลสำหรับกราฟเส้น (12 ชั่วโมงข้างหน้า)
                temps_12h = hourly['temperature_2m'][current_hour : current_hour+12]
                times_12h = hourly['time'][current_hour : current_hour+12]
                # แปลงเวลาเป็นแค่ "HH:00" เพื่อให้อ่านง่าย
                times_12h_short = [t.split('T')[1] for t in times_12h]
                
                if p['name'] in ['Bangkok', 'Chiang Mai', 'Phuket', 'Khon Kaen']:
                    trend_data[p['name']] = {'times': times_12h_short, 'temps': temps_12h}

                # 2. ข้อมูลสำหรับจุดพยากรณ์ล่วงหน้า 3 ชั่วโมง (Index + 3)
                idx_target = current_hour + 3
                if idx_target >= len(hourly['time']): idx_target = len(hourly['time']) - 1 # กัน Error ข้ามวัน
                
                temp_plus_3 = hourly['temperature_2m'][idx_target]
                feels_plus_3 = hourly['apparent_temperature'][idx_target]
                
                # 3. ฝนสะสม 3 ชั่วโมงข้างหน้า (รวม Index ปัจจุบัน, +1, +2)
                rain_next_3h = sum(hourly['precipitation'][current_hour : current_hour+3])
                
                summary_data.append({
                    'name': p['name'],
                    'Temp+3h': temp_plus_3,
                    'FeelsLike+3h': feels_plus_3,
                    'RainNext3h': rain_next_3h
                })
                print(f"   ✅ {p['name']}: T+3h = {temp_plus_3}°C")
                
        except Exception as e:
            print(f"   ❌ Error {p['name']}: {e}")

    return pd.DataFrame(summary_data), trend_data, current_hour

# ==============================================================================
# 3. แสดงผล (Visualization)
# ==============================================================================
df_summary, trend_data, start_hour = get_3hour_forecast()

if not df_summary.empty:
    
    # -----------------------------------------------------------
    # 📈 1. กราฟแนวโน้ม 12 ชั่วโมง (Hourly Trend)
    # -----------------------------------------------------------
    plt.figure(figsize=(10, 5))
    colors = {'Bangkok': 'orange', 'Chiang Mai': 'blue', 'Phuket': 'green', 'Khon Kaen': 'red'}
    
    for prov_name, data in trend_data.items():
        plt.plot(data['times'], data['temps'], marker='o', label=prov_name, color=colors.get(prov_name), linewidth=2)
    
    plt.title(f"1. Hourly Temperature Trend (Next 12 Hours from {start_hour}:00)", fontsize=14)
    plt.ylabel("Temp (°C)")
    plt.xlabel("Time")
    plt.legend()
    plt.grid(True, linestyle='--', alpha=0.5)
    plt.tight_layout()
    plt.show()

    # -----------------------------------------------------------
    # 🗺️ 2. แผนที่พยากรณ์ล่วงหน้า 3 ชม. (T+3h Map)
    # -----------------------------------------------------------
    print(f"\nกำลังวาดแผนที่พยากรณ์เวลา {start_hour+3}:00 น. ...")
    try:
        map_url = "https://raw.githubusercontent.com/apisit/thailand.json/master/thailand.json"
        gdf = gpd.read_file(map_url)
        gdf_merged = gdf.merge(df_summary, on='name', how='left')

        fig, ax = plt.subplots(1, 1, figsize=(8, 12))
        gdf.plot(ax=ax, color='#eeeeee', edgecolor='darkgrey')
        
        # ระบายสีตามอุณหภูมิในอีก 3 ชม.
        gdf_merged.dropna().plot(column='Temp+3h', cmap='hot_r', linewidth=0.8, ax=ax, edgecolor='grey', legend=True,
                                 legend_kwds={'label': f"Temp at {start_hour+3}:00 (°C)", 'shrink': 0.5})
        
        for idx, row in gdf_merged.dropna().iterrows():
            ax.text(row.geometry.centroid.x, row.geometry.centroid.y, f"{row['Temp+3h']}°", fontsize=8, ha='center')

        ax.set_title(f"2. Forecast Map: +3 Hours from now", fontsize=14)
        ax.set_axis_off()
        plt.show()
    except Exception as e:
        print(f"Map Error: {e}")

    # -----------------------------------------------------------
    # 🌧️ 3. กราฟฝนสะสม 3 ชั่วโมงข้างหน้า (Rain Accumulation)
    # -----------------------------------------------------------
    plt.figure(figsize=(10, 6))
    df_rain = df_summary.sort_values(by='RainNext3h', ascending=False)
    
    bars = plt.bar(df_rain['name'], df_rain['RainNext3h'], color='cornflowerblue', alpha=0.9)
    
    plt.title(f"3. Total Rainfall Forecast (Next 3 Hours)", fontsize=14)
    plt.ylabel("Rainfall (mm)")
    plt.xticks(rotation=45)
    plt.grid(axis='y', linestyle='--', alpha=0.5)
    
    # ใส่ตัวเลข
    for bar in bars:
        yval = round(bar.get_height(), 1)
        label = f"{yval}" if yval > 0 else "0"
        plt.text(bar.get_x() + bar.get_width()/2, yval + 0.05, label, ha='center', fontsize=9)
            
    plt.tight_layout()
    plt.show()

    # -----------------------------------------------------------
    # 🌡️ 4. เปรียบเทียบ อุณหภูมิจริง vs ความรู้สึกจริง (Apparent Temp)
    # -----------------------------------------------------------
    # ดึงข้อมูล ณ เวลา +3 ชั่วโมงมาเทียบกัน
    plt.figure(figsize=(12, 6))
    
    x = np.arange(len(df_summary))
    width = 0.35

    plt.bar(x - width/2, df_summary['Temp+3h'], width, label='Actual Temp', color='salmon')
    plt.bar(x + width/2, df_summary['FeelsLike+3h'], width, label='Feels Like', color='orange')

    plt.title(f"4. Actual vs Feels Like Temperature (at {start_hour+3}:00)", fontsize=14)
    plt.ylabel("Temperature (°C)")
    plt.xticks(x, df_summary['name'], rotation=45)
    plt.legend()
    plt.grid(axis='y', linestyle='--', alpha=0.5)
    
    plt.tight_layout()
    plt.show()

    print("\n✅ แสดงผล Dashboard พยากรณ์ล่วงหน้าเรียบร้อยครับ")

else:
    print("ไม่พบข้อมูล")
````
### 📝 Data Sources & Credits
Weather Data: Real-time API provided by Open-Meteo (CC BY 4.0).

Map Data: Thailand GeoJSON via apisit/thailand.json.

Visualization: Created using Python (Matplotlib & Geopandas).
