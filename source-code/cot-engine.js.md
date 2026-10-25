/\*\*  
 \* MITRE Cursor on Target (CoT v2.0) Core Protocol Engine  
 \* Implements XML event synthesis, hierarchical MIL-STD-2525 dot taxonomy,  
 \* WGS-84 3D point georeferencing, kinematic track vectors, and TAK detail elements.  
 \*/

// Hierarchical CoT Type Taxonomy (MIL-STD-2525 / CoT Mapping)  
const COT\_TYPES \= {  
  FRIENDLY\_AIR\_FIGHTER: 'a-f-A-M-F',  
  FRIENDLY\_AIR\_DRONE: 'a-f-A-M-F-Q',  
  FRIENDLY\_GROUND\_INFANTRY: 'a-f-G-U-C-I',  
  FRIENDLY\_GROUND\_ARMOR: 'a-f-G-E-V-A',  
  HOSTILE\_AIR\_FIGHTER: 'a-h-A-M-F',  
  HOSTILE\_GROUND\_ARMOR: 'a-h-G-E-V-A',  
  NEUTRAL\_GROUND: 'a-n-G',  
  SENSOR\_POINT\_OF\_INTEREST: 'b-m-p-s-p-i',  
  CAS\_9LINE\_TARGET: 'b-m-p-s-c-a-s',  
  EMERGENCY\_BEACON: 'b-a-o-pan'  
};

// Data Acquisition Provenance ("how")  
const COT\_HOW \= {  
  GPS\_DERIVED: 'm-g',  
  RADAR\_DERIVED: 'm-r',  
  FUSED\_SENSOR: 'm-f',  
  SIMULATED: 'm-s',  
  HUMAN\_ENTERED: 'h-e'  
};

class CoTEngine {  
  constructor(defaultUid \= 'VIPER\_01') {  
    this.defaultUid \= defaultUid;  
    this.eventSequence \= 0;  
  }

  /\*\*  
   \* Generates ISO 8601 UTC timestamp format: YYYY-MM-DDTHH:MM:SS.mmmZ  
   \*/  
  static getIsoTimestamp(offsetSeconds \= 0\) {  
    const d \= new Date(Date.now() \+ offsetSeconds \* 1000);  
    return d.toISOString();  
  }

  /\*\*  
   \* Constructs an XML Cursor on Target (CoT v2.0) Event  
   \*/  
  generateEvent(options \= {}) {  
    this.eventSequence++;  
    const uid \= options.uid \|\| \`\${this.defaultUid}-\${this.eventSequence}\`;  
    const type \= options.type \|\| COT\_TYPES.FRIENDLY\_AIR\_FIGHTER;  
    const how \= options.how \|\| COT\_HOW.GPS\_DERIVED;

    const time \= CoTEngine.getIsoTimestamp(0);  
    const start \= options.start \|\| time;  
    const staleDurationSec \= options.staleSec \|\| 120;  
    const stale \= CoTEngine.getIsoTimestamp(staleDurationSec);

    // Geodetic Point  
    const lat \= options.lat \!== undefined ? options.lat : 33.7490;  
    const lon \= options.lon \!== undefined ? options.lon : \-84.3880;  
    const hae \= options.hae \!== undefined ? options.hae : 7500.0; // Height Above Ellipsoid (m)  
    const ce \= options.ce \!== undefined ? options.ce : 5.0;       // Circular Error (m)  
    const le \= options.le \!== undefined ? options.le : 10.0;      // Linear Error (m)

    // Optional Detail Fields  
    const callsign \= options.callsign \|\| this.defaultUid;  
    const speedMs \= options.speedMs \!== undefined ? options.speedMs : 250.0;  
    const courseDeg \= options.courseDeg \!== undefined ? options.courseDeg : 45.0;  
    const batteryPct \= options.batteryPct \!== undefined ? options.batteryPct : 95;  
    const isEmergency \= options.isEmergency \|\| false;

    // Assemble Detail XML sub-elements  
    let detailInner \= \`\<contact callsign="\${callsign}" endpoint="192.168.1.100:4242"/\>\\n\` \+  
                      \`    \<track speed="\${speedMs.toFixed(1)}" course="\${courseDeg.toFixed(1)}"/\>\\n\` \+  
                      \`    \<status battery="\${batteryPct}" readiness="true"/\>\`;

    if (isEmergency) {  
      detailInner \+= \`\\n    \<emergency alert="true" type="911" cancel="false"/\>\`;  
    }

    if (options.sensor) {  
      detailInner \+= \`\\n    \<sensor azimuth="\${options.sensor.azimuth \|\| 0}" elevation="\${options.sensor.elevation \|\| \-15}" fov="\${options.sensor.fov \|\| 30}" range="\${options.sensor.range \|\| 5000}"/\>\`;  
    }

    // Full CoT Event XML  
    const xml \= \`\<?xml version="1.0" standalone="yes"?\>\\n\` \+  
                \`\<event version="2.0" uid="\${uid}" type="\${type}" time="\${time}" start="\${start}" stale="\${stale}" how="\${how}"\>\\n\` \+  
                \`  \<point lat="\${lat.toFixed(6)}" lon="\${lon.toFixed(6)}" hae="\${hae.toFixed(1)}" ce="\${ce.toFixed(1)}" le="\${le.toFixed(1)}"/\>\\n\` \+  
                \`  \<detail\>\\n\` \+  
                \`    \${detailInner}\\n\` \+  
                \`  \</detail\>\\n\` \+  
                \`\</event\>\`;

    return {  
      uid: uid,  
      type: type,  
      time: time,  
      stale: stale,  
      lat: lat,  
      lon: lon,  
      hae: hae,  
      callsign: callsign,  
      speedMs: speedMs,  
      courseDeg: courseDeg,  
      xml: xml,  
      xmlByteLength: new TextEncoder().encode(xml).length  
    };  
  }

  /\*\*  
   \* Validates standard CoT Event XML structure  
   \*/  
  static validateCoT(xmlString) {  
    const hasEvent \= xmlString.includes('\<event') && xmlString.includes('\</event\>');  
    const hasVersion \= xmlString.includes('version="2.0"');  
    const hasPoint \= xmlString.includes('\<point') && xmlString.includes('lat=') && xmlString.includes('lon=');  
    const hasDetail \= xmlString.includes('\<detail\>') && xmlString.includes('\</detail\>');

    return {  
      isValid: hasEvent && hasVersion && hasPoint && hasDetail,  
      hasEvent: hasEvent,  
      hasVersion: hasVersion,  
      hasPoint: hasPoint,  
      hasDetail: hasDetail  
    };  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= {  
    CoTEngine,  
    COT\_TYPES,  
    COT\_HOW  
  };  
}  
