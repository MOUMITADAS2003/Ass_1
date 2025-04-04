# Ass_1
Assigment_1
from transformers import pipeline, Conversation

# Load the conversational pipeline
chat_pipeline = pipeline("conversational", model="microsoft/DialoGPT-medium")

def get_response(user_input: str) -> str:
    convo = Conversation(user_input)
    response = chat_pipeline(convo)
    return response.generated_responses[-1]
import sqlite3
from datetime import datetime

DB_NAME = "chatbot.db"

def init_db():
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS logs (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT,
            user_input TEXT,
            bot_response TEXT
        )
    ''')
    conn.commit()
    conn.close()

def log_interaction(user_input: str, bot_response: str):
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO logs (timestamp, user_input, bot_response)
        VALUES (?, ?, ?)
    ''', (datetime.now().isoformat(), user_input, bot_response))
    conn.commit()
    conn.close()
from fastapi import FastAPI
from pydantic import BaseModel
from chatbot import get_response
from database import init_db, log_interaction

app = FastAPI()
init_db()

class Message(BaseModel):
    user_input: str

@app.post("/chat")
async def chat(message: Message):
    user_input = message.user_input
    bot_response = get_response(user_input)
    log_interaction(user_input, bot_response)
    return {"response": bot_response}
