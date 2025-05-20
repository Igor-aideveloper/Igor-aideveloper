- 👋 Hi, I’m @Igor-aideveloper
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Igor-aideveloper/Igor-aideveloper is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import streamlit as st
import pandas as pd
from io import BytesIO

st.title("Applicazione per Elaborazione HORIZON")

uploaded_horizon = st.file_uploader("Carica il file HORIZON.csv", type="csv")
uploaded_xyz = st.file_uploader("Carica il file XYZ_DEM.csv", type="csv")

if uploaded_horizon and uploaded_xyz:
    horizon_df = pd.read_csv(uploaded_horizon)
    xyz_df = pd.read_csv(uploaded_xyz)

    merged_df = pd.merge(horizon_df, xyz_df[['NUM_OSS', 'X', 'Y', 'Z']],
                         left_on='HOLEID', right_on='NUM_OSS')

    merged_df['X'] = merged_df['X_y']
    merged_df['Y'] = merged_df['Y_y']
    merged_df['Z_DEM'] = merged_df['Z']
    filtered_df = merged_df[merged_df['HOLEID'] == merged_df['NUM_OSS']].copy()
    filtered_df['Z_CALC'] = (filtered_df['Z_DEM'] - filtered_df['FROM']).round(3)

    filtered_df.sort_values(by=['HOLEID', 'FROM'], inplace=True)
    filtered_df.reset_index(drop=True, inplace=True)

    new_rows = []
    for i in range(1, len(filtered_df)):
        curr = filtered_df.iloc[i]
        prev = filtered_df.iloc[i - 1]
        if curr['HOLEID'] != prev['HOLEID']:
            new_row = prev.copy()
            new_row['FROM'] = None
            new_row['DESC_'] = None
            new_row['LITOLOGIA'] = None
            new_row['HORIZON'] = None
            new_row['Z_CALC'] = round(prev['Z_DEM'] - prev['TO'], 3)
            new_rows.append((i, new_row))

    for idx, row in reversed(new_rows):
        filtered_df = pd.concat(
            [filtered_df.iloc[:idx], pd.DataFrame([row]), filtered_df.iloc[idx:]]
        ).reset_index(drop=True)

    final_df = filtered_df[horizon_df.columns.tolist() + ['Z_CALC']]

    st.success("Elaborazione completata.")
    st.dataframe(final_df)

    # Download
    output = BytesIO()
    final_df.to_excel(output, index=False, engine='openpyxl')
    st.download_button("📥 Scarica Excel", data=output.getvalue(), file_name="HORIZON_modificato.xlsx")
