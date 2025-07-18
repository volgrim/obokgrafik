# Aplikacja webowa Streamlit do układania grafiku pracy na podstawie dyspozycji

# === Plik: app.py ===
import streamlit as st
import pandas as pd
from disposition import wczytaj_dyspozycje
from scheduler import generuj_grafik, zapotrzebowanie_domyslne
from exporter import zapisz_grafik_do_excel
import tempfile
import datetime

st.set_page_config(page_title="Generator Grafiku", layout="wide")
st.title("📅 Generator grafiku pracy")

uploaded_file = st.file_uploader("Wgraj plik z dyspozycją (.xlsx)", type=["xlsx"])

# Edytowalne zapotrzebowanie
st.sidebar.header("🛠️ Edytuj zapotrzebowanie")
zap = {}
dni_tygodnia = ["poniedziałek", "wtorek", "środa", "czwartek", "piątek", "sobota", "niedziela"]
stanowiska = ["BAR", "KUCHNIA", "KOORDYNATOR"]
for dzien in dni_tygodnia:
    zap[dzien] = {}
    st.sidebar.markdown(f"**{dzien.capitalize()}**")
    for s in stanowiska:
        zap[dzien][s] = st.sidebar.number_input(f"{s} ({dzien})", min_value=0, max_value=10, value=zapotrzebowanie_domyslne[dzien][s], key=f"{dzien}_{s}")

# Wybór zakresu dat
start_date = st.date_input("Początek zakresu dat", value=datetime.date.today())
end_date = st.date_input("Koniec zakresu dat", value=datetime.date.today() + datetime.timedelta(days=6))

if uploaded_file is not None:
    with tempfile.NamedTemporaryFile(delete=False, suffix=".xlsx") as tmp_file:
        tmp_file.write(uploaded_file.getvalue())
        tmp_path = tmp_file.name

    dyspozycje = wczytaj_dyspozycje(tmp_path)
    st.success(f"Wczytano {len(dyspozycje)} wpisów dyspozycyjności.")

    # Filtrowanie po zakresie dat
    dyspozycje = [w for w in dyspozycje if start_date <= w["data"] <= end_date]

    if st.button("Generuj grafik"):
        grafik = generuj_grafik(dyspozycje, zap)
        st.write("### 📋 Wygenerowany grafik")
        df_grafik = pd.DataFrame(grafik)
        st.dataframe(df_grafik)

        with tempfile.NamedTemporaryFile(delete=False, suffix=".xlsx") as export_file:
            zapisz_grafik_do_excel(grafik, export_file.name)
            st.download_button(
                label="📥 Pobierz grafik jako Excel",
                data=open(export_file.name, "rb").read(),
                file_name="grafik.xlsx",
                mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
            )

# === Plik: disposition.py ===
import pandas as pd
from typing import List

def wczytaj_dyspozycje(plik_sciezka: str) -> List[dict]:
    df_raw = pd.read_excel(plik_sciezka, header=None)

    dyspozycje = []
    for i in range(2, df_raw.shape[1], 2):
        name = df_raw.iloc[0, i]
        position = df_raw.iloc[1, i]
        col_od = i
        col_do = i + 1

        if pd.isna(name) or pd.isna(position):
            continue

        for row in range(3, df_raw.shape[0]):
            data = df_raw.iloc[row, 0]
            dzien = df_raw.iloc[row, 1]
            od = df_raw.iloc[row, col_od]
            do = df_raw.iloc[row, col_do]

            if pd.notna(od) and pd.notna(do) and od != "-" and do != "-":
                dyspozycje.append({
                    "data": pd.to_datetime(data).date() if pd.notna(data) else None,
                    "dzień": dzien,
                    "imię": name,
                    "stanowisko": position,
                    "od": float(od),
                    "do": float(do)
                })

    return dyspozycje

# === Plik: scheduler.py ===
from collections import defaultdict
from typing import List

zapotrzebowanie_domyslne = {
    "poniedziałek": {"BAR": 1, "KUCHNIA": 1, "KOORDYNATOR": 1},
    "wtorek": {"BAR": 1, "KUCHNIA": 1, "KOORDYNATOR": 1},
    "środa": {"BAR": 1, "KUCHNIA": 1, "KOORDYNATOR": 1},
    "czwartek": {"BAR": 1, "KUCHNIA": 1, "KOORDYNATOR": 1},
    "piątek": {"BAR": 1, "KUCHNIA": 1, "KOORDYNATOR": 1},
    "sobota": {"BAR": 2, "KUCHNIA": 2, "KOORDYNATOR": 1},
    "niedziela": {"BAR": 2, "KUCHNIA": 2, "KOORDYNATOR": 1},
}

def generuj_grafik(dyspozycje: List[dict], zapotrzebowanie: dict) -> List[dict]:
    przydzial = []
    wykorzystani = defaultdict(list)

    for wpis in dyspozycje:
        dzien = wpis["dzień"]
        stanowisko = wpis["stanowisko"].upper()
        imie = wpis["imię"]
        od = wpis["od"]
        do = wpis["do"]
        data = wpis["data"]

        if dzien not in zapotrzebowanie:
            continue

        for potrzebne_stanowisko, liczba_osob in zapotrzebowanie[dzien].items():
            if potrzebne_stanowisko in stanowisko and wykorzystani[(data, potrzebne_stanowisko)].count(imie) == 0:
                if wykorzystani[(data, potrzebne_stanowisko)].__len__() < liczba_osob:
                    przydzial.append({
                        "data": data,
                        "dzień": dzien,
                        "imię": imie,
                        "stanowisko": potrzebne_stanowisko,
                        "od": od,
                        "do": do
                    })
                    wykorzystani[(data, potrzebne_stanowisko)].append(imie)
                    break

    return przydzial

# === Plik: exporter.py ===
import pandas as pd
from typing import List

def zapisz_grafik_do_excel(grafik: List[dict], nazwa_pliku: str):
    df = pd.DataFrame(grafik)
    df.sort_values(by=["data", "stanowisko", "od"], inplace=True)
    df.to_excel(nazwa_pliku, index=False)
