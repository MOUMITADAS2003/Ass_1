from transformers import pipeline

# Load sentiment-analysis pipeline
classifier = pipeline("sentiment-analysis")

def analyze_sentiment(text):
    result = classifier(text)[0]
    return {
        "label": result["label"],  # POSITIVE or NEGATIVE
        "score": round(result["score"], 3)
    }
from pymongo import MongoClient
from datetime import datetime

client = MongoClient("mongodb://localhost:27017/")
db = client["sentiment_db"]
collection = db["reviews"]

def log_review(text, sentiment):
    document = {
        "text": text,
        "sentiment": sentiment["label"],
        "score": sentiment["score"],
        "timestamp": datetime.utcnow()
    }
    collection.insert_one(document)

def get_sentiment_counts():
    return list(collection.aggregate([
        {"$group": {"_id": "$sentiment", "count": {"$sum": 1}}}
    ]))
import matplotlib.pyplot as plt
from database import get_sentiment_counts

def plot_sentiment_distribution():
    data = get_sentiment_counts()
    labels = [d["_id"] for d in data]
    counts = [d["count"] for d in data]

    plt.figure(figsize=(6, 4))
    plt.bar(labels, counts, color=["green", "red", "gray"])
    plt.title("Sentiment Distribution")
    plt.xlabel("Sentiment")
    plt.ylabel("Number of Reviews")
    plt.tight_layout()
    plt.savefig("sentiment_report.png")
from flask import Flask, request, jsonify, send_file
from sentiment import analyze_sentiment
from database import log_review
from visualize import plot_sentiment_distribution

app = Flask(__name__)

@app.route("/analyze", methods=["POST"])
def analyze():
    data = request.json
    text = data.get("text")
    result = analyze_sentiment(text)
    log_review(text, result)
    return jsonify(result)

@app.route("/report", methods=["GET"])
def report():
    plot_sentiment_distribution()
    return send_file("sentiment_report.png", mimetype='image/png')

if __name__ == "__main__":
    app.run(debug=True)
