# SmartCrack.ai App Setup (NCERT + PYQ Import + OpenAI + Supabase Backend + Test Series Upload)
# Main backend script to be run on Replit

# Ensure the required packages are listed in 'requirements.txt':
# openai
# supabase-py

import os

# Check for missing packages gracefully
missing_dependencies = []

try:
    import openai
except ImportError:
    openai = None
    missing_dependencies.append('openai')

try:
    from supabase import create_client, Client
except ImportError:
    create_client = Client = None
    missing_dependencies.append('supabase-py')

if missing_dependencies:
    raise ImportError(
        f"\n❌ Required packages are missing: {', '.join(missing_dependencies)}."
        "\nPlease install them via the Replit Packages tab or add them to requirements.txt."
    )

# ---- Configuration ----
SUPABASE_URL = os.getenv("SUPABASE_URL")
SUPABASE_KEY = os.getenv("SUPABASE_KEY")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")

if openai and not OPENAI_API_KEY:
    raise EnvironmentError("❌ Please set OPENAI_API_KEY in your Replit Secrets.")
if create_client and not all([SUPABASE_URL, SUPABASE_KEY]):
    raise EnvironmentError("❌ Please set SUPABASE_URL and SUPABASE_KEY in your Replit Secrets.")

if openai:
    openai.api_key = OPENAI_API_KEY
if create_client:
    supabase: Client = create_client(SUPABASE_URL, SUPABASE_KEY)

# ---- NCERT / PYQ Importer ----
def import_ncert_pyq_to_supabase(file_path, subject, exam_type):
    if not create_client:
        print("❌ Supabase not available. Skipping import.")
        return

    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
    except FileNotFoundError:
        print(f"❌ File not found: {file_path}")
        return

    chunks = [chunk.strip() for chunk in content.split('\n\n') if chunk.strip()]

    for chunk in chunks:
        response = supabase.table("study_material").insert({
            "subject": subject,
            "exam": exam_type,
            "content": chunk
        }).execute()
        if hasattr(response, 'error') and response.error:
            print(f"❌ Error inserting chunk: {response.error}")

    print(f"✅ Imported {len(chunks)} chunks from {file_path}.")

# ---- OpenAI-Powered MCQ Generator ----
def generate_mcqs(text):
    if not openai:
        return "❌ OpenAI API not available in this environment."

    prompt = f"Generate 3 MCQs with 4 options each and correct answers from this content:\n{text}"
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message['content']
    except Exception as e:
        return f"❌ Error generating MCQs: {e}"

# ---- Insert generated MCQs into Supabase ----
def save_mcqs_to_db(mcq_text, subject, exam_type):
    if not create_client:
        print("❌ Supabase not available. Skipping MCQ save.")
        return

    response = supabase.table("mcqs").insert({
        "subject": subject,
        "exam": exam_type,
        "mcq": mcq_text
    }).execute()
    if hasattr(response, 'error') and response.error:
        print(f"❌ Error saving MCQs: {response.error}")
    else:
        print("✅ MCQs saved to Supabase.")

# ---- Upload Test Series and Questions to Supabase ----
def import_test_series(data, exam_name):
    if not create_client:
        print("❌ Supabase not available.")
        return

    exam = supabase.table("exams").select("id").eq("name", exam_name).execute()
    if exam.data:
        exam_id = exam.data[0]["id"]
    else:
        exam_resp = supabase.table("exams").insert({"name": exam_name}).execute()
        exam_id = exam_resp.data[0]["id"]

    for ts in data:
        ts_resp = supabase.table("test_series").insert({
            "exam_id": exam_id,
            "title": ts["title"],
            "total_questions": len(ts["questions"]),
            "language": ts.get("language", ["English"]),
            "difficulty": ts.get("difficulty", "Medium"),
            "price": ts.get("price", 0)
        }).execute()
        ts_id = ts_resp.data[0]["id"]

        for q in ts["questions"]:
            supabase.table("questions").insert({
                "test_series_id": ts_id,
                "question": q["question"],
                "options": q["options"],
                "answer": q["answer"],
                "tags": q.get("tags", []),
                "difficulty": q.get("difficulty", "Medium")
            }).execute()

    print(f"✅ Imported {len(data)} test series for '{exam_name}'.")

# ---- Example Usage ----
if __name__ == "__main__":
    example_file = "ncert_history_class6.txt"
    if os.path.exists(example_file):
        import_ncert_pyq_to_supabase(example_file, "History", "UPSSSC")

        sample_text = "The Harappan civilization was the first urban civilization of the Indian subcontinent."
        mcqs = generate_mcqs(sample_text)
        print("\nGenerated MCQs:\n", mcqs)
        save_mcqs_to_db(mcqs, "History", "UPSSSC")
    else:
        print(f"❌ Example file '{example_file}' not found. Please upload it to the Replit workspace.")

    # Sample Test Series Import
    sample_test_series = [
        {
            "title": "SSC CGL Tier-I Mock Test #1",
            "difficulty": "Medium",
            "price": 0,
            "language": ["English", "Hindi"],
            "questions": [
                {
                    "question": "What is the value of √144?",
                    "options": ["10", "11", "12", "13"],
                    "answer": "12",
                    "tags": ["Maths", "Square Roots"],
                    "difficulty": "Easy"
                },
                
                {
                    "question": "What is the capital of France?",
                    "options": ["Berlin", "Madrid", "Paris", "Lisbon"],
                    "answer": "Paris",
                    "tags": ["Static GK", "Geography"]
                }
            ]
        }
    ]

    import_test_series(sample_test_series, "SSC CGL")
