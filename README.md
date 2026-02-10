# Intern_project_task
import os
from flask import Flask, render_template, request
import plotly.express as px
import pandas as pd
import json
from transformers import pipeline
import pydub

app = Flask(__name__)
# Load sentiment model
sentiment_classifier = pipeline("sentiment-analysis", model="distilbert-base-uncased-finetuned-sst-2-english")

# Helper function to segment audio and analyze
def analyze_audio(file_path):
    # Load audio and convert to segments (e.g., 5-second chunks)
    audio = pydub.AudioSegment.from_file(file_path)
    chunk_length = 5000 # ms
    chunks = [audio[i:i+chunk_length] for i in range(0, len(audio), chunk_length)]
    
    results = []
    for i, chunk in enumerate(chunks):
        # In a real app, convert audio chunk to text here.
        # For simplicity, we assume text extraction or use raw sentiment on audio features.
        # Here we simulate emotion/sentiment for demonstration.
        
        # --- PLACEHOLDER FOR ACTUAL SPEECH-TO-TEXT ---
        # text = transcribe(chunk) 
        # emotion = sentiment_classifier(text)
        
        # Simulating sentiment score over time
        sentiment = "POSITIVE" if i % 2 == 0 else "NEGATIVE"
        time_str = f"{i*5//60}:{ (i*5)%60:02d}"
        
        results.append({
            "time": time_str,
            "emotion": sentiment,
            "score": 0.85 if sentiment == "POSITIVE" else 0.9
        })
    return results

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        file = request.files['audio_file']
        file_path = os.path.join("uploads", file.filename)
        file.save(file_path)
        
        data = analyze_audio(file_path)
        df = pd.DataFrame(data)
        
        # Create Plotly Chart
        fig = px.line(df, x="time", y="score", color="emotion", 
                      title="Voice Emotion Timeline",
                      markers=True)
        graph_json = json.dumps(fig, indent=4)
        
        return render_template('index.html', graphJSON=graph_json, data=data)
    
    return render_template('index.html', graphJSON=None, data=None)

if __name__ == '__main__':
    os.makedirs("uploads", exist_ok=True)
    app.run(debug=True)
