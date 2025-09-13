# bmi_tracker.py
import streamlit as st
import pandas as pd
from datetime import datetime

st.set_page_config(page_title="🩺 BMI Calculator & Tracker", layout="centered")

st.title("🩺 BMI Calculator & Tracker")
st.markdown(
    "Calculate your BMI, see the health category, and keep a session history. "
    "You can export your history as CSV."
)

# ---- Sidebar options ----
with st.sidebar:
    st.header("Options")
    precision = st.number_input("Decimal places", value=1, min_value=0, max_value=4, step=1)
    keep_history = st.checkbox("Keep history in this session", value=True)
    st.markdown("---")
    st.caption("BMI = weight(kg) / (height(m))²")

# ---- Input area ----
st.subheader("Enter your measurements")

col1, col2 = st.columns([1,1])
with col1:
    units = st.selectbox("Units", ["Metric (kg, cm)", "Imperial (lb, in)"])
with col2:
    sex = st.selectbox("Sex (for reference)", ["Not specified", "Male", "Female"])

if units.startswith("Metric"):
    wt = st.number_input("Weight (kg)", min_value=1.0, value=70.0, step=0.1, format="%.2f")
    ht_cm = st.number_input("Height (cm)", min_value=50.0, value=170.0, step=0.5, format="%.1f")
    height_m = ht_cm / 100.0
else:
    wt_lb = st.number_input("Weight (lb)", min_value=22.0, value=154.0, step=0.5, format="%.1f")
    ht_in = st.number_input("Height (in)", min_value=20.0, value=67.0, step=0.5, format="%.1f")
    wt = wt_lb * 0.45359237
    height_m = ht_in * 0.0254

# Calculate BMI
def calc_bmi(weight_kg, height_m):
    if height_m <= 0:
        return None
    return weight_kg / (height_m * height_m)

def bmi_category(bmi):
    if bmi is None:
        return "N/A"
    if bmi < 18.5:
        return "Underweight"
    if bmi < 25:
        return "Normal weight"
    if bmi < 30:
        return "Overweight"
    return "Obese"

if st.button("Calculate BMI"):
    bmi_value = calc_bmi(wt, height_m)
    if bmi_value is None:
        st.error("Invalid height.")
    else:
        bmi_rounded = round(bmi_value, precision)
        cat = bmi_category(bmi_value)

        # display main results
        colA, colB = st.columns([1,2])
        with colA:
            st.metric("BMI", f"{bmi_rounded}")
            # simple color-coded badge-like text
            if bmi_value < 18.5:
                st.success("Underweight")
            elif bmi_value < 25:
                st.success("Normal")
            elif bmi_value < 30:
                st.warning("Overweight")
            else:
                st.error("Obese")

        with colB:
            st.write(f"**Category:** {cat}")
            st.write(f"**Sex:** {sex}")
            if units.startswith("Metric"):
                st.write(f"**Weight:** {wt:.2f} kg")
                st.write(f"**Height:** {height_m*100:.1f} cm")
            else:
                st.write(f"**Weight:** {wt:.2f} kg (converted from lb)")
                st.write(f"**Height:** {height_m*100:.1f} cm (converted from in)")

        # save to history
        if keep_history:
            if "bmi_history" not in st.session_state:
                st.session_state.bmi_history = []
            st.session_state.bmi_history.insert(0, {
                "time": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                "bmi": float(bmi_rounded),
                "category": cat,
                "weight_kg": round(wt, 2),
                "height_cm": round(height_m * 100, 1),
                "units": "metric" if units.startswith("Metric") else "imperial",
                "sex": sex
            })

# show formulas and ranges
st.markdown("---")
st.subheader("About BMI")
st.write("Body Mass Index (BMI) is a simple calculation using weight and height.")
st.markdown(
    "- Underweight: BMI < 18.5\n"
    "- Normal weight: 18.5 ≤ BMI < 25\n"
    "- Overweight: 25 ≤ BMI < 30\n"
    "- Obese: BMI ≥ 30"
)

# show history if enabled
if keep_history:
    st.markdown("---")
    st.subheader("Session History")
    hist = st.session_state.get("bmi_history", [])
    if hist:
        df = pd.DataFrame(hist)
        # show small line chart of BMI trend
        st.line_chart(df.set_index("time")["bmi"])
        st.dataframe(df)
        csv = df.to_csv(index=False).encode("utf-8")
        st.download_button("Download history as CSV", data=csv, file_name="bmi_history.csv")
        if st.button("Clear history"):
            st.session_state.bmi_history = []
            st.success("History cleared.")
    else:
        st.info("No history yet. Calculate your BMI to start tracking.")

st.markdown("---")
st.caption("Built By Bhaskar — simple BMI tracker using Streamlit.")
