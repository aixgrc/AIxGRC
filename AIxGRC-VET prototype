# =============================================================================
# IMPORTS
# =============================================================================

import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from IPython.display import display, HTML

# =============================================================================
# 1. MOCK DATA
# =============================================================================

departments = pd.DataFrame({
    "Department":          ["Emergency Dept", "Pharmacy", "Nursing", "Administration"],
    "Staff":               [312, 86, 540, 128],
    "Readiness Score":     [54, 71, 69, 88],
    "Training Completion": [68, 84, 79, 96],
    "Simulation Pass Rate":[41, 73, 70, 91],
    "Behavior Score":      [58, 68, 66, 85],
    "Risk Level":          ["High Risk", "Mitigation", "Mitigation", "Ready"],
})

simulations = pd.DataFrame({
    "Scenario": [
        "Renal dosing validation",
        "Medication reconciliation",
        "AI triage alert review",
        "Escalation of unsafe AI recommendation",
        "Clinical documentation workflow",
    ],
    "Department":          ["Emergency Dept", "Pharmacy", "Emergency Dept",
                            "Emergency Dept", "Nursing"],
    "Pass Rate":           [41, 73, 62, 46, 70],
    "Unsafe Override Rate":[38, 22, 25, 41, 18],
    "Escalation Accuracy": [46, 61, 58, 39, 72],
})

# Risk level derived from pass rate  --  same thresholds as the WRI logic
simulations["Risk Level"] = simulations["Pass Rate"].apply(
    lambda p: "High Risk" if p < 60 else ("Mitigation" if p < 75 else "Ready")
)

interventions = pd.DataFrame({
    "Intervention": [
        "Renal Dosing Retraining",
        "Escalation Procedure Workshop",
        "Clinical Workflow Simulation",
        "Documentation Standards Refresher",
        "Medication Reconciliation Drill",
    ],
    "Department": ["Emergency Dept", "Pharmacy", "Nursing",
                   "Emergency Dept", "Pharmacy"],
    "Owner":      ["Clinical Education Lead", "Pharmacy Lead", "Nursing Informatics",
                   "Clinical Education Lead", "Pharmacy Lead"],
    "Status":     ["In Progress", "Scheduled", "In Progress", "Overdue", "Complete"],
    "Completion": [62, 20, 45, 15, 100],
})

risk_register = pd.DataFrame({
    "Risk": [
        "Medication dosing simulation failures",
        "Escalation workflow confusion",
        "AI output verification below target",
        "Documentation completeness gap",
        "Simulation environment data lag",
    ],
    "Department": ["Emergency Dept", "Emergency / Pharmacy", "Nursing",
                   "Emergency Dept", "Platform-wide"],
    "Severity":   ["High", "High", "Medium", "Medium", "Low"],
    "Status":     ["Mitigating", "Mitigating", "Mitigating",
                   "Overdue Action", "Monitoring"],
})

compliance = pd.DataFrame({
    "Compliance Area": [
        "HIPAA Security Rule",
        "Joint Commission AI Use Standard",
        "Internal AI Governance Policy v3.2",
        "State Health AI Disclosure Rule",
    ],
    "Status": ["Compliant", "In Remediation", "Compliant", "Not Yet Due"],
    "Description": [
        "Workforce training requirement",
        "Pre-deployment competency verification",
        "Mandatory go/no-go checkpoint",
        "Patient notification requirement",
    ],
})

# =============================================================================
# 2. READINESS LOGIC
# =============================================================================

# Staff-weighted WRI  --  Emergency's 312 staff carry more weight than Pharmacy's 86,
# reflecting their operational impact on a healthcare system deployment
overall_wri = round(
    (departments["Readiness Score"] * departments["Staff"]).sum()
    / departments["Staff"].sum()
)

# Deployment risk: staff-weighted composite of readiness gap + simulation gap
# + behavior gap. Simulation failures are weighted highest (45%) because unsafe
# AI override behavior is the most direct patient-safety risk.
total_staff = departments["Staff"].sum()
readiness_gap = round(
    ((100 - departments["Readiness Score"]) * departments["Staff"]).sum() / total_staff
)
sim_gap = round(
    ((100 - departments["Simulation Pass Rate"]) * departments["Staff"]).sum() / total_staff
)
behavior_gap = round(
    ((100 - departments["Behavior Score"]) * departments["Staff"]).sum() / total_staff
)
deployment_risk = round(
    readiness_gap * 0.35 + sim_gap * 0.45 + behavior_gap * 0.20
)

