# コード  
import streamlit as st  
import pandas as pd  
from scipy.stats import poisson  
  
# --- キングハナハナ詳細設定データ ---  
settings_data = {  
    "設定1": {"big": 292, "reg": 489, "suika": 48.0, "side_blue": 0.13, "side_red": 0.07},  
    "設定2": {"big": 280, "reg": 452, "suika": 46.0, "side_blue": 0.07, "side_red": 0.13},  
    "設定3": {"big": 268, "reg": 417, "suika": 44.0, "side_blue": 0.13, "side_red": 0.07},  
    "設定4": {"big": 254, "reg": 381, "suika": 42.0, "side_blue": 0.07, "side_red": 0.13},  
    "設定5": {"big": 240, "reg": 344, "suika": 40.0, "side_blue": 0.13, "side_red": 0.07},  
    "設定6": {"big": 232, "reg": 313, "suika": 38.0, "side_blue": 0.10, "side_red": 0.10},  
}  
  
st.set_page_config(page_title="キンハナ設定推測Pro", layout="wide")  
st.title("🌺 キングハナハナ 設定推測Pro")  
  
# --- 入力エリア ---  
col1, col2 = st.columns(2)  
with col1:  
    st.header("基本データ")  
    total_g = st.number_input("総回転数", 0, 20000, 1000)  
    big = st.number_input("BIG回数", 0, 200, 3)  
    reg = st.number_input("REG回数", 0, 200, 2)  
    suika = st.number_input("BIG中のスイカ回数", 0, 200, 1)  
  
with col2:  
    st.header("REGサイドランプ")  
    st.caption("左側・右側の強弱（青・黄を弱、緑・赤を強として合算）")  
    side_left = st.number_input("左側（青・黄・緑・赤）合計", 0, 200, 1)  
    side_right = st.number_input("右側（青・黄・緑・赤）合計", 0, 200, 1)  
  
# --- ベイズ推定計算 ---  
likelihoods = []  
for s in settings_data.keys():  
    d = settings_data[s]  
    # ボーナスとスイカの生起確率  
    p_big = poisson.pmf(big, total_g / d["big"])  
    p_reg = poisson.pmf(reg, total_g / d["reg"])  
    # BIG中のゲーム数を1回あたり24Gとして計算  
    p_suika = poisson.pmf(suika, (big * 24) / d["suika"])  
      
    # サイドランプの比率による尤度  
    # （左が強い設定で左が出れば高評価、右も同様）  
    p_side = (d["side_blue"] ** side_left) * (d["side_red"] ** side_right)  
      
    # すべての要素を掛け合わせる  
    likelihoods.append(p_big * p_reg * p_suika * p_side)  
  
# 確率の正規化  
total_l = sum(likelihoods)  
if total_l > 0:  
    results = [(l / total_l) * 100 for l in likelihoods]  
else:  
    results = [16.66] * 6  
  
# --- 結果表示 ---  
res_df = pd.DataFrame({"設定": settings_data.keys(), "期待度": results})  
st.subheader("📊 解析結果")  
  
# 棒グラフ  
st.bar_chart(res_df.set_index("設定"))  
  
# 数値テーブル  
st.table(res_df.style.format({"期待度": "{:.2f}%"}))  
  
# 結論  
best_setting = res_df.loc[res_df["期待度"].idxmax(), "設定"]  
st.success(f"【結論】 現在、最も可能性が高いのは **{best_setting}** です。")  
