# Odontolog-a-a-su-alcance-
Odontología estetica y ortodoncia 
import streamlit as st
import sqlite3
from datetime import datetime, date, time

# --- DB SETUP ---
conn = sqlite3.connect('pacientes.db', check_same_thread=False)
c = conn.cursor()

c.execute('''CREATE TABLE IF NOT EXISTS pacientes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT,
    telefono TEXT,
    email TEXT,
    motivo TEXT,
    fecha TEXT,
    fecha_cita TEXT,
    hora_cita TEXT
)''')
conn.commit()

# --- UI ---
st.set_page_config(page_title="Odontología a su Alcance", layout="centered")

st.title("🦷 Odontología a su Alcance")
st.markdown("**Consultorio odontológico**")
st.markdown("📍 **Ubicación:** Chía, Cundinamarca – Calle 10 # 13-19")
st.markdown("📞 **Teléfono / WhatsApp:** 313 816 0436")
st.markdown("📸 **Instagram:** [@odonto.asualcance](https://www.instagram.com/odonto.asualcance)")

st.divider()

# --- QUICK ACTION BUTTONS ---
col1, col2, col3 = st.columns(3)
with col1:
    st.link_button("📲 WhatsApp", "https://wa.me/573138160436?text=Hola,%20vengo%20desde%20Instagram%20y%20quiero%20agendar%20una%20cita")
with col2:
    st.link_button("📸 Instagram", "https://www.instagram.com/odonto.asualcance")
with col3:
    st.link_button("📍 Ubicación", "https://www.google.com/maps/search/?api=1&query=Calle+10+%2313-19+Chía+Cundinamarca")

st.divider()

# --- SERVICES ---
st.subheader("🦷 Nuestros servicios")
st.markdown("""
- ✅ Valoración odontológica
- 😁 Limpieza dental
- 🦷 Tratamiento de caries
- ✨ Estética dental
- 📐 Ortodoncia
- 🦷 Prótesis dental
- 🚑 Urgencias odontológicas
""")

st.divider()

# --- AGENDA ---
st.subheader("📅 Agenda automática")

fecha_cita = st.date_input("Selecciona la fecha", min_value=date.today())

horarios_disponibles = [
    "08:00", "09:00", "10:00", "11:00",
    "14:00", "15:00", "16:00", "17:00"
]

# Eliminar horarios ya ocupados
c.execute("SELECT hora_cita FROM pacientes WHERE fecha_cita = ?", (str(fecha_cita),))
ocupados = [h[0] for h in c.fetchall()]
horarios_libres = [h for h in horarios_disponibles if h not in ocupados]

hora_cita = st.selectbox("Selecciona la hora", horarios_libres if horarios_libres else ["No hay horarios disponibles"])

st.divider()

# --- FORM ---
st.subheader("📝 Datos del paciente")

with st.form("form_cita"):
    nombre = st.text_input("Nombre completo")
    telefono = st.text_input("Teléfono / WhatsApp")
    email = st.text_input("Correo electrónico")
    motivo = st.selectbox("Motivo de consulta", [
        "Valoración",
        "Dolor dental",
        "Limpieza",
        "Ortodoncia",
        "Estética dental",
        "Otro"
    ])
    submit = st.form_submit_button("Confirmar cita")

if submit:
    if nombre and telefono and horarios_libres:
        c.execute("""
            INSERT INTO pacientes (nombre, telefono, email, motivo, fecha, fecha_cita, hora_cita)
            VALUES (?,?,?,?,?,?,?)
        """, (
            nombre, telefono, email, motivo,
            datetime.now().strftime("%Y-%m-%d %H:%M"),
            str(fecha_cita), hora_cita
        ))
        conn.commit()
        st.success(f"✅ Cita agendada para el {fecha_cita} a las {hora_cita}")
        st.markdown(f"[📲 Confirmar por WhatsApp](https://wa.me/573138160436?text=Hola,%20tengo%20una%20cita%20agendada%20el%20{fecha_cita}%20a%20las%20{hora_cita})")
    else:
        st.error("⚠️ Completa los datos y selecciona un horario disponible")

st.divider()

# --- MAP ---
st.subheader("📍 ¿Cómo llegar?")
st.markdown("""
<iframe src="https://www.google.com/maps?q=Calle+10+%2313-19+Chía+Cundinamarca&output=embed" width="100%" height="300" style="border:0;"></iframe>
""", unsafe_allow_html=True)

st.divider()

# --- ADMIN ---
st.subheader("🔐 Panel administrativo")
clave = st.text_input("Contraseña", type="password")

if clave == "admin123":
    st.success("Acceso concedido")
    c.execute("SELECT nombre, telefono, motivo, fecha_cita, hora_cita FROM pacientes ORDER BY fecha_cita, hora_cita")
    datos = c.fetchall()

    for p in datos:
        st.write(f"🧑 {p[0]} | 📞 {p[1]} | 📝 {p[2]} | 📅 {p[3]} | ⏰ {p[4]}")
        