simulation_pass_rate = round(
    (departments["Simulation Pass Rate"] * departments["Staff"]).sum() / total_staff
)

open_risks      = 7
days_to_go_live = 12

# Go/No-Go thresholds
if overall_wri >= 80:
    recommendation = "GO"
    rec_color      = "#15824F"
    rec_bg         = "#E3F7EC"
elif overall_wri >= 65:
    recommendation = "GO WITH MITIGATION"
    rec_color      = "#C97F12"
    rec_bg         = "#FCF1DE"
else:
    recommendation = "NO-GO"
    rec_color      = "#D64545"
    rec_bg         = "#FCE9E9"

# Deployment risk label  --  thresholds calibrated to this dataset
# where a staff-weighted composite of ~35 represents genuine moderate risk
if deployment_risk >= 55:
    risk_label      = "High Risk"
    risk_bar_color  = "#D64545"
elif deployment_risk >= 32:
    risk_label      = "Moderate"
    risk_bar_color  = "#F2A93B"
else:
    risk_label      = "Low Risk"
    risk_bar_color  = "#1E9E6C"

print("=" * 48)
print("AIxGRC-VET  --  Readiness Summary")
print("=" * 48)
print(f"  Workforce Readiness Index:  {overall_wri}/100")
print(f"  Deployment Risk Score:      {deployment_risk}/100  ({risk_label})")
print(f"  Simulation Pass Rate:       {simulation_pass_rate}%  (staff-weighted)")
print(f"  AI Recommendation:          {recommendation}")
print(f"  Open Risks:                 {open_risks}")
print(f"  Days to Go-Live:            {days_to_go_live}")
print("=" * 48)

# =============================================================================
# 3. EXECUTIVE DASHBOARD
# =============================================================================

# --- WRI Gauge ---
fig_wri = go.Figure(go.Indicator(
    mode="gauge+number",
    value=overall_wri,
    title={"text": "Workforce Readiness Index (WRI)", "font": {"size": 16}},
    number={"suffix": "/100", "font": {"size": 36}},
    gauge={
        "axis": {"range": [0, 100]},
        "bar":  {"color": rec_color, "thickness": 0.22},
        "steps": [
            {"range": [0,  65],  "color": "#FCE9E9"},
            {"range": [65, 80],  "color": "#FCF1DE"},
            {"range": [80, 100], "color": "#E3F7EC"},
        ],
        "threshold": {
            "line":      {"color": "#0B1E3D", "width": 3},
            "thickness": 0.85,
            "value":     65,
        },
    },
))
fig_wri.update_layout(height=300, margin=dict(t=60, b=20, l=30, r=30))
fig_wri.show()

# --- Deployment Risk Gauge ---
fig_risk = go.Figure(go.Indicator(
    mode="gauge+number",
    value=deployment_risk,
    title={"text": f"Deployment Risk Score  --  {risk_label}", "font": {"size": 16}},
    number={"suffix": "/100", "font": {"size": 36}},
    gauge={
        "axis": {"range": [0, 100]},
        "bar":  {"color": risk_bar_color, "thickness": 0.22},
        "steps": [
            {"range": [0,  32], "color": "#E3F7EC"},
            {"range": [32, 55], "color": "#FCF1DE"},
            {"range": [55, 100], "color": "#FCE9E9"},
        ],
    },
))
fig_risk.update_layout(height=300, margin=dict(t=60, b=20, l=30, r=30))
fig_risk.show()

# --- Department Readiness Bar Chart ---
color_map = {
    "High Risk":  "#D64545",
    "Mitigation": "#F2A93B",
    "Ready":      "#1E9E6C",
}
fig_dept = px.bar(
    departments,
    x="Department",
    y="Readiness Score",
    color="Risk Level",
    color_discrete_map=color_map,
    text="Readiness Score",
    title="Department Readiness Scores",
)
fig_dept.add_hline(
    y=65,
    line_dash="dash",
    line_color="#0B1E3D",
    annotation_text="Deployment threshold (65)",
    annotation_font={"size": 11},
)
fig_dept.update_layout(yaxis_range=[0, 100], height=380)
fig_dept.show()

# =============================================================================
# 4. READINESS ASSESSMENT
# =============================================================================

print("\n-- Readiness Assessment " + "-" * 36)
display(departments)

