# News-Summarization-and-Text-to-Speech-Application

import streamlit as st
import requests
from bs4 import BeautifulSoup
import random
import json



def fetch_news(query):
    search_url = f"https://news.google.com/search?q={query.replace(' ', '+')}&hl=en"
    response = requests.get(search_url, headers={"User-Agent": "Mozilla/5.0"})
    soup = BeautifulSoup(response.text, 'html.parser')
    
    news_links = []
    for a in soup.find_all('a', href=True):
        if './articles/' in a['href']:
            full_url = "https://news.google.com" + a['href'][1:]
            news_links.append(full_url)
        if len(news_links) >= 5:
            break
    return news_links



def get_content(news_url):
    try:
        res = requests.get(news_url, headers={"User-Agent": "Mozilla/5.0"})
        soup = BeautifulSoup(res.text, 'html.parser')
        headline = soup.title.string if soup.title else "No title"
        
        paragraphs = soup.find_all('p')
        content = ' '.join([p.get_text() for p in paragraphs[:5]])
        summary = content[:200] if content else "No content available"
        
        return {"headline": headline, "summary": summary, "link": news_url}
    except:
        return None



def analyze_sentiment(text):
    positive_words = ["good", "great", "excellent", "positive", "happy"]
    negative_words = ["bad", "poor", "terrible", "negative", "sad"]
    
    pos_count = sum(text.lower().count(word) for word in positive_words)
    neg_count = sum(text.lower().count(word) for word in negative_words)
    
    if pos_count > neg_count:
        return "Positive"
    elif neg_count > pos_count:
        return "Negative"
    else:
        return "Neutral"



def translate_to_hindi(text):
    translation_dict = {
        "Positive": "सकारात्मक",
        "Negative": "नकारात्मक",
        "Neutral": "तटस्थ"
    }
    return translation_dict.get(text, text)



def run_app():
    st.title("Company News & Sentiment Analysis")
    company_name = st.text_input("Enter Organization Name:")
    
    if st.button("Fetch Insights"):
        articles = fetch_news(company_name)
        results = []
        sentiment_list = []
        
        for article in articles:
            details = get_content(article)
            if details:
                sentiment = analyze_sentiment(details['summary'])
                sentiment_list.append(sentiment)
                details['sentiment'] = sentiment
                results.append(details)
        
        summary = {"Positive": sentiment_list.count("Positive"),
                   "Negative": sentiment_list.count("Negative"),
                   "Neutral": sentiment_list.count("Neutral")}
        
        st.text("Sentiment Overview:")
        st.json(summary)
        
        st.text("Hindi Sentiment:")
        hindi_summary = {translate_to_hindi(key): val for key, val in summary.items()}
        st.json(hindi_summary)

if __name__ == "__main__":
    run_app()

