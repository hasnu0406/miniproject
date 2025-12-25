# 🥗 Indie Diety — Your Personal Health Assistant

A smart, all-in-one health assistant application built with **Streamlit** that provides personalized **Diet Planning**, **Calorie and Nutrient Analysis**, and crucial **BP/Diabetes Tracking** with a placeholder for **AI-based Risk Prediction** (IBM WML ready).

Built with **Python**, **Pandas**, and **Matplotlib**, the app features a modern **Glassy UI** with adaptive dark/light themes for a superior user experience.

-----

## 🚀 Features Overview

### 🏠 Home Page

The application opens to a clean, highly stylized **glassmorphic interface** with a responsive background and centralized navigation buttons leading to the three main functional modules.

### 🥦 1. Personalized Diet Planner

This module calculates the optimal calorie intake and generates a customized, 7-day meal plan based on user goals and regional cuisine preferences.

| Input Field | Details |
| :--- | :--- |
| **Biometrics** | Age, Gender, Height, Weight, Desired Weight |
| **Activity Level** | Select from Sedentary up to Super Active (used for TDEE calculation) |
| **Location** | Region (State in India) for culture-specific meal suggestions |
| **Diet** | Food Type (Veg / Non-Veg / Both) |
| **Goal** | Weight Loss / Weight Gain / Balanced Diet |

  * **Core Calculation**: Accurately calculates **BMR** using the **Mifflin-St Jeor Formula** and determines the daily **Calorie Target** based on the chosen goal.
  * **Meal Generation**: Generates a 7-day plan by matching the user's regional/dietary filters to suggested meals tailored to hit the calculated target calories.
  * **Exports**: Download the complete weekly diet plan as a detailed **.csv** or a simplified **.txt** file.

# 🔍 2. Calorie & Nutrient Analyzer

A dedicated tool for assessing weight goals and potential nutritional risks.

  * **Calorie Needs**: Estimates Total Daily Energy Expenditure (TDEE) using BMR and activity level.
  * **Goal Adjustment**: Suggests precise calorie intake adjustments required to achieve the **Desired Weight** within a given timeframe.
  * **Risk Assessment**: Calculates **approximate nutrient deficiency risks** (Protein, Vitamins, Minerals, Fiber), displayed in sleek, semi-transparent inline summary cards.

#  ❤️‍🩹 3. BP / Diabetes Tracker (In-Depth)

This is the heart of the health monitoring system, focusing on time-series data visualization and risk simulation.

  * **Data Entry**: Allows logging of **Blood Pressure** (Systolic/Diastolic) and **Blood Glucose** readings, complete with exact timestamps and optional notes.
  * **Data Persistence**: All readings are automatically stored in a dedicated local database file: `.indie_diety_data/bp_diabetes.csv`.
  * **Visualization**:
      * **Recent Readings Table**: Displays the last 10 entries for quick review.
      * **Trend Charts**: Generates dynamic **Matplotlib trend charts** to visualize the historical pattern of **Systolic/Diastolic pressure** and **Blood Glucose levels** over time.
  * **AI Integration (Simulated)**: Features an optional button to trigger a **Simulated AI Risk Prediction** based on the latest reading, showcasing the app's capability to integrate with cloud-based Machine Learning models.

-----

## 🧠 AI Integration (IBM WML Ready)

The function `get_wml_risk_prediction()` is a conceptual placeholder designed to demonstrate how a user's latest health data could be passed to an external AI service.

To activate real-time prediction, replace the placeholder with your **IBM Watson Machine Learning API** call logic.

**Secrets required in `.streamlit/secrets.toml`:**

```toml
IBM_WML_APIKEY = "your_ibm_api_key"
IBM_WML_URL = "your_ibm_service_url"
IBM_WML_DEPLOYMENT_ID = "your_ibm_deployment_id"
```

-----

## 🧩 Project Structure

```
Indie_Diety/
├── app.py                         # Main Streamlit app file
├── .streamlit/
│   └── secrets.toml               # (Optional) IBM WML credentials
├── .indie_diety_data/
│   └── bp_diabetes.csv            # Automatically created tracker database
├── veg_regional.csv               # Vegetarian dataset (for Diet Planner)
├── nonveg_regional.csv            # Non-vegetarian dataset (for Diet Planner)
├── both_regional.csv              # Mixed diet dataset (for Diet Planner)
└── README.md                      # This file
```

-----

## ⚙️ Installation & Setup

1️⃣ **Clone or Download the Repository**

```bash
git clone https://github.com/yourusername/Indie-Diety.git
cd Indie-Diety
```