heatmap_data = departments.set_index("Department")[[
    "Training Completion",
    "Simulation Pass Rate",
    "Behavior Score",
    "Readiness Score",
]]
fig_heatmap = px.imshow(
    heatmap_data,
    text_auto=True,
    color_continuous_scale="RdYlGn",
    zmin=0, zmax=100,
    title="Readiness Heatmap  --  Score Band by Department",
)
fig_heatmap.update_layout(height=320)
fig_heatmap.show()

# =============================================================================
# 5. BEHAVIORAL SIMULATION CENTER
# =============================================================================

print("\n-- Behavioral Simulation Center " + "-" * 28)
display(simulations)

# Simulation Pass Rates
fig_sim = px.bar(
    simulations,
    x="Scenario",
    y="Pass Rate",
    color="Risk Level",
    color_discrete_map=color_map,
    text="Pass Rate",
    title="Behavioral Simulation Pass Rates",
)
fig_sim.add_hline(
    y=75,
    line_dash="dash",
    line_color="#0B1E3D",
    annotation_text="Target: 75%",
    annotation_font={"size": 11},
)
fig_sim.update_layout(yaxis_range=[0, 100], xaxis_tickangle=-30, height=420)
fig_sim.show()

# Unsafe Override Rate
fig_override = px.bar(
    simulations,
    x="Scenario",
    y="Unsafe Override Rate",
    color="Risk Level",
    color_discrete_map=color_map,
    text="Unsafe Override Rate",
    title="Unsafe AI Override Rate by Simulation Scenario",
)
fig_override.update_layout(yaxis_range=[0, 100], xaxis_tickangle=-30, height=420)
fig_override.show()

# =============================================================================
# 6. INTERVENTION MANAGEMENT
# =============================================================================

print("\n-- Intervention Management " + "-" * 34)
display(interventions)

status_color_map = {
    "Complete":    "#1E9E6C",
    "In Progress": "#F2A93B",
    "Scheduled":   "#0E7C7B",
    "Overdue":     "#D64545",
}
fig_interventions = px.bar(
    interventions,
    x="Intervention",
    y="Completion",
    color="Status",
    color_discrete_map=status_color_map,
    text="Completion",
    title="Intervention Completion Progress",
)
fig_interventions.update_layout(yaxis_range=[0, 100], xaxis_tickangle=-30, height=420)
fig_interventions.show()

# =============================================================================
# 7. DEPLOYMENT DECISION CENTER
# =============================================================================

emg_score = int(
    departments.loc[departments["Department"] == "Emergency Dept",
                    "Readiness Score"].values[0]
)

