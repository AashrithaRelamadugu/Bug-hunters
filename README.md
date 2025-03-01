#code
import streamlit as st

# Function to generate a study plan based on student inputs
def generate_study_plan(subjects, hours, preferred_time, strengths, weaknesses, preferences, subject_contents):
    study_plan = {}
    
    # Loop through subjects and distribute study hours
    for subject, hour in zip(subjects, hours):
        # Prioritize weak areas by adding extra time
        if subject in weaknesses:
            hour += 1  # Add more time to weak subjects
        
        # Get content for the subject based on preferred time
        content = subject_contents.get(subject, {}).get(preferred_time, ["General Study"])
        
        # Create a mock study session for the subject
        study_plan[subject] = {
            'study_time': hour,
            'preferred_time': preferred_time,
            'study_method': preferences.get(subject, 'focused'),  # Default to 'focused' if not provided
            'content': content[:hour]  # Allocate topics based on study hours
        }

    return study_plan

# Streamlit UI with state management
def main():
    # Initialize session state variables
    if 'context' not in st.session_state:
        st.session_state.context = 'waiting'  # Initial state when waiting for input

    # Check for the current context and manage app flow accordingly
    if st.session_state.context == 'waiting':
        # Show form for student inputs
        st.title("AI Study Planner")
        st.subheader("Enter Your Study Details:")

        # Input for subjects
        subjects_input = st.text_input("Subjects (comma-separated):", "Math, Science, History")
        subjects = [s.strip() for s in subjects_input.split(",")]

        # Input for hours
        hours_input = st.text_input("Study Hours per Subject (comma-separated):", "3, 2, 1")
        hours = [int(h.strip()) for h in hours_input.split(",")]

        # Input for preferred study time
        preferred_time = st.selectbox("Preferred Study Time:", ["morning", "afternoon", "evening"])

        # Input for strengths
        strengths_input = st.text_input("Your Strengths (comma-separated):", "Math")
        strengths = [s.strip() for s in strengths_input.split(",")]

        # Input for weaknesses
        weaknesses_input = st.text_input("Your Weaknesses (comma-separated):", "History")
        weaknesses = [s.strip() for s in weaknesses_input.split(",")]

        # Input for study preferences (could be focused, interactive, etc.)
        preferences_input = st.text_area("Study Preferences per Subject (e.g., 'Math: focused, Science: interactive'):", 
                                          "Math: focused, Science: interactive")
        preferences = {}
        if preferences_input:
            for pref in preferences_input.split(","):
                subject, method = pref.split(":")
                preferences[subject.strip()] = method.strip()

        # Input for subject contents
        st.subheader("Enter Topics for Each Subject and Time:")
        subject_contents = {}
        for subject in subjects:
            st.write(f"**{subject}**")
            morning_topics = st.text_input(f"Topics for {subject} in the morning (comma-separated):", "Topic 1, Topic 2")
            afternoon_topics = st.text_input(f"Topics for {subject} in the afternoon (comma-separated):", "Topic 1, Topic 2")
            evening_topics = st.text_input(f"Topics for {subject} in the evening (comma-separated):", "Topic 1, Topic 2")
            
            subject_contents[subject] = {
                "morning": [t.strip() for t in morning_topics.split(",")],
                "afternoon": [t.strip() for t in afternoon_topics.split(",")],
                "evening": [t.strip() for t in evening_topics.split(",")]
            }

        # Button to trigger the generation of the study plan
        if st.button('Generate Study Plan'):
            # Save the context to 'running' state and store user inputs in session state
            st.session_state.context = 'running'
            st.session_state.subjects = subjects
            st.session_state.hours = hours
            st.session_state.preferred_time = preferred_time
            st.session_state.strengths = strengths
            st.session_state.weaknesses = weaknesses
            st.session_state.preferences = preferences
            st.session_state.subject_contents = subject_contents

            # Generate the study plan
            study_plan = generate_study_plan(subjects, hours, preferred_time, strengths, weaknesses, preferences, subject_contents)

            # Store the result in session state
            st.session_state.study_plan = study_plan

            # Display a message to indicate that the plan is being generated
            st.info("Generating your personalized study plan...")

            # Switch context to display results
            st.session_state.context = 'displaying'

    elif st.session_state.context == 'displaying':
        # Show the generated study plan
        st.title("Your Personalized Study Plan:")
        
        # Display the study plan in a more user-friendly format
        study_plan = st.session_state.study_plan
        for subject, details in study_plan.items():
            st.subheader(f"Subject: {subject}")
            st.write(f"- Study Time: {details['study_time']} hours")
            st.write(f"- Preferred Time: {details['preferred_time']}")
            st.write(f"- Study Method: {details['study_method']}")
            st.write("- Topics to Cover:")
            for i, topic in enumerate(details['content'], 1):
                st.write(f"  {i}. {topic}")
            st.write("---")

        # Provide an option to go back and edit the plan
        if st.button('Edit Plan'):
            # Reset context to 'waiting' to allow re-entry of inputs
            st.session_state.context = 'waiting'
            st.session_state.study_plan = None  # Clear the generated plan

if __name__ == "__main__":
    main()