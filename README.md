import streamlit as st
import cv2
import mediapipe as mp
import numpy as np

# --- 1. AI ANGLE CALCULATION LOGIC ---
def calculate_angle(a, b, c):
    a = np.array(a) # First point (e.g., Hip)
    b = np.array(b) # Mid point (e.g., Shoulder)
    c = np.array(c) # End point (e.g., Elbow)
    
    radians = np.arctan2(c[1]-b[1], c[0]-b[0]) - np.arctan2(a[1]-b[1], a[0]-b[0])
    angle = np.abs(radians*180.0/np.pi)
    if angle > 180.0:
        angle = 360-angle
    return angle

# --- 2. YOUR SPREADSHEET SCORING LOGIC ---
def get_recommendation(flex, abd, er):
    # Points based on your specific criteria
    f_pts = 2 if flex >= 160 else (1 if flex >= 140 else 0)
    a_pts = 2 if abd >= 170 else (1 if abd >= 135 else 0)
    e_pts = 2 if er >= 90 else (1 if er >= 70 else 0)
    
    total = f_pts + a_pts + e_pts
    
    if total >= 5:
        return total, "✅ SAFE TO HIT", "No action needed", "green"
    elif 3 <= total <= 4:
        return total, "⚠️ CAUTION", "Needs motivation/mobility drills", "orange"
    else:
        return total, "❌ DO NOT HIT", "Needs Rehab / Physical Therapy", "red"

# --- 3. STREAMLIT INTERFACE ---
st.title("AI Shoulder Form Check")
st.write("Stand sideways for Flexion or face forward for Abduction.")

# Placeholder for the webcam feed
frame_placeholder = st.empty()
result_placeholder = st.empty()

# Initialize MediaPipe
mp_pose = mp.solutions.pose
pose = mp_pose.Pose(min_detection_confidence=0.5, min_tracking_confidence=0.5)

cap = cv2.VideoCapture(0)

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    # Convert to RGB for MediaPipe
    image = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = pose.process(image)

    angle_to_show = 0
    
    if results.pose_landmarks:
        landmarks = results.pose_landmarks.landmark
        
        # Get coordinates for Shoulder Flexion (Hip-Shoulder-Elbow)
        shoulder = [landmarks[mp_pose.PoseLandmark.LEFT_SHOULDER.value].x, landmarks[mp_pose.PoseLandmark.LEFT_SHOULDER.value].y]
        hip = [landmarks[mp_pose.PoseLandmark.LEFT_HIP.value].x, landmarks[mp_pose.PoseLandmark.LEFT_HIP.value].y]
        elbow = [landmarks[mp_pose.PoseLandmark.LEFT_ELBOW.value].x, landmarks[mp_pose.PoseLandmark.LEFT_ELBOW.value].y]
        
        angle_to_show = calculate_angle(hip, shoulder, elbow)

        # Draw on image
        mp.solutions.drawing_utils.draw_landmarks(image, results.pose_landmarks, mp_pose.POSE_CONNECTIONS)

    # Display the Video
    frame_placeholder.image(image, channels="RGB")

    # Display Live Scoring (Assuming we are testing Flexion live)
    total, verdict, action, color = get_recommendation(angle_to_show, 175, 95) # Placeholders for Abd/ER
    
    result_placeholder.markdown(f"""
    ### Current Live Angle: {int(angle_to_show)}°
    ---
    **Status:** :{color}[{verdict}]  
    **Action:** {action}
    """)

    if st.button("Stop App"):
        break

cap.release()
