# Load the existing "Current and Target Profile" sheet to update with risk assessment data
df_profile = current_target_profile.copy()

# Define mappings based on DBIR insights and Encryption Policy
risk_priorities = {
    "GV.OC-01": "High",  # Governance - Cyber risk management importance
    "GV.OC-02": "High",  # Understanding internal/external risks (Aligns with DBIR threats)
    "PR.AC-01": "High",  # Access Control - Stolen credentials major attack vector
    "PR.DS-01": "High",  # Data Security - Encryption policy alignment (AES, RSA, ECC)
    "DE.AE-01": "High",  # Anomaly Detection - Ransomware & credential misuse detection
    "RS.RP-01": "High",  # Incident Response - Rapid response to breaches
    "RC.RP-01": "Medium", # Recovery - Ensuring encrypted backups for ransomware resilience
}

# Define a function to update the profile with risk-based priorities and maturity levels
def update_profile(df):
    for index, row in df.iterrows():
        csf_id = row["CSF Outcome (Function, Category, or Subcategory)"]
        
        # Assign risk priority if mapped
        if csf_id in risk_priorities:
            df.at[index, "Current Priority"] = risk_priorities[csf_id]
            df.at[index, "Target Priority"] = "High"  # Set all target priorities to High for security maturity
        
        # Assign maturity levels (Example: Stolen credentials protection should aim for Managed+)
        if csf_id in ["PR.AC-01", "PR.DS-01", "DE.AE-01"]:
            df.at[index, "Current Status"] = 2  # Developing: Some controls exist but inconsistently applied
            df.at[index, "Target CSF Tier"] = 4  # Managed: Actively monitored and adjusted controls
        
        if csf_id in ["GV.OC-01", "GV.OC-02", "RS.RP-01", "RC.RP-01"]:
            df.at[index, "Current Status"] = 3  # Defined: Standardized controls but limited monitoring
            df.at[index, "Target CSF Tier"] = 5  # Optimized: Fully integrated security processes
        
    return df

# Apply the updates
df_updated = update_profile(df_profile)

# Save the updated file
updated_file_path = "/mnt/data/CSF_2.0_Organizational_Profile_Updated.xlsx"
with pd.ExcelWriter(updated_file_path, engine="xlsxwriter") as writer:
    column_descriptions.to_excel(writer, sheet_name="Column Descriptions", index=False)
    df_updated.to_excel(writer, sheet_name="Current and Target Profile", index=False)

# Provide the updated file to the user
updated_file_path