2️⃣ **Create a Virtual Environment (Recommended)**

```bash
python -m venv venv
source venv/bin/activate      # On macOS/Linux
venv\Scripts\activate         # On Windows
```

3️⃣ **Install Dependencies**

Ensure you have a `requirements.txt` file containing:

```
streamlit
pandas
numpy
matplotlib
```

Install with:

```bash
pip install -r requirements.txt
```

4️⃣ **Run the App**

```bash
streamlit run app.py
```

Then open your browser at:

```
http://localhost:8501
```


# Program:
```PYTHON
import streamlit as st
import pandas as pd
import numpy as np
import os
import datetime
import matplotlib.pyplot as plt
from matplotlib import colors # <<< ADDED THIS IMPORT FOR RGBA CONVERSION
from io import BytesIO

# --- Constants ---
DATA_DIR = ".indie_diety_data"
BP_FILE = os.path.join(DATA_DIR, "bp_diabetes.csv")
os.makedirs(DATA_DIR, exist_ok=True)

st.set_page_config(page_title="Indie Diety", layout="wide")

# --- Custom CSS for UI/UX Enhancement ---
st.markdown("""
<style>
/* ----------------------------------------------------------- */
/* --- 1. FULL-PAGE BACKGROUND INJECTION (HOME PAGE ONLY) --- */
/* Target the root Streamlit container for the home page background */
.home-page-style [data-testid="stAppViewContainer"] {
    background-image: url("https://www.health.com/thmb/VTFKH3uDDUdr-xekmilbNim5QOM=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc()/GettyImages-1498826563-db5d95e08d294ae6bc3860f901ee0c40.jpg");
    background-size: cover;
    background-position: center; 
    background-attachment: fixed;
    z-index: 0; /* Ensures background image is at the very back */
}

/* --- 2. GLASSY BLUR AND OVERLAY EFFECT (HOME PAGE ONLY) --- */
/* CRITICAL: Low opacity dark tint + strong blur for transparent, glossy effect */
.home-page-style [data-testid="stAppViewContainer"]::before {
    content: '';
    position: fixed; 
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    
    background-color: rgba(15, 15, 15, 0.2); /* Very low opacity dark tint */
    backdrop-filter: blur(16px); /* Stronger blur for frosted glass */
    
    z-index: 1; 
}

/* ----------------------------------------------------------- */
/* --- 3. OTHER PAGES BACKGROUND (Dark abstract for non-home pages) --- */
.other-pages-background [data-testid="stAppViewContainer"] {
    background-image: url("https://images.unsplash.com/photo-1571772922129-399a986c7478?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1470&q=80"); 
    background-size: cover;
    background-position: center; 
    background-attachment: fixed;
    z-index: 0; 
}

/* Dark overlay for other pages */
.other-pages-background [data-testid="stAppViewContainer"]::before {
    content: '';
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.7); 
    backdrop-filter: blur(8px); 
    z-index: 1; 
}
/* ----------------------------------------------------------- */


/* --- GENERAL STYLING --- */

/* Base App Styling for all content */
.stApp {
    z-index: 2; 
}

/* Default text color (for dark backgrounds on non-home pages) */
.stApp, label, .st-bf, .st-ck, .st-cu, div[data-testid="stAlert"] {
    color: #F8F8F8; /* Light text for dark backgrounds */
}
/* Default header colors (for dark backgrounds on non-home pages) */
h1, h2, h3, h4 {
    color: #E0E0E0; 
    font-weight: 700;
    /* Minimal shadows for text: no text-shadow used */
}

/* Main Title Styling */
.stTitle {
    color: #4CAF50; 
    font-weight: 800;
}

/* --- ADAPTIVE COLORS FOR THE LIGHTER/TRANSPARENT HOME PAGE --- */
/* Override text/header colors to be dark on the home page's background */
.home-page-style .stApp, 
.home-page-style label, 
.home-page-style .st-bf, 
.home-page-style .st-ck, 
.home-page-style .st-cu,
.home-page-style div[data-testid="stAlert"] {
    color: #333333 !important; /* Dark text for the light/transparent home page */
}
.home-page-style h1, .home-page-style h2, .home-page-style h3, .home-page-style h4 {
    color: #1a1a1a !important; /* Darker headers on light home page for contrast */
}


/* --- Button Styling (No change) --- */
div.stButton > button, .stDownloadButton > button {
    background-color: #007BFF;
    color: white;
    border-radius: 10px; 
    border: none;
    padding: 10px 25px;
    font-weight: 600;
    box-shadow: 0 5px 0 #0056b3; 
    transition: all 0.15s ease-out; 
    transform: translateY(0);
    cursor: pointer;
}
div.stButton > button:hover, .stDownloadButton > button:hover {
    background-color: #0084ff;
    box-shadow: 0 7px 0 #0056b3; 
    transform: translateY(-2px);
}
div.stButton > button:active, .stDownloadButton > button:active {
    background-color: #0056b3;
    box-shadow: 0 2px 0 #0056b3;
    transform: translateY(3px);
}
/* --- END Button Styling --- */


/* Main Containers and Forms Styling (Glass effect) */
.main .block-container {
    padding-top: 3rem;
    padding-bottom: 3rem;
    z-index: 2; 
}

/* Semi-transparent dark glass effect for non-home page containers */
section.main, div[data-testid="stSidebar"] > div:first-child, [data-testid="stVerticalBlock"] > div:has(div[data-testid="stSidebarNav"]) {
    background-color: rgba(33, 37, 41, 0.7); /* Dark translucent for default pages */
    border-radius: 15px;
    padding: 20px;
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.4);
    border: 1px solid rgba(255, 255, 255, 0.1); 
}
/* Lighter, more glassy effect for home page containers */
.home-page-style section.main, .home-page-style div[data-testid="stSidebar"] > div:first-child, .home-page-style [data-testid="stVerticalBlock"] > div:has(div[data-testid="stSidebarNav"]) {
    background-color: rgba(255, 255, 255, 0.4); 
    border: 1px solid rgba(255, 255, 255, 0.3); 
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15); 
}


/* 3D Glassy Interactive Finish for st.dataframe */
.stDataFrame {
    background: rgba(255, 255, 255, 0.15); 
    backdrop-filter: blur(10px); 
    border-radius: 15px;
    border: 1px solid rgba(255, 255, 255, 0.2); 
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.2);
    padding: 15px;
    overflow: hidden; 
    color: #F8F8F8; 
    transition: transform 0.3s ease-in-out, box-shadow 0.3s ease-in-out;
}
.stDataFrame:hover {
    transform: translateY(-5px) scale(1.01); 
    box-shadow: 0 8px 40px rgba(0, 0, 0, 0.3);
}
/* Adjust dataframe background/text for home page (dark glass on light background) */
.home-page-style .stDataFrame {
    background: rgba(0, 0, 0, 0.15); 
    border: 1px solid rgba(0, 0, 0, 0.2);
    color: #F8F8F8; 
}

/* Adjusting Matplotlib chart text colors */
.st-emotion-cache-16obb1t path, .st-emotion-cache-16obb1t text, 
.st-emotion-cache-10q7j36 path, .st-emotion-cache-10q7j36 text { 
    color: #E0E0E0 !important; 
    fill: #E0E0E0 !important;
}
/* Ensure Matplotlib chart text is dark on home page */
.home-page-style .st-emotion-cache-16obb1t path, .home-page-style .st-emotion-cache-16obb1t text, 
.home-page-style .st-emotion-cache-10q7j36 path, .home-page-style .st-emotion-cache-10q7j36 text {
    color: #1a1a1a !important; 
    fill: #1a1a1a !important;
}

/* Special styling for the calorie display section in the 'calorie' page */
div.st-emotion-cache-1629p8f { 
    background-color: rgba(76, 175, 80, 0.1) !important; 
    border: 1px solid #4CAF50 !important;
    border-radius: 15px;
    padding: 20px;
    z-index: 2; 
    color: inherit; 
}

</style>
""", unsafe_allow_html=True)
# --- END Custom CSS ---

# --- Helper Functions (No changes needed here) ---
def save_tracker_entry(row):
    df = pd.DataFrame([row])
    if os.path.exists(BP_FILE):
        df_old = pd.read_csv(BP_FILE) if os.path.getsize(BP_FILE) > 0 else pd.DataFrame()
        df_all = pd.concat([df_old, df], ignore_index=True)
    else:
        df_all = df
    df_all.to_csv(BP_FILE, index=False)

def load_tracker():
    if os.path.exists(BP_FILE) and os.path.getsize(BP_FILE) > 0:
        return pd.read_csv(BP_FILE)
    else:
        return pd.DataFrame()

def bmr_mifflin_combined(age, gender, weight, height):
    if gender.lower() == 'male':
        return 88.362 + (13.397 * weight) + (4.799 * height) - (5.677 * age)
    else:
        return 447.593 + (9.247 * weight) + (3.098 * height) - (4.330 * age)

def activity_multiplier_combined(level):
    mapping = {
        "Sedentary (little or no exercise)": 1.2,
        "Lightly active (1–3 days/week)": 1.375,
        "Moderately active (3–5 days/week)": 1.55,
        "Very active (6–7 days/week)": 1.725,
        "Super active (twice/day)": 1.9,
        "Extra active (very hard exercise or job)": 1.9
    }
    return mapping.get(level, 1.2)

def convert_df_to_csv(df):
    return df.to_csv(index=False).encode('utf-8')

def to_pdf_placeholder(df):
    buffer = BytesIO()
    buffer.write(df.to_string().encode('utf-8'))
    return buffer.getvalue()

# --- IBM WML Placeholder Function (No changes needed here) ---
def get_wml_risk_prediction(data_row, api_key, url, deployment_id):
    """
    CONCEPTUAL FUNCTION: This is where you would integrate the IBM WML API call.
    It takes the latest BP/Glucose reading and WML credentials.
    """
    st.info("Attempting to connect to IBM WML...")
    
    # --- PLACEHOLDER LOGIC ---
    bp_status = "Normal"
    if data_row['systolic'] >= 130 or data_row['diastolic'] >= 80:
        bp_status = "Elevated"
    
    glucose_status = "Normal"
    if data_row['glucose'] > 140:
        glucose_status = "High"

    st.success(f"Successfully simulated WML call.")
    st.warning("⚠️ Remember to replace this function with actual IBM WML API logic.")
    return f"**Simulated BP/Diabetes Risk:** BP Status: {bp_status} | Glucose: {glucose_status}"
# --- END IBM WML Placeholder Function ---

# --- Navigation ---
if 'page' not in st.session_state:
    st.session_state.page = 'home'

def go_to(page):
    st.session_state.page = page

# Apply conditional styling based on the current page
if st.session_state.page == 'home':
    st.markdown('<div class="home-page-style">', unsafe_allow_html=True)
else:
    st.markdown('<div class="other-pages-background">', unsafe_allow_html=True)

st.title("Indie Diety — Your Personal Health Assistant")
st.markdown("A Streamlit app: Diet Planner • Calorie & Nutrient Analyzer • BP/Diabetes Tracker")

# --- HOME PAGE ---
if st.session_state.page == 'home':
    col1, col2, col3 = st.columns(3)
    with col1:
        if st.button("Diet Planner 🥗", use_container_width=True): 
            go_to('diet')
    with col2:
        if st.button("Calorie & Nutrient Analyzer 🔍", use_container_width=True): 
            go_to('calorie')
    with col3:
        if st.button("BP / Diabetes Tracker ❤️‍🩹", use_container_width=True): 
            go_to('tracker')

# --- DIET PLANNER PAGE ---
elif st.session_state.page == 'diet':
    st.header("Personalized Diet Planner 🥦")
    st.sidebar.button("🏠 Home", on_click=lambda: go_to('home')) 

    DATA_FILES = {
        "Veg": "veg_regional.csv",
        "Non-Veg": "nonveg_regional.csv",
        "Both": "both_regional.csv",
    }

    with st.form("diet_form"):
        col1, col2, col3 = st.columns(3)
        with col1:
            age = st.number_input("Enter your age", 10, 100, 25)
            gender = st.selectbox("Select Gender", ["Male", "Female", "Other"])
        with col2:
            height = st.number_input("Enter your height (cm)", 100.0, 250.0, 170.0)
            weight = st.number_input("Enter your current weight (kg)", 30.0, 200.0, 65.0)
        with col3:
            desired_weight = st.number_input("Enter your desired weight (kg)", 30.0, 200.0, 60.0)
            activity_level = st.selectbox("Activity Level", [
                "Sedentary (little or no exercise)",
                "Lightly active (1–3 days/week)",
                "Moderately active (3–5 days/week)",
                "Very active (6–7 days/week)",
                "Super active (twice/day)"
            ])

        region = st.selectbox("Select your region (State in India)", sorted([
            "Tamil Nadu", "Kerala", "Karnataka", "Andhra Pradesh", "Telangana",
            "Maharashtra", "Gujarat", "Rajasthan", "Delhi", "Punjab", "Haryana",
            "Uttar Pradesh", "West Bengal", "Bihar", "Odisha", "Madhya Pradesh",
            "Assam", "Goa", "Jammu and Kashmir", "Others"
        ]))

        food_type_options = ['Veg', 'Non-Veg', 'Both']
        food_type = st.radio("Choose your food type:", food_type_options)
        goal = st.selectbox("Select your goal:", ["Balanced Diet", "Weight Loss", "Weight Gain"])
        submit = st.form_submit_button("Generate Diet Plan 🚀")

    if submit:
        dataset_file = DATA_FILES.get(food_type)
        dataset_path = os.path.join(DATA_DIR, dataset_file)
        if not os.path.exists(dataset_path):
            dataset_path = dataset_file

        try:
            diet_df = pd.read_csv(dataset_path)
            rename_map = {
                'Dish Name': 'dish_name', 'Food Name': 'dish_name',
                'Food Type': 'food_type', 'Cuisine State/Region': 'region',
                'Region': 'region', 'Calories (kcal)': 'calories', 'Calories': 'calories',
                'Protein (g)': 'protein', 'Protein': 'protein', 'Carbs (g)': 'carbs',
                'Carbohydrates (g)': 'carbs', 'Fat (g)': 'fat', 'Meal Type': 'meal_type'
            }
            diet_df = diet_df.rename(columns=rename_map)
            for col in ['dish_name', 'food_type', 'region', 'meal_type']:
                if col in diet_df.columns:
                    diet_df[col] = diet_df[col].astype(str).str.strip().str.title()
            st.success(f"✅ Loaded {len(diet_df)} {food_type} food items successfully.")
        except Exception as e:
            st.error(f"⚠️ Failed to load dataset: {e}")
            diet_df = pd.DataFrame()

        bmr = bmr_mifflin_combined(age, gender, weight, height)
        maintenance_calories = bmr * activity_multiplier_combined(activity_level)
        target_calories = maintenance_calories + (500 if goal == "Weight Gain" else -500 if goal == "Weight Loss" else 0)

        st.subheader("💪 Your Daily Calorie Goal")
        st.info(f"For a **{goal.lower()}**, your daily calorie target is **{int(target_calories)} kcal/day**")

        if not diet_df.empty:
            filtered_df = diet_df.copy()
            if "region" in filtered_df.columns:
                filtered_df = filtered_df[filtered_df["region"].astype(str).str.contains(region, case=False, na=False)]
            if food_type == 'Both':
                filtered_df = filtered_df[filtered_df["food_type"].isin(['Veg', 'Non-Veg', 'Both'])]
            else:
                filtered_df = filtered_df[filtered_df["food_type"].str.lower() == food_type.lower()]
            if "calories" in filtered_df.columns:
                target_meal_calories = target_calories / 3
                filtered_df["calories"] = pd.to_numeric(filtered_df["calories"], errors='coerce')
                filtered_df = filtered_df.dropna(subset=["calories"])
                filtered_df["calorie_diff"] = abs(filtered_df["calories"] - target_meal_calories)
                filtered_df = filtered_df.sort_values(by="calorie_diff")

            DAYS_OF_WEEK = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
            MEAL_TYPES = ["Breakfast", "Lunch", "Dinner"]
            weekly_plan, full_plan_list = {}, []

            meal_groups = {
                meal: filtered_df[filtered_df['meal_type'].astype(str).str.contains(meal, case=False, na=False)]
                .head(7).reset_index(drop=True) for meal in MEAL_TYPES
            }

            for day_index, day in enumerate(DAYS_OF_WEEK):
                daily_meals = []
                for meal_type in MEAL_TYPES:
                    meal_pool = meal_groups[meal_type]
                    if not meal_pool.empty:
                        dish_index = day_index % len(meal_pool)
                        dish = meal_pool.loc[dish_index]
                        meal_entry = {
                            "Day": day, "Meal": meal_type, "Dish": dish['dish_name'],
                            "Calories": round(dish['calories']),
                            "Protein (g)": round(dish['protein'], 1),
                            "Carbs (g)": round(dish['carbs'], 1),
                            "Fat (g)": round(dish['fat'], 1)
                        }
                    else:
                        meal_entry = {"Day": day, "Meal": meal_type, "Dish": "No match found",
                                      "Calories": 0, "Protein (g)": 0, "Carbs (g)": 0, "Fat (g)": 0}
                    daily_meals.append(meal_entry)
                    full_plan_list.append(meal_entry)
                weekly_plan[day] = pd.DataFrame(daily_meals)

            st.subheader(f"📅 Weekly {goal} Diet Plan for {region} ({food_type})")
            st.caption(f"Meals target ~{int(target_calories/3)} kcal per meal.")

            cols = st.columns(2)
            for i, day in enumerate(DAYS_OF_WEEK):
                with cols[i % 2]:
                    st.markdown(f"**{day} Plan**")
                    # Using container styling on the dataframe for visual appeal
                    st.dataframe(weekly_plan[day].set_index('Meal')[["Dish", "Calories"]],
                                 use_container_width=True)

            st.markdown("---")
            full_plan_df = pd.DataFrame(full_plan_list)
            csv_data = convert_df_to_csv(full_plan_df)
            pdf_data = to_pdf_placeholder(full_plan_df)

            col1, col2 = st.columns(2)
            with col1:
                st.download_button("Download CSV 💾", data=csv_data, 
                                   file_name=f'IndieDietPlan_{region}_{food_type}_{goal}.csv',
                                   mime='text/csv', use_container_width=True)
            with col2:
                st.download_button("Download TXT (Simplified) 📄",
                                   data=pdf_data,
                                   file_name=f'IndieDietPlan_{region}_{food_type}_{goal}.txt',
                                   mime='text/plain', use_container_width=True)

# --- CALORIE DEFICIENCY PAGE ---
elif st.session_state.page == 'calorie':
    st.header('Nutrient Deficiency & Calorie Assessment 📊') 
    st.sidebar.button('🏠 Home', on_click=lambda: go_to('home'))

    with st.form('deficiency_form'):
        col1, col2 = st.columns(2)
        with col1:
            weight = st.number_input('Weight (kg)', 20.0, 300.0, 70.0)
            height = st.number_input('Height (cm)', 100.0, 250.0, 170.0)
            age = st.number_input('Age (years)', 5, 120, 30)
            gender = st.selectbox('Gender', ['Male', 'Female', 'Other'])
        with col2:
            activity = st.selectbox('Activity Level', [
                'Sedentary (little or no exercise)',
                "Lightly active (light exercise 1-3 days/week)",
                "Moderately active (moderate exercise 3-5 days/week)",
                "Very active (hard exercise 6-7 days/week)",
                "Extra active (very hard exercise or job)"
            ])
            diet_type = st.selectbox('Diet Type', ['Vegetarian', 'Non-Vegetarian', 'Vegan'])
            calorie_intake = st.number_input('Daily Calorie Intake (kcal)', 500, 7000, 2200)
            meals = st.number_input('Meals per Day', 1, 10, 3)
            target_days = st.number_input('Target Duration (days)', 1, 365, 30)
            desired_weight = st.number_input('Desired Weight (kg)', 20.0, 300.0, 65.0)
            purpose = st.selectbox('Purpose', ['Weight Loss', 'Weight Gain', 'Balanced Diet'])
        submit = st.form_submit_button('Analyze & Get Recommendations 💡')

    if submit:
        bmr = 10 * weight + 6.25 * height - 5 * age + (5 if gender.lower() == 'male' else -161)
        mult = activity_multiplier_combined(activity)
        maintenance = int(bmr * mult)
        calorie_diff = (desired_weight - weight) * 7700 / target_days
        recommended_intake = maintenance + calorie_diff

        st.markdown(
            f"<div style='padding:20px;border-radius:15px;background:rgba(76, 175, 80, 0.1); border: 1px solid #4CAF50; color: inherit;'>",
            unsafe_allow_html=True
        )
        st.markdown(f"<h3 style='color:#4CAF50;'>Estimated Daily Calorie Target: {recommended_intake:.0f} kcal</h3>", unsafe_allow_html=True)
        st.markdown(
            f"<p>Based on your goal to <b>{purpose}</b> from {weight}kg to {desired_weight}kg in {target_days} days.</p>",
            unsafe_allow_html=True
        )
        st.markdown("<hr>", unsafe_allow_html=True)
        st.markdown("<h4>Approximate Nutrient Deficiency Risks:</h4>", unsafe_allow_html=True)
        st.write("Protein: 10–15% | Vitamin: 5–10% | Minerals: 5–10% | Fiber: 10–20%")
        st.markdown("</div>", unsafe_allow_html=True)

# --- BP / DIABETES TRACKER PAGE ---
elif st.session_state.page == 'tracker':
    st.header('BP / Diabetes Tracker ❤️‍🩹') 
    st.sidebar.button("🏠 Home", on_click=lambda: go_to('home'))

    with st.form('entry_form'):
        col1, col2, col3 = st.columns(3)
        with col1:
            date = st.date_input('Date', datetime.date.today())
            time = st.time_input('Time', datetime.datetime.now().time())
        with col2:
            systolic = st.number_input('Systolic (mmHg)', 50, 250, 120)
            diastolic = st.number_input('Diastolic (mmHg)', 30, 150, 80)
        with col3:
            glucose = st.number_input('Blood Glucose (mg/dL)', 20.0, 600.0, 100.0)
            notes = st.text_input('Notes (optional)')
        save = st.form_submit_button('Save Reading ➕')

    if save:
        ts = datetime.datetime.combine(date, time)
        row = {
            "timestamp": ts,
            "systolic": systolic,
            "diastolic": diastolic,
            "glucose": glucose,
            "notes": notes
        }
        save_tracker_entry(row)
        st.success('Reading saved successfully! ✅')

    df = load_tracker()
    
    # --- IBM WML Integration Section ---
    st.markdown("### 🧠 Risk Assessment (AI Powered)") 
    if not df.empty:
        latest_reading = df.sort_values(by='timestamp', ascending=False).iloc[0].to_dict()
        st.write(f"Latest Reading ({latest_reading['timestamp']}): BP={latest_reading['systolic']}/{latest_reading['diastolic']}, Glucose={latest_reading['glucose']}")
        
        if st.button('Get WML-based Risk Prediction 🤖'):
            api_key = st.secrets.get("IBM_WML_APIKEY")
            url = st.secrets.get("IBM_WML_URL")
            deployment_id = st.secrets.get("IBM_WML_DEPLOYMENT_ID")
            
            if api_key and url and deployment_id:
                risk_result = get_wml_risk_prediction(latest_reading, api_key, url, deployment_id)
                st.success(risk_result)
            else:
                st.error("IBM WML secrets (APIKEY, URL, DEPLOYMENT_ID) are missing from Streamlit secrets.")
    else:
        st.info("Save a reading first to get a risk prediction.")
    
    st.markdown("---")
    # --- End IBM WML Integration Section ---

    if not df.empty:
        df['timestamp'] = pd.to_datetime(df['timestamp'])
        df = df.sort_values(by='timestamp')
        
        st.subheader("📈 Recent Readings Table") 
        st.dataframe(df.tail(10)) 

        # --- Chart Styling and RGBA Fix ---
        
        # 1. Determine the correct colors for the current page
        text_color_chart = '#E0E0E0' if st.session_state.page != 'home' else '#1a1a1a'
        
        # Define the RGBA CSS strings for the legend facecolor
        legend_facecolor_str = 'rgba(33, 37, 41, 0.8)' if st.session_state.page != 'home' else 'rgba(255, 255, 255, 0.8)'
        legend_edgecolor = '#E0E0E0' if st.session_state.page != 'home' else '#333333'
        
        # 2. CONVERT THE RGBA STRING to a Matplotlib-compatible tuple
        # THIS FIXES THE ValueError
        try:
            legend_facecolor_tuple = colors.to_rgba(legend_facecolor_str)
        except ValueError:
            # Fallback to a safe hex color if conversion fails for any reason
            legend_facecolor_tuple = '#212529' 


        st.subheader("🩸 Blood Pressure Trend")
        fig, ax = plt.subplots(figsize=(10, 5))
        
        # Setting background and title/label colors for Matplotlib
        fig.patch.set_facecolor('none') 
        ax.set_facecolor('none')
        
        ax.plot(df['timestamp'], df['systolic'], label='Systolic (mmHg)', marker='o', linestyle='-', color='#007BFF')
        ax.plot(df['timestamp'], df['diastolic'], label='Diastolic (mmHg)', marker='o', linestyle='-', color='#28a745')
        
        # Apply colors
        ax.set_title('Blood Pressure Trend', color=text_color_chart)
        ax.set_ylabel('Pressure (mmHg)', color=text_color_chart)
        ax.set_xlabel('Date/Time', color=text_color_chart)
        
        ax.tick_params(axis='x', colors=text_color_chart)
        ax.tick_params(axis='y', colors=text_color_chart)
        
        ax.spines['left'].set_color(text_color_chart)
        ax.spines['bottom'].set_color(text_color_chart)
        ax.spines['right'].set_color('none')
        ax.spines['top'].set_color('none')
        
        ax.grid(True, linestyle='--', alpha=0.4, color='#999999')
        # Use the CONVERTED tuple here:
        ax.legend(facecolor=legend_facecolor_tuple, edgecolor=legend_edgecolor, labelcolor=text_color_chart) 
        
        plt.xticks(rotation=45, ha='right')
        plt.tight_layout()
        st.pyplot(fig)
        
        st.subheader("🍬 Blood Glucose Trend")
        fig2, ax2 = plt.subplots(figsize=(10, 5))

        fig2.patch.set_facecolor('none')
        ax2.set_facecolor('none')
        
        ax2.plot(df['timestamp'], df['glucose'], label='Blood Glucose (mg/dL)', color='#ffc107', marker='o', linestyle='-')
        
        # Apply colors
        ax2.set_title('Blood Glucose Trend', color=text_color_chart)
        ax2.set_ylabel('Glucose (mg/dL)', color=text_color_chart)
        ax2.set_xlabel('Date/Time', color=text_color_chart)
        
        ax2.tick_params(axis='x', colors=text_color_chart)
        ax2.tick_params(axis='y', colors=text_color_chart)
        
        ax2.spines['left'].set_color(text_color_chart)
        ax2.spines['bottom'].set_color(text_color_chart)
        ax2.spines['right'].set_color('none')
        ax2.spines['top'].set_color('none')
        
        ax2.grid(True, linestyle='--', alpha=0.4, color='#999999')
        ax2.legend(facecolor=legend_facecolor_tuple, edgecolor=legend_edgecolor, labelcolor=text_color_chart)

        plt.xticks(rotation=45, ha='right')
        plt.tight_layout()
        st.pyplot(fig2)

st.markdown('---')
st.caption('Indie Diety — IBM z Datathon / LinuxOne Prototype')
# Close the div added for conditional styling
st.markdown('</div>', unsafe_allow_html=True)
```
-----

