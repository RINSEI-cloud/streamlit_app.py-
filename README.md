# streamlit_app.py-
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

CSV_FILE = "worklog.csv"

st.set_page_config(page_title="勤務記録アプリ", layout="centered")
st.title("📋 勤務記録アプリ（スマホ対応）")

with st.form("勤務記録"):
    st.markdown("### 📝 勤務記録を入力")
    name = st.text_input("👤 名前")
    date = st.date_input("📅 日付", datetime.now().date())
    start_time = st.time_input("🕒 出勤時間")
    end_time = st.time_input("🕒 退勤時間")
    task = st.text_area("🛠 作業内容", height=100)
    submitted = st.form_submit_button("✅ 記録を保存")

if submitted:
    hours = (datetime.combine(date, end_time) - datetime.combine(date, start_time)).seconds / 3600
    data = pd.DataFrame([{
        "名前": name,
        "日付": date.strftime("%Y-%m-%d"),
        "出勤": start_time.strftime("%H:%M"),
        "退勤": end_time.strftime("%H:%M"),
        "作業内容": task,
        "勤務時間": round(hours, 2)
    }])
    
    try:
        old_data = pd.read_csv(CSV_FILE)
        data = pd.concat([old_data, data], ignore_index=True)
    except FileNotFoundError:
        pass

    data.to_csv(CSV_FILE, index=False)
    st.success("✅ 勤務記録を保存しました")

try:
    df = pd.read_csv(CSV_FILE)
    st.markdown("## 📊 勤務時間の推移")
    df["日付"] = pd.to_datetime(df["日付"])
    df = df.sort_values("日付")
    fig, ax = plt.subplots()
    for name in df["名前"].unique():
        user_data = df[df["名前"] == name]
        ax.plot(user_data["日付"], user_data["勤務時間"], marker='o', label=name)
    ax.set_ylabel("勤務時間（h）")
    ax.set_xlabel("日付")
    ax.legend()
    st.pyplot(fig)
except:
    st.info("📭 まだ記録がありません。")
