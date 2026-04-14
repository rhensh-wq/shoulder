# shoulder
shoulder eval 
def shoulder_assessment(flexion_angle, abduction_angle, er_angle):
    """
    Calculates shoulder health based on spreadsheet criteria.
    Inputs: Angles in degrees.
    """
    
    # 1. OH Flexion ROM Scoring
    if flexion_angle >= 160:
        flex_score, flex_color = 2, "Green (Normal)"
    elif 140 <= flexion_angle < 160:
        flex_score, flex_color = 1, "Yellow (Moderate)"
    else:
        flex_score, flex_color = 0, "Red (Poor)"

    # 2. Shoulder Abduction Scoring
    if abduction_angle >= 170:
        abd_score, abd_color = 2, "Green (Normal)"
    elif 135 <= abduction_angle < 170:
        abd_score, abd_color = 1, "Yellow (Moderate)"
    else:
        abd_score, abd_color = 0, "Red (Poor)"

    # 3. 90/90 External Rotation Scoring
    if er_angle >= 90:
        er_score, er_color = 2, "Green (Normal)"
    elif 70 <= er_angle < 90:
        er_score, er_color = 1, "Yellow (Moderate)"
    else:
        er_score, er_color = 0, "Red (Poor)"

    # Calculate Total
    total_score = flex_score + abd_score + er_score
    
    # Determine "Safe to Hit" and "Action Needed"
    if total_score >= 5:
        safe_to_hit = "Yes"
        action_needed = "No action needed"
        risk_level = "Low"
    elif 3 <= total_score <= 4:
        safe_to_hit = "Caution"
        action_needed = "Needs mobility/motivation drills"
        risk_level = "Moderate"
    else:
        safe_to_hit = "No"
        action_needed = "Needs Rehab / Physical Therapy"
        risk_level = "High"

    return {
        "Total Score": total_score,
        "Risk Level": risk_level,
        "Safe to Hit": safe_to_hit,
        "Action Needed": action_needed,
        "Details": {
            "Flexion": flex_color,
            "Abduction": abd_color,
            "ER": er_color
        }
    }

# --- TEST IT OUT ---
# Example: An athlete with limited flexion and abduction
athlete_results = shoulder_assessment(flexion_angle=145, abduction_angle=120, er_angle=95)

print(f"TOTAL SCORE: {athlete_results['Total Score']}/6")
print(f"SAFE TO HIT: {athlete_results['Safe to Hit']}")
print(f"ACTION NEEDED: {athlete_results['Action Needed']}")
