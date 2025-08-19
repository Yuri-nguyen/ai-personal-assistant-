# ai-personal-assistant-
MeoMeo
import sqlite3
from datetime import datetime
from ai_analysis import analyze_user_data, suggest_growth_path
from ai_chat import chat_with_user, external_learn

DB_PATH = "user_data.db"

def init_db():
    conn = sqlite3.connect(DB_PATH)
    c = conn.cursor()
    c.execute("""
        CREATE TABLE IF NOT EXISTS chats (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT,
            message TEXT,
            ai_response TEXT
        )
    """)
    conn.commit()
    conn.close()

def save_chat(message, ai_response):
    conn = sqlite3.connect(DB_PATH)
    c = conn.cursor()
    c.execute("INSERT INTO chats (timestamp, message, ai_response) VALUES (?, ?, ?)",
              (datetime.now().isoformat(), message, ai_response))
    conn.commit()
    conn.close()

def monthly_review():
    conn = sqlite3.connect(DB_PATH)
    c = conn.cursor()
    month = datetime.now().month
    year = datetime.now().year
    c.execute("""
        SELECT message, ai_response FROM chats
        WHERE strftime('%m', timestamp) = ? AND strftime('%Y', timestamp) = ?
    """, (f"{month:02d}", f"{year}"))
    chats = c.fetchall()
    conn.close()
    analysis = analyze_user_data(chats)
    strengths, weaknesses = analysis["strengths"], analysis["weaknesses"]
    growth_path = suggest_growth_path(strengths, weaknesses)
    return {
        "month": month,
        "year": year,
        "summary": analysis["summary"],
        "strengths": strengths,
        "weaknesses": weaknesses,
        "suggested_growth_path": growth_path
    }

def main():
    init_db()
    print("Personal AI Assistant is running...")
    while True:
        user_message = input("Bạn: ")
        ai_response = chat_with_user(user_message)
        save_chat(user_message, ai_response)
        print("AI:", ai_response)
        # AI chủ động học hỏi thêm về các chủ đề liên quan
        external_learn(user_message)
        # Nếu là đầu tháng, review tổng quan
        if datetime.now().day == 1:
            review = monthly_review()
            print(f"\n=== Tổng quan tháng {review['month']}/{review['year']} ===")
            print("Tóm tắt:", review["summary"])
            print("Điểm mạnh:", ', '.join(review["strengths"]))
            print("Điểm yếu:", ', '.join(review["weaknesses"]))
            print("Gợi ý lộ trình phát triển:", review["suggested_growth_path"])

if __name__ == "__main__":
    main()
