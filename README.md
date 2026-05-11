# streamlit_app.pyimport streamlit as st
import pandas as pd
import datetime
import random

# App Setup
st.set_page_config(page_title="Froggy Eats", page_icon="🐸")

# Custom CSS for a kid-friendly look
st.markdown("""
    <style>
    .main { background-color: #e1f5fe; }
    .stButton>button { width: 100%; border-radius: 20px; height: 3em; background-color: #4CAF50; color: white; font-weight: bold; }
    </style>
    """, unsafe_allow_html=True)

# Initialize Session Data
if 'step' not in st.session_state:
    st.session_state.update({
        'step': 'intro', 'name': '', 'food_log': [], 
        'days': 0, 'current_food': '', 'meal_type': ''
    })

# Frog Evolution Logic
def get_frog():
    days = st.session_state.days
    if days < 4: return "🐟", "Baby Tadpole"
    if days < 9: return "🦎", "Teenage Polliwog"
    return "🐸", "Happy Grown-Up Frog"

def get_joke():
    return random.choice([
        "What do frogs order at restaurants? French flies!",
        "What's a frog's favorite flower? A Croak-us!",
        "Why are frogs so happy? They eat whatever bugs them!",
        "What do you call a frog with no legs? Un-hoppy!"
    ])

# UI Header
frog_emoji, stage_name = get_frog()
st.sidebar.title(f"Level: {stage_name}")
st.sidebar.header(f"Day {st.session_state.days} / 14")
st.sidebar.progress(min(st.session_state.days / 14, 1.0))

# --- APP FLOW ---

if st.session_state.step == 'intro':
    st.title("Hi! I'm Froggy! 🐸")
    name = st.text_input("What is your name, friend?")
    if name and st.button("Let's Eat!"):
        st.session_state.name = name
        st.session_state.step = 'ask_food'
        st.rerun()

elif st.session_state.step == 'ask_food':
    hour = datetime.datetime.now().hour
    m_type = "Breakfast" if hour < 11 else "Lunch" if hour < 16 else "Dinner"
    st.session_state.meal_type = m_type
    
    st.title(f"Hi {st.session_state.name}! 👋")
    st.header(f"It's {m_type} time. What are we eating?")
    food = st.text_input("Type your food here:")
    if food and st.button("I have my plate!"):
        st.session_state.current_food = food
        st.session_state.step = 'eating'
        st.rerun()

elif st.session_state.step == 'eating':
    st.title(f"{frog_emoji} Yum!!")
    st.write(f"### Look! I have some **{st.session_state.current_food}** too!")
    
    # The "Visual" interaction
    c1, c2 = st.columns(2)
    with c1: st.metric("Your Plate", st.session_state.current_food)
    with c2: st.metric("Froggy's Plate", st.session_state.current_food)
    
    st.info(f"**Froggy says:** {get_joke()}")
    
    st.success("Take a BIG bite! Did you do it?")
    if st.button("✅ I took a bite!"):
        st.balloons()
        st.session_state.step = 'finish'
        st.rerun()

elif st.session_state.step == 'finish':
    st.title("Great Job! 🌟")
    st.write("We are getting bigger and stronger together!")
    
    if st.button("Finish Meal"):
        # Log data for parents
        entry = {
            "Date": datetime.date.today().strftime("%Y-%m-%d"),
            "Meal": st.session_state.meal_type,
            "Food": st.session_state.current_food,
            "Est. Calories": random.randint(200, 500) # Simple estimation
        }
        st.session_state.food_log.append(entry)
        st.session_state.days += 1
        st.session_state.step = 'ask_food'
        st.rerun()

# --- PARENT SECTION ---
with st.sidebar.expander("📊 Parent Dashboard"):
    if st.session_state.food_log:
        df = pd.DataFrame(st.session_state.food_log)
        st.write("Meal History")
        st.dataframe(df)
        st.bar_chart(df.set_index("Date")["Est. Calories"])
    else:
        st.write("No data yet. Start eating!")
