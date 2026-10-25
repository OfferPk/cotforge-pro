# MITRE Cursor on Target (CoT v2.0) Technical Architecture & Specification

## 1\. Executive Summary & Specification Scope

Cursor on Target (CoT) is an XML-based tactical information exchange standard developed by MITRE and adopted by the US Department of Defense, US SOCOM, NATO, and global defense forces. CoT defines an elegant, highly structured data model designed to enable seamless machine-to-machine interoperability across heterogeneous battlefield sensors, weapons systems, drone ground control stations, and handheld tactical displays (ATAK / WinTAK / iTAK).

### Key Architectural Concepts:

1. **Core Tactical Questions: What, Where, When**:  
   - Every CoT message answers three essential questions:  
     - **What**: The hierarchical MIL-STD-2525 dot-delimited type string (e.g., `a-f-A-M-F` for Friendly Military Fixed-Wing aircraft).  
     - **Where**: The 3D geodetic position defined by `<point lat="..." lon="..." hae="..." ce="..." le="..."/>` in WGS-84 coordinates.  
     - **When**: The temporal validity defined by ISO 8601 UTC timestamps: `time` (creation), `start` (validity start), and `stale` (expiration timestamp).  
2. **Provenance & Source Attribution ("how")**:  
   - `how="m-g"`: Machine-generated GPS.  
   - `how="m-r"`: Radar tracking track.  
   - `how="m-f"`: Multi-sensor fusion engine.  
   - `how="h-e"`: Human-entered tactical point.  
3. **TAK Extension Detail Elements**:  
   - `<contact callsign="..." endpoint="..."/>`: User callsign and network addressing.  
   - `<track speed="..." course="..."/>`: Real-time kinematic velocity (m/s) and true heading (degrees).  
   - `<sensor azimuth="..." elevation="..." fov="..." range="..."/>`: Gimbal / electro-optical sensor pointing line-of-sight and footprint polygon.  
   - `<emergency alert="true" type="911"/>`: Soldier-in-danger 911 distress beacon broadcast.  
4. **Network Transport Topologies**:  
   - **UDP Multicast**: High-speed local tactical team broadcasting over `239.2.3.1:6969`.  
   - **TCP / TLS TAK Server**: Enterprise wide-area tactical hub routing over ports 8087 (clear) and 8089 (mutual TLS certificate authentication).