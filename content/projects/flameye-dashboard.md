---
title: "Flameye — AI-Powered Wildfire Detection & Intelligence System"
description: "A real-time wildfire detection and situational awareness platform using YOLOv8, live camera streams, and GIS integration."
date: 2026-06-06
draft: false
url: "https://github.com/Hajorda/Flameye-dashboard"
---

# Flameye — AI-Powered Wildfire Detection & Intelligence System

Flameye is a real-time wildfire detection and situational awareness platform. It ingests live camera streams, runs a YOLOv8-based fire/smoke detection model on every frame, and presents detected events on an interactive GIS dashboard with physics-based fire spread prediction, NASA satellite cross-referencing, and multi-camera incident clustering.

## Key Features
- **Real-time YOLO inference:** Runs on every camera frame to detect fire and smoke.
- **GIS & Fire Intelligence:** Integrates Rothermel fire spread models, NASA FIRMS hotspots, and elevation data.
- **Incident Management:** Alert deduplication and incident clustering using Haversine distance.
- **Live Dashboard:** Zero polling latency with WebSocket pushed live feed, interactive map, and camera streams.