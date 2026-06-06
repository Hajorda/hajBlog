---
title: "✈️ Flight Delay Predictor"
description: "An end-to-end Machine Learning pipeline and interactive dashboard predicting flight delays based on real-world weather conditions and route metrics."
date: 2026-06-06
draft: false
url: "https://github.com/Hajorda/flight-delay-predictor"
---

# ✈️ Flight Delay Predictor (Real-World Aviation & Weather Data Integration)

An end-to-end Machine Learning pipeline and interactive dashboard predicting flight delays based on real-world weather conditions, route metrics, time-of-day, and airport congestion, using SHAP explainability. 

This repository supports both instant local synthetic execution and robust real-world integration using official flight data from the US Bureau of Transportation Statistics (BTS) and weather observations from the National Oceanic and Atmospheric Administration (NOAA).

## Key Features
- **Real-World Integration:** Merges actual flight schedules with historical airport weather reports (NOAA ISD).
- **Leakage-Safe Feature Engineering:** Computes rolling airport congestion and delay rates without future leakage.
- **Explainable AI (SHAP):** Provides both global insights and individual flight delay breakdowns using tree and linear explainers.
- **Interactive Dashboard:** A polished, dark-theme Streamlit application displaying delay probabilities and waterfall explainers.