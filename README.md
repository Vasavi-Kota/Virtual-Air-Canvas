# Virtual-Air-Canvas





def run_canvas_app():
    st.title("Virtual Air Canvas")
    st.write("Use your hand gestures to draw and control the canvas. Say 'change color' or 'erase' to control.")

    drawing_points = []

    cap = cv2.VideoCapture(0)
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break

        frame = cv2.flip(frame, 1)

        # Process the frame to detect hands
        frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        result = hands.process(frame_rgb)

        if result.multi_hand_landmarks:
            for hand_landmarks in result.multi_hand_landmarks:
                for landmark in hand_landmarks.landmark:
                    h, w, _ = frame.shape
                    cx, cy = int(landmark.x * w), int(landmark.y * h)
                    if 0 < cx < w and 0 < cy < h:
                        drawing_points.append((cx, cy))

        # Draw on canvas
        draw_on_canvas(frame, drawing_points)

        # Show video frame in Streamlit
        st.image(frame, channels="BGR", use_column_width=True)

        # Use a unique key for the button
        if st.button("Listen for Voice Command", key="voice_command_button"):
            voice_commands()

        # Break on 'q' key press
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()
