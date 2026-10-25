# CoTForge Pro User Guide & Developer Manual

## 1\. Quickstart Guide

1. Launch `index.html` in your web browser.  
2. In the "TAK Geospatial Entity Config" panel, set your Callsign / UID (e.g. `VIPER_01`).  
3. Select an entity type from the MIL-STD-2525 CoT Type taxonomy (e.g. `a-f-A-M-F Friendly Air: Fighter` or `b-m-p-s-p-i Sensor Point of Interest`).  
4. Enter WGS-84 Decimal Coordinates (Latitude, Longitude), Height Above Ellipsoid (HAE in meters), and ground speed (m/s).  
5. Specify the Stale Duration in seconds (e.g. `120`).  
6. Click **Generate CoT Event XML** to synthesize the complete XML event payload, validate schema compliance, and inspect the XML dissector.  
7. Click **Run Normative Audit** to evaluate all 10 MITRE Cursor on Target protocol compliance checks.

## 2\. Standalone Headless Python CLI

Run automated regression testing and compliance verification:

python3 cot-cli.py \--audit \--callsign VIPER\_01 \--type a-f-A-M-F