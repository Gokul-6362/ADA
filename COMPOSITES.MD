flowchart LR
 subgraph G1["1. SOFTWARE INPUTS"]
    direction TB
        I1["Upload Composite CAD File<br>(STEP / STP / STL)"]
        I2["Select Prepreg Material<br>(Material Specification from Dropdown)"]
  end
 subgraph STAGE1[" "]
    direction TB
        G1
        N_I2["⚙️ Configurable Parameter:<br>Prepreg Density (kg/m3), Single Ply Thickness (mm) &amp; Prepreg Cost/kg (Rs/kg)"]
  end
 subgraph G2["2. CAD EXTRACTED DATA &amp; LAMINATE PROPERTIES"]
    direction TB
        E1["CAD Model Ingestion &amp; Bounding Box:<br>Length (L), Width (W), Height (H)"]
        E2["Shell Geometry Extraction:<br>Geometric Thickness (T in mm)<br>Surface Area of One Ply (A in m2)"]
        E3["Laminate Calculations:<br>• Number of Layers: N_layers = T / T_ply<br>• Total Part Surface Area (A1): A1 = N_layers × A<br>• Volume of Part (m3): V_part = (T × A) / 1000<br>• Part Mass (kg): Mass = V_part × Density"]
        E1 --> E2
        E2 --> E3
  end
 subgraph G3["3. PARAMETERS &amp; TOOL STOCK GEOMETRY"]
    direction TB
        P1["Determine Master Tool Stock Dimensions:<br>Stock Length: L_stk = L + 100 mm<br>Stock Width: W_stk = W + 100 mm<br>Stock Height: H_stk = H (Unchanged)"]
        P2["Compute Composite Tool Volume &amp; Weight:<br>• Volume of Tool (V in m3) = (H_stk × W_stk × L_stk) / 10^9<br>• Tool Stock Weight (kg) = V × Epoxy_Block_Density (700 kg/m3)"]
        P3["Part Tool Thickness &amp; Surface Scaling:<br>• Increased Tool Thickness: T_tool = T + 5 mm (Added Thickness)<br>• Tool Surface Area: A_tool = (T_tool / T_ply) × A<br>• Increased Surface Area (A2): A2 = A_tool × (1 + 30% Area_Increase)"]
        P1 --> P2
        P2 --> P3
  end
 subgraph STAGE3[" "]
    direction TB
        G3
        N_P1["🔒 Fixed Backend Rule:<br>Master Tool Stock Allowance: W + 100mm, L + 100mm | Epoxy Density: 700 kg/m3"]
        N_P2["🔒 Fixed Backend Rule:<br>Laminate Theory: Layers = T / T_ply | Part Vol = (T×A)/1000 | Tool T = T+5mm, A2 = A_tool×1.30"]
  end
 subgraph G4["4. MANUFACTURING PROCESSES &amp; TIME CALCULATION"]
    direction TB
    subgraph G4_VOL["Time Calculation Based on Tool Volume V (m3): [Ranges: 0-2, 2-4, 4-10 m3]"]
        direction TB
        T_V1["Master Model Hours:<br>• Design Hours: (80 / 100 / 120 hrs)<br>• Machining Hours: (80 / 100 / 120 hrs)<br>• Assembly Hours: (80 / 100 / 120 hrs)<br>• Finishing Hours: (8 / 10 / 12 hrs)<br>• Inspection Hours: (8 / 10 / 12 hrs)"]
        T_V2["Part Tool Hours:<br>• Oven Curing: 24 hrs (Fixed)<br>• Trimming &amp; Inspection: (8 / 10 / 12 hrs)"]
        T_V3["Part Fabrication Hours:<br>• Autoclave Curing: 24 hrs (Fixed)<br>• Trimming &amp; Inspection: (8 / 10 / 12 hrs)<br>• NDTS Inspection: (8 / 10 / 12 hrs)"]
    end
    subgraph G4_AREA["Time Calculation Based on Surface Area:"]
        direction TB
        T_A1["Part Tool Layup Hours [Based on Increased Area A2]:<br>• Range 0-50 m2: 180 hrs<br>• Range 50-150 m2: 200 hrs<br>• Range 150-300 m2: 220 hrs"]
        T_A2["Part Fabrication Layup Hours [Based on Part Total Area A1]:<br>• Range 0-30 m2: 180 hrs<br>• Range 30-80 m2: 200 hrs<br>• Range 80-150 m2: 220 hrs"]
    end
  end
 subgraph STAGE4[" "]
    direction TB
        G4
        N_T1["🔒 Fixed Backend Rule:<br>Tool Volume Lookup: 0-2 m3, 2-4 m3, 4-10 m3 determine Curing, Machining, Assy, Trimming &amp; NDTS hours"]
        N_T2["🔒 Fixed Backend Rule:<br>Area Lookup: A2 (0-50, 50-150, 150-300 m2) &amp; A1 (0-30, 30-80, 80-150 m2) determine Layup hours"]
  end
 subgraph G5["5. MULTI-TIER COST ACCUMULATION"]
    direction TB
    subgraph G5_MASTER["Tier 1: Master Model Cost Calculation"]
        direction TB
        C_MM["Master Model Cost Breakdown:<br>• Design Cost = Design_Hours × Rs 2,500/hr<br>• Frame Fabrication Cost = V × Rs 1,00,000/m3<br>• Epoxy Block &amp; Bonding = (V × Rs 12,00,000) + Rs 20,000<br>• Machining Cost = Machining_Hours × Rs 2,500/hr<br>• Assembly Cost = Assembly_Hours × Rs 2,500/hr<br>• Finishing Cost = Finishing_Hours × Rs 2,500/hr<br>• Inspection Cost = Inspection_Hours × Rs 1,500/hr<br><b>Total Master Model Cost = Sum(All 7 Operations)</b>"]
    end
    subgraph G5_TOOL["Tier 2: Part Tool Cost Calculation"]
        direction TB
        C_PT["Part Tool Cost Breakdown:<br>• Dry Carbon Fabric Cost = A2 × Rs 9,000/m2<br>• Tool Layup Cost = Layup_Hours(A2) × Rs 2,500/hr<br>• Oven Curing Cost = Oven_Curing_Hours(V) × Rs 3,000/hr<br>• LT Inspection Cost = 1 × Rs 50,000/part<br>• Trimming &amp; Inspection = Trimming_Hours(V) × Rs 2,000/hr<br>• CFC Tubes Substructure = A1 × Rs 32,000/m2<br>• Bonding &amp; Riveting Substructure = A1 × Rs 16,500/m2<br><b>Total Tool Cost = Sum(All 7 Operations)</b>"]
    end
    subgraph G5_PART["Tier 3: Composite Part Fabrication Cost Calculation"]
        direction TB
        C_PF["Part Fabrication Cost Breakdown:<br>• Part Wt with Extra = Mass × (1 + 40% Extra)<br>• Prepreg Material Cost = Part_Wt_Extra × Prepreg_Cost/kg (Rs 30,000)<br>• Part Layup Cost = Layup_Hours(A1) × Rs 2,500/hr<br>• Consumables Cost = 20% × Prepreg_Material_Cost<br>• Autoclave Curing Cost = Autoclave_Hours(V) × Rs 4,000/hr<br>• Trimming &amp; Inspection = Trimming_Hours(V) × Rs 2,500/hr<br>• NDTS Inspection Cost = NDTS_Hours(V) × Rs 4,000/hr<br><b>Total Part Cost = Sum(All 6 Operations)</b>"]
    end
  end
 subgraph STAGE5[" "]
    direction TB
        G5
        N_CM1["⚙️ Configurable Parameter:<br>Master Tool Rates: Design (2500), Machining (2500), Assy (2500), Fin (2500), Insp (1500), Frame (100000/m3), Epoxy (1200000/m3 + 20000)"]
        N_CM2["⚙️ Configurable Parameter:<br>Part Tool Rates: Dry Carbon (9000/m2), Layup (2500/h), Oven (3000/h), LT Insp (50000), Trim (2000/h), CFC Tubes (32000/m2), Riveting (16500/m2)"]
        N_CM3["⚙️ Configurable Parameter:<br>Part Fab Rates: Extra Prepreg Wt (40%), Consumables (20%), Layup (2500/h), Autoclave (4000/h), Trim (2500/h), NDTS (4000/h)"]
  end
 subgraph G6["6. TOTAL COST AGGREGATION &amp; OVERHEADS"]
    direction TB
        TOT2["Direct Cost Subtotal:<br>Direct Cost = Total Tool Cost + Total Part Cost"]
        TOT3["Statutory Adders &amp; Overheads:<br>• Import / Export Cost = Direct Cost × 5%<br>• Contingency Cost = Direct Cost × 10%<br>• GST = Direct Cost × 18%"]
        TOT1["Total Cost Per Unit (Incl. All Adders):<br>Unit Cost = Direct Cost + Imp/Exp (5%) + Contingency (10%) + GST (18%)"]
        TOT4["Grand Total Composite Batch Cost:<br>Grand Total Cost = Unit Cost × Batch Quantity (Q)"]
        TOT2 --> TOT3
        TOT3 --> TOT1
        TOT1 --> TOT4
  end
 subgraph INFO["💡 USER INTERACTIVE FEATURES (EDITABLE IN UI)"]
    direction TB
        U1["1. Batch Quantity (Q) [Default = 1]"]
        U2["2. Master Tool Stock Dimensions Override (L_stk, W_stk, H_stk)"]
        U3["3. Material Parameter Overrides (Prepreg Density, Ply Thickness, Prepreg Cost/kg)"]
  end
    I1 --> E1
    I2 --> E3
    I2 -.-> N_I2
    E1 --> P1
    E2 --> P3
    E3 --> T_A2 & C_PF
    P1 --> P2
    P2 --> T_V1 & T_V2 & T_V3 & C_MM
    P3 --> T_A1 & C_PT
    P1 -.-> N_P1
    E2 -.-> N_P2
    T_V1 --> C_MM
    T_V2 & T_A1 --> C_PT
    T_V3 & T_A2 --> C_PF
    T_V1 -.-> N_T1
    T_A1 -.-> N_T2
    C_MM -.-> N_CM1
    C_PT -.-> N_CM2
    C_PF -.-> N_CM3
    C_PT --> TOT2
    C_PF --> TOT2
    G6 ~~~ SP_GAP1["&nbsp;"]
    SP_GAP1 ~~~ SP_GAP2["&nbsp;"]
    SP_GAP2 ~~~ INFO

     I1:::calcStyle
     I2:::calcStyle
     N_I2:::configStyle
     E1:::calcStyle
     E2:::calcStyle
     E3:::calcStyle
     P1:::calcStyle
     P2:::calcStyle
     P3:::calcStyle
     N_P1:::fixedStyle
     N_P2:::fixedStyle
     T_V1:::calcStyle
     T_V2:::calcStyle
     T_V3:::calcStyle
     T_A1:::calcStyle
     T_A2:::calcStyle
     N_T1:::fixedStyle
     N_T2:::fixedStyle
     C_MM:::calcStyle
     C_PT:::calcStyle
     C_PF:::calcStyle
     N_CM1:::configStyle
     N_CM2:::configStyle
     N_CM3:::configStyle
     TOT2:::calcStyle
     TOT3:::calcStyle
     TOT1:::calcStyle
     TOT4:::calcStyle
     U1:::infoStyle
     U2:::infoStyle
     U3:::infoStyle
    classDef default fill:#ffffff,stroke:#334155,stroke-width:1.5px,color:#0f172a
    classDef calcStyle fill:#ffffff,stroke:#1e293b,stroke-width:1.5px,color:#0f172a
    classDef configStyle fill:#ffffff,stroke:#d97706,stroke-width:1.5px,stroke-dasharray:4 4,color:#0f172a
    classDef fixedStyle fill:#ffffff,stroke:#e11d48,stroke-width:1.5px,stroke-dasharray:4 4,color:#0f172a
    classDef infoStyle fill:#ffffff,stroke:#2563eb,stroke-width:1.5px,color:#0f172a
    style G1 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G2 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G3 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G4 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G4_VOL fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G4_AREA fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G5 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G5_MASTER fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G5_TOOL fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G5_PART fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G6 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style SP_GAP1 fill:none,stroke:none,color:transparent
    style SP_GAP2 fill:none,stroke:none,color:transparent
    style INFO fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style STAGE1 fill:none,stroke:none,color:transparent
    style STAGE3 fill:none,stroke:none,color:transparent
    style STAGE4 fill:none,stroke:none,color:transparent
    style STAGE5 fill:none,stroke:none,color:transparent
