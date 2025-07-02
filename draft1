import streamlit as st
import pandas as pd
import requests
from datetime import datetime, timedelta

st.set_page_config(page_title="Henrico Move-In Finder", layout="centered")
st.title("🏡 Henrico County New Move-Ins")

# User inputs
zipcode = st.text_input("Enter ZIP Code (Henrico only):", max_chars=5)
days_back = st.slider("How many days back?", 1, 90, 30)

# Setup API query if user input is given
if zipcode:
    try:
        # Calculate date cutoff in milliseconds since epoch
        cutoff_date = datetime.today() - timedelta(days=days_back)
        cutoff_millis = int(cutoff_date.timestamp()) * 1000

        # Define Henrico ArcGIS endpoint
        BASE_URL = "https://gis3.henrico.us/arcgis/rest/services/External/Property/MapServer/0/query"
        params = {
            "where": f"SALEDATE >= {cutoff_millis} AND SITUSZIP = '{zipcode}'",
            "outFields": "SITUSADDR,SITUSZIP,SALEDATE,OWNERNAME",
            "f": "json",
            "returnGeometry": "false"
        }

        # Make the request
        response = requests.get(BASE_URL, params=params)
        data = response.json()

        # Parse data
        features = data.get("features", [])
        records = [f["attributes"] for f in features]
        df = pd.DataFrame(records)

        if not df.empty:
            df["SALEDATE"] = pd.to_datetime(df["SALEDATE"], unit="ms")
            df.rename(columns={
                "SITUSADDR": "Address",
                "SITUSZIP": "ZIP Code",
                "SALEDATE": "Sale Date",
                "OWNERNAME": "Owner"
            }, inplace=True)
            st.success(f"Found {len(df)} properties sold in the last {days_back} days in ZIP {zipcode}.")
            st.dataframe(df.sort_values("Sale Date", ascending=False).reset_index(drop=True))
        else:
            st.info("No recent move-ins found for that ZIP code.")

    except Exception as e:
        st.error("Error fetching data. Please try again later.")
        st.exception(e)
else:
    st.warning("Please enter a ZIP code to search Henrico County.")
