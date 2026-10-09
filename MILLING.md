flowchart LR
 subgraph G1["1. SOFTWARE INPUTS"]
    direction TB
        I1["Upload Design File<br>(STEP / STL / IGES)"]
        I2["Select Material<br>(materia_specification)"]
  end
 subgraph STAGE1[" "]
    direction TB
        G1
        N_I2["⚙️ Configurable Parameter:<br>Material Density  &amp; Unit Cost per kg"]
  end
 subgraph G2["2. CAD EXTRACTED DATA"]
    direction TB
        E1["PCA Alignment & Geometry Analysis"]
        E2["Actual Bounding Box:<br>L_actual, B_actual, H_actual"]
        E3["Actual Part Volume:<br>V_actual (mm3) &amp; V_actual (m3)"]
  end
 subgraph G3["3. PARAMETERS & STOCK GEOMETRY"]
    direction TB
        P1["Determine Stock Allowances:<br>delta L, delta B, delta H"]
        P2["Compute &amp; Round Stock Size:<br>L_stock = round(L_act + delta L, 10)<br>B_stock = round(B_act + delta B, 10)<br>H_stock = round(H_act + delta H, 5)"]
        P3["Stock Volume:<br>V_stock (mm3) = L_stk × B_stk × H_stk<br>V_stock (m3) = V_stock / 10^9"]
        P4["Material Removed Volume:<br>V_removed = V_stock - V_actual"]
  end
 subgraph STAGE3[" "]
    direction TB
        G3
        N_P1["🔒 Fixed Backend Rule:<br>Size-dependent Allowances "]
        N_P2["🔒 Fixed Backend Rule:<br>Rounding Length &amp; Breadth to nearest 10mm, Height to nearest 5mm"]
  end
 subgraph G4["4. MACHINING & MANUFACTURING TIME"]
    direction TB
        T1["Match Milling Parameters by L_stock<br>Fetch Feed, DOC, WOC for Material"]
        T2["Compute MRR (mm3/min):<br>MRR_rough = Feed_r × DOC_r × WOC_r<br>MRR_finish = Feed_f × DOC_f × WOC_f"]
        T3["Calculate Machining Times (min):<br>T_rough = (V_removed × P_rough / 100) / MRR_rough<br>T_finish = (V_removed × P_finish / 100) / MRR_finish"]
        T4["Total Machining Time (hrs):<br>T_machining_hr = (T_rough + T_finish) / 60"]
        T5["Total Manufacturing Time (hrs):<br>T_manuf_hr = T_machining_hr × (100 / P_manuf)"]
  end
 subgraph STAGE4[" "]
    direction TB
        G4
        N_T1["⚙️ Configurable Parameter:<br>Feed Rates, Depths of Cut (DOC), Radial WOC"]
        N_T3["⚙️ Configurable Parameter:<br>Roughing % &amp; Finishing % Splits"]
        N_T5["⚙️ Configurable Parameter:<br>Manufacturing Time % Ratio"]
  end
 subgraph G5["5. MATERIAL & MACHINING COST"]
    direction TB
        C_M1["Calculate Mass:<br>W_stock = V_stock (m3) × Density (rho)<br>W_actual = V_actual (m3) × Density (rho)"]
        C_M2["Material Cost Per Unit:<br>C_mat_unit = W_stock × C_mat"]
        C_MC1["Match Machine Rates by Size:<br>Filter machines where L,B,H_stk ≤ Machine Envelope<br>Pick smallest suitable machine"]
        C_MC2["Calculate Machine Type Cost:<br>Cost_type = T_manuf_hr × Hourly_Rate × (Dist_% / 100)"]
        C_MC3["Total Machining Cost Per Unit:<br>C_mach_unit = Sum(Cost_type) × K_corr"]
  end
 subgraph STAGE5[" "]
    direction TB
        G5
        N_CM1["⚙️ Configurable Parameter:<br>Material Density (rho)"]
        N_CM2["⚙️ Configurable Parameter:<br>Material Cost per kg"]
        N_CMC1_A["⚙️ Configurable Parameter:<br>Machine Hourly Rates &amp; Machine Envelope Sizes (L×B×H)"]
        N_CMC1_B["🔒 Fixed Backend Rule:<br>Stock Size ≤ Machine Envelope (L, B, H) &amp; Pick Smallest Envelope"]
        N_CMC3["⚙️ Configurable Parameter:<br>Manufacturing Correction Factor"]
  end
 subgraph G6["6. TOTAL COST AGGREGATION"]
    direction TB
        TOT2["Material Cost for Batch:<br>Total_Material_Cost = C_mat_unit"]
        TOT3["Manufacturing Cost Without Material:<br>C_manuf_without_mat = C_mach_unit"]
        TOT1["Total Cost Per Unit (Incl. Material):<br>C_total_unit = C_mach_unit + C_mat_unit"]
        TOT4["Grand Total Cost (With Material):<br>C_grand_total = C_total_unit × Q"]
  end
 subgraph INFO["💡 USER INTERACTIVE FEATURES (EDITABLE IN UI)"]
    direction TB
        U1["1. Batch Quantity (Q) [Default = 1]"]
        U2["2. Raw Stock Dimensions Override (L_stock, B_stock, H_stock)"]
        U3["3. Machine Distribution % (e.g., VMC 3-Axis: 60%, 5-Axis: 40%)"]
  end
    I2 -.-> N_I2
    E1 --> E2 & E3
    P1 --> P2
    P2 --> P3 & T1 & C_MC1
    P3 --> P4 & C_M1
    P1 -.-> N_P1
    P2 -.-> N_P2
    T1 --> T2
    T2 --> T3
    T3 --> T4
    T4 --> T5
    T1 -.-> N_T1
    T3 -.-> N_T3
    T5 -.-> N_T5
    C_M1 --> C_M2
    C_MC1 --> C_MC2
    C_MC2 --> C_MC3
    C_M1 -.-> N_CM1
    C_M2 -.-> N_CM2
    C_MC1 -.-> N_CMC1_A & N_CMC1_B
    C_MC3 -.-> N_CMC3
    TOT2 --> TOT1
    TOT3 --> TOT1
    TOT1 --> TOT4
    I1 --> E1
    I2 --> C_M1
    E2 --> P1
    E3 --> P4 & C_M1
    P4 --> T3
    T5 --> C_MC2
    C_M2 --> TOT2
    C_MC3 --> TOT3
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
     P4:::calcStyle
     N_P1:::fixedStyle
     N_P2:::fixedStyle
     T1:::calcStyle
     T2:::calcStyle
     T3:::calcStyle
     T4:::calcStyle
     T5:::calcStyle
     N_T1:::configStyle
     N_T3:::configStyle
     N_T5:::configStyle
     C_M1:::calcStyle
     C_M2:::calcStyle
     C_MC1:::calcStyle
     C_MC2:::calcStyle
     C_MC3:::calcStyle
     N_CM1:::configStyle
     N_CM2:::configStyle
     N_CMC1_A:::configStyle
     N_CMC1_B:::fixedStyle
     N_CMC3:::configStyle
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
    style G3 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G4 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G5 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G6 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style SP_GAP1 fill:none,stroke:none,color:transparent
    style SP_GAP2 fill:none,stroke:none,color:transparent
    style INFO fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G2 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style STAGE1 fill:none,stroke:none,color:transparent
    style STAGE3 fill:none,stroke:none,color:transparent
    style STAGE4 fill:none,stroke:none,color:transparent
    style STAGE5 fill:none,stroke:none,color:transparent