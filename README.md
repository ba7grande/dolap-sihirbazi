import streamlit as st
import ezdxf
import os

# Streamlit başlık
st.set_page_config(page_title="Dolap Sihirbazı", layout="centered")
st.title("🛠️ Dolap Toplama Sihirbazı")

# 📏 Giriş Alanları
st.subheader("1️⃣ Ölçüleri Girin")
genislik = st.number_input("Genişlik (mm)", value=600, min_value=300)
yukseklik = st.number_input("Yükseklik (mm)", value=720, min_value=300)
derinlik = st.number_input("Derinlik (mm)", value=500, min_value=300)
kalinlik = st.number_input("Malzeme Kalınlığı (mm)", value=18)

# ✅ Panel Listesi Hesapla
st.subheader("2️⃣ Panel Listesi")
paneller = [
    {"isim": "sol_panel", "w": derinlik, "h": yukseklik},
    {"isim": "sag_panel", "w": derinlik, "h": yukseklik},
    {"isim": "arka_panel", "w": genislik, "h": yukseklik},
    {"isim": "alt_panel", "w": genislik - 2 * kalinlik, "h": derinlik},
    {"isim": "ust_panel", "w": genislik - 2 * kalinlik, "h": derinlik},
    {"isim": "kapak", "w": genislik, "h": yukseklik}
]
for p in paneller:
    st.markdown(f"✅ `{p['isim']}`: {int(p['w'])} x {int(p['h'])} mm")

# 🧩 DXF Çizim Fonksiyonu
def dxf_ciz(panel, klasor, delik_offset=37, delik_cap=5):
    doc = ezdxf.new()
    msp = doc.modelspace()
    w, h = panel["w"], panel["h"]
    msp.add_lwpolyline([(0, 0), (w, 0), (w, h), (0, h)], close=True)
    for x in [delik_offset, w - delik_offset]:
        for y in [delik_offset, h - delik_offset]:
            msp.add_circle((x, y), delik_cap)
    os.makedirs(klasor, exist_ok=True)
    dosya = os.path.join(klasor, f"{panel['isim']}.dxf")
    doc.saveas(dosya)

# 📂 DXF ve Yerleşim Üret
st.subheader("3️⃣ DXF & Yerleşim Üret")
if st.button("📁 DXF Üret ve Plakaya Yerleştir"):
    panel_klasor = "paneller_dxf"
    for p in paneller:
        dxf_ciz(p, panel_klasor)

    # 🔲 Yerleşim için yeni DXF
    doc = ezdxf.new()
    msp = doc.modelspace()
    plaka_w, plaka_h = 2100, 2800
    x, y, max_y = 0, 0, 0
    padding = 10

    for p in paneller:
        panel_file = os.path.join(panel_klasor, f"{p['isim']}.dxf")
        if not os.path.exists(panel_file): continue
        panel_doc = ezdxf.readfile(panel_file)
        panel_msp = panel_doc.modelspace()
        width, height = p["w"], p["h"]

        if x + width > plaka_w:
            x = 0
            y += max_y + padding
            max_y = 0
        if y + height > plaka_h:
            st.warning(f"{p['isim']} plakaya sığmadı.")
            continue
        for e in panel_msp:
            e_copy = e.copy()
            e_copy.translate(dx=x, dy=y)
            msp.add_entity(e_copy)
        x += width + padding
        if height > max_y:
            max_y = height

    doc.saveas("yerlesim.dxf")
    st.success("✅ DXF dosyaları ve yerleşim çizimi tamamlandı!")