## 🎨 UI Highlights

  * **Dual Background Modes**: Light transparent home background (for a glossy feel) and dark blurred backgrounds for functional pages (for better data contrast).
  * **Interactive Design**: Fully responsive buttons and forms with engaging visual feedback.
  * **Glass Effects**: Custom CSS provides 3D hover effects for dataframes and charts, enhancing interactivity.
  * **Adaptive Theming**: Dynamic Matplotlib theming automatically adjusts text and grid colors based on the page's background context.

-----

## 🧮 Core Calculations

| Calculation | Formula Used |
| :--- | :--- |
| **✅ BMR (Mifflin-St Jeor)** | **Male**: $88.362 + (13.397 \times weight) + (4.799 \times height) − (5.677 \times age)$<br>**Female**: $447.593 + (9.247 \times weight) + (3.098 \times height) − (4.330 \times age)$ |
| **✅ Calorie Goal Adjustment** | **Weight Loss**: Maintenance level $-500\ kcal/day$<br>**Weight Gain**: Maintenance level $+500\ kcal/day$ |

-----

## 📊 Data Outputs

| Output | Type | Content |
| :--- | :--- | :--- |
| **Weekly Plan** | CSV/TXT | Day-wise list of dishes with estimated calories, protein, carbs, and fat. |
| **BP Tracker** | CSV | Timestamped systolic, diastolic, glucose readings, and notes. |
| **Charts** | Matplotlib | Interactive time-series trends for Blood Pressure and Glucose. |

