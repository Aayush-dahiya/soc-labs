# Samba Spy - Static Java Code Analysis

This lab focused on reviewing a suspicious Java program provided as part of a SOC investigation challenge.  
The objective was to extract behavioral details, detect anti-analysis logic, and identify key IOCs.

**Skills Demonstrated:**
- Extracting and reviewing Java bytecode
- Identifying anti-virtualization checks
- Understanding execution flow from source
- Documenting IOCs for SOC detection rules

**Key Findings:**
1. **VM Check Method:** isRunningInVM
2. **Required System Language:** Italian
3. **Extraction Method:** extractLibs
4. **Extraction Path:** `C:\Users\Public\`
5. **Payload File Name:** Prodotto.png
6. **Execution Command:** `java -jar C:\Users\Public\Prodotto.png`
7. **Additional VM Check:** `wmic baseboard get manufacturer`
8. **VM Vendors Checked:** 4

**Investigation Date:** August 2025  
**Platform:** LetsDefend