decision_html = f"""
<div style="font-family:sans-serif; max-width:700px; margin:8px 0;">

  <div style="background:{rec_bg}; border:2px solid {rec_color};
              border-radius:12px; padding:24px 28px; margin-bottom:16px;">
    <div style="font-size:11px; font-weight:700; text-transform:uppercase;
                letter-spacing:0.08em; color:#6B7280; margin-bottom:8px;">
      AI Recommendation
    </div>
    <div style="font-size:36px; font-weight:800; color:{rec_color};
                letter-spacing:-0.02em; margin-bottom:12px;">
      {recommendation}
    </div>
    <div style="font-size:13px; color:#3B4250; line-height:1.6;">
      Emergency Department readiness ({emg_score}/100) falls below the 65-point
      governance threshold. Deployment may proceed in Pharmacy, Nursing, and
      Administration. Emergency Dept requires completion of 2 active interventions
      before go-live.
    </div>
  </div>

  <div style="display:flex; gap:14px; margin-bottom:16px; flex-wrap:wrap;">
    <div style="flex:1; min-width:130px; background:white; border:1px solid #E5EAF0;
                border-radius:10px; padding:14px 16px;">
      <div style="font-size:10px; font-weight:700; text-transform:uppercase;
                  letter-spacing:0.04em; color:#6B7280; margin-bottom:6px;">WRI</div>
      <div style="font-size:26px; font-weight:800; color:#0B1E3D;">
        {overall_wri}<span style="font-size:13px; color:#6B7280;">/100</span></div>
    </div>
    <div style="flex:1; min-width:130px; background:white; border:1px solid #E5EAF0;
                border-radius:10px; padding:14px 16px;">
      <div style="font-size:10px; font-weight:700; text-transform:uppercase;
                  letter-spacing:0.04em; color:#6B7280; margin-bottom:6px;">
        Deployment Risk</div>
      <div style="font-size:26px; font-weight:800; color:#0B1E3D;">
        {deployment_risk}<span style="font-size:13px; color:#6B7280;">/100  --  {risk_label}</span></div>
    </div>
    <div style="flex:1; min-width:130px; background:white; border:1px solid #E5EAF0;
                border-radius:10px; padding:14px 16px;">
      <div style="font-size:10px; font-weight:700; text-transform:uppercase;
                  letter-spacing:0.04em; color:#6B7280; margin-bottom:6px;">Open Risks</div>
      <div style="font-size:26px; font-weight:800; color:#0B1E3D;">{open_risks}</div>
    </div>
    <div style="flex:1; min-width:130px; background:white; border:1px solid #E5EAF0;
                border-radius:10px; padding:14px 16px;">
      <div style="font-size:10px; font-weight:700; text-transform:uppercase;
                  letter-spacing:0.04em; color:#6B7280; margin-bottom:6px;">
        Days to Go-Live</div>
      <div style="font-size:26px; font-weight:800; color:#0B1E3D;">{days_to_go_live}</div>
    </div>
  </div>

  <div style="background:white; border:1px solid #E5EAF0; border-radius:12px;
              padding:18px 20px; margin-bottom:16px;">
    <div style="font-size:13px; font-weight:700; color:#0B1E3D; margin-bottom:12px;">
      &#9672; Why the AI Recommended This
    </div>
    <div style="font-size:13px; color:#3B4250; line-height:2.0;">
      <span style="font-weight:700; color:#D64545;">1. </span>
        Emergency Dept readiness ({emg_score}/100) is below the 65-point threshold<br>
      <span style="font-weight:700; color:#D64545;">2. </span>
        Medication dosing simulation failures  --  41% pass rate vs 75% target<br>
      <span style="font-weight:700; color:#D64545;">3. </span>
        Escalation workflow confusion  --  23 help desk tickets flagged<br>
      <span style="font-weight:700; color:#F2A93B;">4. </span>
        Unsafe AI override behavior remains above acceptable levels
    </div>
  </div>

  <div style="background:#E3F5F4; border-radius:10px; padding:14px 18px;">
    <div style="font-size:11px; font-weight:700; color:#0E7C7B; margin-bottom:8px;
                text-transform:uppercase; letter-spacing:0.06em;">Evidence Sources</div>
    <div style="font-size:12.5px; color:#3B4250;">
      &#128196; Training records &nbsp;&nbsp;
      &#9200; Behavioral simulation outcomes &nbsp;&nbsp;
      &#128172; Help desk tickets &nbsp;&nbsp;
      &#9888; Risk assessments
    </div>
  </div>

</div>
"""
display(HTML(decision_html))

# =============================================================================
# 8. REASSESSMENT
# =============================================================================

before_wri = 54
delta      = overall_wri - before_wri
pct_gain   = round(delta / before_wri * 100)

reassessment = pd.DataFrame({
    "Stage": ["Before Interventions", "After Interventions"],
    "WRI":   [before_wri, overall_wri],
})
fig_reassessment = px.bar(
    reassessment,
    x="Stage",
    y="WRI",
    text="WRI",
    color="Stage",
    color_discrete_map={
        "Before Interventions": "#D64545",
        "After Interventions":  "#0E7C7B",
    },
    title=f"Readiness Improvement After Interventions (+{delta} pts, +{pct_gain}%)",
)
fig_reassessment.update_layout(yaxis_range=[0, 100], showlegend=False, height=360)
fig_reassessment.show()

# WRI Trend Line
trend = pd.DataFrame({
    "Date": ["Mar 10", "Mar 24", "Apr 7", "Apr 21", "May 19", "Jun 21"],
    "WRI":  [41, 47, 52, 54, 61, 68],
})
fig_trend = px.line(
    trend,
    x="Date",
    y="WRI",
    markers=True,
    title="WRI Trend Across Assessment Cycles",
    line_shape="spline",
)
fig_trend.add_hline(
    y=65,
    line_dash="dash",
    line_color="#F2A93B",
    annotation_text="Deployment threshold (65)",
    annotation_font={"size": 11},
)
fig_trend.update_traces(line_color="#0E7C7B", marker_color="#0E7C7B", marker_size=8)
fig_trend.update_layout(yaxis_range=[0, 100], height=360)
fig_trend.show()

# =============================================================================
# 9. AUDIT & COMPLIANCE
# =============================================================================

print("\n-- Risk Register " + "-" * 42)
display(risk_register)

print("\n-- Compliance Status " + "-" * 38)
display(compliance)
