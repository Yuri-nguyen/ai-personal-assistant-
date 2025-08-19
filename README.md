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
from transformers import pipeline

# Chatbot AI
chatbot = pipeline("conversational", model="microsoft/DialoGPT-medium")

def chat_with_user(message):
    resp = chatbot(message)
    return resp[0]['generated_text']

def external_learn(topic):
    # TODO: Tích hợp search web/crawl/LLM để tự động học hỏi
    print(f"AI đang học thêm về chủ đề: {topic}")
    # Ví dụ: Gọi API Bing/GPT, crawl wikipedia, lưu kết quả vào database/phân tích
def analyze_user_data(chats):
    # Phân tích lịch sử chat: tìm điểm mạnh/yếu, tổng quan
    # Đây là ví dụ đơn giản, có thể dùng NLP nâng cao hơn
    strengths, weaknesses = [], []
    summary = f"Bạn đã có {len(chats)} cuộc trò chuyện trong tháng."
    for msg, ai_resp in chats:
        if "giỏi" in ai_resp or "xuất sắc" in ai_resp:
            strengths.append(msg)
        if "cần cải thiện" in ai_resp or "yếu" in ai_resp:
            weaknesses.append(msg)
    return {
        "summary": summary,
        "strengths": strengths,
        "weaknesses": weaknesses
    }

def suggest_growth_path(strengths, weaknesses):
    # Gợi ý lộ trình dựa vào phân tích (ví dụ)
    if weaknesses:
        return f"Hãy tập trung cải thiện các điểm sau: {', '.join(weaknesses)}."
    if strengths:
        return f"Tiếp tục phát huy điểm mạnh: {', '.join(strengths)}."
    return "Chưa đủ dữ liệu để đưa ra lộ trình, hãy trò chuyện thêm với AI."