-----

## Interface :

### Home:
<img width="1911" height="911" alt="image" src="https://github.com/user-attachments/assets/44d39d78-1478-46bf-ac25-a6d648530b67" />

### Diet planner:
<img width="1879" height="896" alt="image" src="https://github.com/user-attachments/assets/eb9ebc00-68be-4b20-a74b-38c9a31daf52" />

### Nutrient Deficiency & Calorie Assessment 📊

<img width="1883" height="842" alt="image" src="https://github.com/user-attachments/assets/78024654-34e6-46db-8ad8-2badf7575d75" />

### BP / Diabetes Tracker ❤️‍🩹
<img width="1856" height="841" alt="image" src="https://github.com/user-attachments/assets/de8b241c-23e9-4939-9a9f-f55d27ae6bc0" />
<img width="1894" height="771" alt="image" src="https://github.com/user-attachments/assets/c3d1b5e8-796b-4b58-bd45-b83d879fe5e6" />

## 🔒 Data Privacy

All sensitive user data, including BP/Glucose readings, is stored **locally** in the `.indie_diety_data/bp_diabetes.csv` file. The application is built to prioritize privacy, with **no external storage or tracking** used by default.

-----

## 🌐 Future Enhancements

  * ✅ Integration with **real IBM WML prediction API**.
  * 🔄 **Real-time food nutrition lookup** via external APIs (e.g., Edamam, FatSecret).
  * 📱 Improved **Mobile-optimized UI**.
  * 🔔 Automated **Health alerts** and progress tracking dashboard.

-----

### 👨‍💻 Developed By

**code vectors**
