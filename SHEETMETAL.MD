flowchart LR
 subgraph G1["1. SOFTWARE INPUTS"]
    direction TB
        I1["Upload CAD Files<br>(1 Part File or 3 Tool Files: Male, Female, Part)"]
        I2["Select Tool Material<br>(material_specification)"]
  end
 subgraph STAGE1[" "]
    direction TB
        G1
        N_I2["⚙️ Configurable Parameter:<br>Tool Material Density &amp; Unit Cost per kg"]
  end
 subgraph G2["2. CAD EXTRACTED DATA"]
    direction TB
    subgraph G2_A["Option A: Single File Workflow (1 Part File)"]
        direction TB
        E_S1["Extract Part Dimensions &amp; Volume:<br>Part: L_part, B_part, H_part &amp; V_part"]
        E_S2["Derive Male &amp; Female Tools:<br>Tool L = int(L_part), Tool B = int(B_part), Tool H = 50mm<br>Tool Volume (mm3) = L × B × 50 × 0.7"]
        E_S1 --> E_S2
    end
    subgraph G2_B["Option B: Three Files Workflow (Female, Male, Part)"]
        direction TB
        E_M1["Extract Dimensions &amp; Volumes for All 3 Files"]
        E_M2["Classify by Volume Hierarchy:<br>1. Female Tool (Largest Volume)<br>2. Male Tool (Middle Volume)<br>3. Part (Smallest Volume)"]
        E_M1 --> E_M2
    end
  end
 subgraph G3["3. PARAMETERS & STOCK GEOMETRY"]
    direction TB
        P1["Determine Stock Allowances (δX, δY, δZ):<br>Size-dependent extra allowance for Male &amp; Female tools"]
        P2["Compute &amp; Round Tool Stock Sizes:<br>L_stk = round(L + δX, 10)<br>B_stk = round(B + δY, 10)<br>H_stk = round(H + δZ, 5)"]
        P3["Tool Stock Volumes (Male &amp; Female):<br>V_stock (mm3) = L_stk × B_stk × H_stk<br>V_stock (m3) = V_stock / 10^9"]
        P4["Material Removed Volume per Tool:<br>V_removed = V_stock - V_tool"]
  end
 subgraph STAGE3[" "]
    direction TB
        G3
        N_P1["🔒 Fixed Backend Rule:<br>Allowances (L,B ≤ 50: +20mm, ≤ 200: +30mm, ≤ 500: +50mm, &gt; 500: +100mm; H ≤ 200: +5mm, &gt; 200: +10mm)"]
        N_P2["🔒 Fixed Backend Rule:<br>Rounding L, B to nearest 10mm, H to nearest 5mm | Derived Tool H=50mm, Factor=0.7"]
  end
 subgraph G4["4. MACHINING & MANUFACTURING TIME"]
    direction TB
        T1["Match Tool Milling Parameters by L_tool:<br>Fetch Feed, Axial DOC, Radial WOC for Tool Material"]
        T2["Compute MRR (mm3/min):<br>MRR_rough = Feed_r × DOC_r × WOC_r<br>MRR_finish = Feed_f × DOC_f × WOC_f"]
        T3["Calculate Tool Machining Times (hrs):<br>T_rough = (V_rem × P_rough / 100) / MRR_rough / 60<br>T_finish = (V_rem × P_finish / 100) / MRR_finish / 60"]
        T4["Total Tool Machining Time (hrs):<br>T_machining_hr = T_rough + T_finish (Male &amp; Female)"]
        T5["Total Manufacturing Time per Tool (hrs):<br>T_manuf_hr = (T_machining_hr × 100) / P_manuf"]
  end
 subgraph STAGE4[" "]
    direction TB
        G4
        N_T1["⚙️ Configurable Parameter:<br>Feed Rates, Axial DOC, Radial WOC for Tooling"]
        N_T3["⚙️ Configurable Parameter:<br>Roughing % &amp; Finishing % Splits"]
        N_T5["⚙️ Configurable Parameter:<br>Manufacturing Time % Ratio"]
  end
 subgraph G5["5. MATERIAL, TOOLING & FORMING COST"]
    direction TB
        C_M1["Calculate Tool Stock Mass:<br>W_male = V_stk_male × Density<br>W_female = V_stk_female × Density"]
        C_M2["Tool Material Cost:<br>C_mat_tool = (W_male + W_female) × C_mat"]
        C_MC1["Match Machine Rates by Tool Size:<br>Filter 3-Axis machines where L,B,H_tool ≤ Machine Envelope"]
        C_MC2["Calculate Tool Manufacturing Cost:<br>C_mfg_tools = (T_manuf_male + T_manuf_female) × Hourly_Rate"]
        C_MC3["Tool Design Cost:<br>Match Part Size Category → C_design = Design_Hours × Rate_hr"]
        C_MC4["Part Forming Cost:<br>C_forming = (P_forming / 100) × (C_male_total + C_female_total)"]
  end
 subgraph STAGE5[" "]
    direction TB
        G5
        N_CM1["⚙️ Configurable Parameter:<br>Tool Material Density (rho) &amp; Cost per kg"]
        N_CMC1_A["⚙️ Configurable Parameter:<br>3-Axis Machine Hourly Rates &amp; Envelope Sizes (L×B×H)"]
        N_CMC1_B["🔒 Fixed Backend Rule:<br>Tool Size ≤ Machine Envelope &amp; Part Size Category Match"]
        N_CMC2["⚙️ Configurable Parameter:<br>Tool Design Hours &amp; Design Cost per Hour"]
        N_CMC3["⚙️ Configurable Parameter:<br>Part Forming Cost % &amp; Manufacturing Correction Factor"]
  end
 subgraph G6["6. TOTAL COST AGGREGATION"]
    direction TB
        TOT2["Tool Material Cost for Batch:<br>Total_Tool_Material = C_mat_tool × Q"]
        TOT3["Tool Manufacturing &amp; Design Cost:<br>C_tooling_mfg = (C_mfg_tools + C_design) × Q"]
        TOT5["Part Forming Cost for Batch:<br>Total_Forming_Cost = C_forming × Q"]
        TOT1["Total Cost Per Unit (All Components):<br>C_total_unit = (C_male + C_female + C_forming + C_design) × K_corr"]
        TOT4["Grand Total Sheet Metal Cost:<br>C_grand_total = C_total_unit × Q"]
  end
 subgraph INFO["💡 USER INTERACTIVE FEATURES (EDITABLE IN UI)"]
    direction TB
        U1["1. Batch Quantity (Q) [Default = 1]"]
        U2["2. Raw Stock Dimensions Override (Male &amp; Female L_stk, B_stk, H_stk)"]
        U3["3. File Mode Selection (Single Part STEP with auto-tooling or 3 STEP files)"]
  end
    I1 --> E_S1 & E_M1
    I2 --> C_M1
    I2 -.-> N_I2
    E_S2 & E_M2 --> P1
    E_S2 & E_M2 --> P4 & C_M1
    E_S1 & E_M2 --> C_MC3
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
    T5 --> C_MC2
    C_M1 --> C_M2
    C_MC1 --> C_MC2
    C_MC2 --> C_MC4
    C_MC3 --> TOT3
    C_MC4 --> TOT5
    C_M2 --> TOT2
    C_M1 -.-> N_CM1
    C_MC1 -.-> N_CMC1_A & N_CMC1_B
    C_MC3 -.-> N_CMC2
    C_MC4 -.-> N_CMC3
    TOT2 --> TOT1
    TOT3 --> TOT1
    TOT5 --> TOT1
    TOT1 --> TOT4
    G6 ~~~ SP_GAP1["&nbsp;"]
    SP_GAP1 ~~~ SP_GAP2["&nbsp;"]
    SP_GAP2 ~~~ INFO

     I1:::calcStyle
     I2:::calcStyle
     N_I2:::configStyle
     E_S1:::calcStyle
     E_S2:::calcStyle
     E_M1:::calcStyle
     E_M2:::calcStyle
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
     C_MC4:::calcStyle
     N_CM1:::configStyle
     N_CMC1_A:::configStyle
     N_CMC1_B:::fixedStyle
     N_CMC2:::configStyle
     N_CMC3:::configStyle
     TOT2:::calcStyle
     TOT3:::calcStyle
     TOT5:::calcStyle
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
    style G2_A fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G2_B fill:#ffffff,stroke:#cbd5e1,stroke-width:1px,color:#0f172a
    style G3 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G4 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G5 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style G6 fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style SP_GAP1 fill:none,stroke:none,color:transparent
    style SP_GAP2 fill:none,stroke:none,color:transparent
    style INFO fill:#ffffff,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a
    style STAGE1 fill:none,stroke:none,color:transparent
    style STAGE3 fill:none,stroke:none,color:transparent
    style STAGE4 fill:none,stroke:none,color:transparent
    style STAGE5 fill:none,stroke:none,color:transparent
