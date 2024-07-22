from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer 
from tkinter import *
import tkinter.messagebox as messagebox
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import speech_recognition as sr
import pyttsx3

# Function for clearing all fields
def clearAll():
    negativeField.delete(0, END)
    neutralField.delete(0, END)
    positiveField.delete(0, END)
    overallField.delete(0, END)
    textArea.delete(1.0, END)
    # Clear image display if exists
    if 'canvas' in globals():
        canvas.get_tk_widget().destroy()

# Function to check if microphone is available
def check_microphone():
    try:
        recognizer = sr.Recognizer()
        with sr.Microphone() as source:
            print("Microphone is available.")
        messagebox.showinfo("Microphone Status", "Microphone is available.")
    except sr.RequestError:
        print("No microphone detected or unavailable.")
        messagebox.showerror("Microphone Status", "No microphone detected or unavailable.")
    except Exception as e:
        print(f"Error: {str(e)}")
        messagebox.showerror("Microphone Status", f"Error: {str(e)}")

# Function to detect sentiment from text
def detect_sentiment_text():
    sentence = textArea.get("1.0", "end").strip()
    if sentence:
        analyze_sentiment(sentence)
    else:
        messagebox.showwarning("Empty Input", "Please enter a sentence for sentiment analysis.")

# Function to detect sentiment from audio
def detect_sentiment_audio():
    try:
        recognizer = sr.Recognizer()
        with sr.Microphone() as source:
            print("Speak something...")
            audio = recognizer.listen(source)

        sentence = recognizer.recognize_google(audio)
        textArea.delete(1.0, END)
        textArea.insert(1.0, sentence)
        analyze_sentiment(sentence)
    except sr.UnknownValueError:
        print("Could not understand audio")
        textArea.delete(1.0, END)
        textArea.insert(1.0, "Could not understand audio")
    except sr.RequestError as e:
        print(f"Error fetching results; {e}")
        textArea.delete(1.0, END)
        textArea.insert(1.0, f"Error fetching results; {e}")

def analyze_sentiment(sentence):
    sid_obj = SentimentIntensityAnalyzer() 
    sentiment_dict = sid_obj.polarity_scores(sentence) 

    # Display sentiment percentages
    negativeField.delete(0, END)
    negativeField.insert(10, f"{sentiment_dict['neg']*100:.2f}% Negative")

    neutralField.delete(0, END)
    neutralField.insert(10, f"{sentiment_dict['neu']*100:.2f}% Neutral")

    positiveField.delete(0, END)
    positiveField.insert(10, f"{sentiment_dict['pos']*100:.2f}% Positive")

    # Determine overall sentiment
    if sentiment_dict['compound'] >= 0.05:
        overall_sentiment = "Positive"
        generate_image('positive', 'green')  # Change color to green
        speak_text(f"The sentiment is positive.")
    elif sentiment_dict['compound'] <= -0.05:
        overall_sentiment = "Negative"
        generate_image('negative', 'red')  # Change color to red
        speak_text(f"The sentiment is negative.")
    else:
        overall_sentiment = "Neutral"
        generate_image('neutral', 'orange')  # Change color to orange
        speak_text(f"The sentiment is neutral.")

    overallField.delete(0, END)
    overallField.insert(10, overall_sentiment)

    # Display tips based on sentiment
    display_sentiment_tips(sentiment_dict)

def display_sentiment_tips(sentiment_dict):
    # Example: Display tips based on sentiment analysis results
    if sentiment_dict['compound'] >= 0.5:
        messagebox.showinfo("Sentiment Analysis", "Looks like you're feeling very positive! Keep it up!")
    elif sentiment_dict['compound'] <= -0.5:
        messagebox.showinfo("Sentiment Analysis", "Seems like there's some negativity. Take a moment to relax and find positivity.")
    else:
        messagebox.showinfo("Sentiment Analysis", "Your sentiment is neutral. Keep a balanced perspective.")

def generate_image(sentiment, color):
    # Generate image based on sentiment and color
    fig, ax = plt.subplots(figsize=(2, 2))
    ax.axis('off')  # Hide axes
    if sentiment == 'positive':
        ax.text(0.5, 0.5, '😊', fontsize=100, ha='center', va='center', color=color)
    elif sentiment == 'negative':
        ax.text(0.5, 0.5, '😔', fontsize=100, ha='center', va='center', color=color)
    elif sentiment == 'neutral':
        ax.text(0.5, 0.5, '😐', fontsize=100, ha='center', va='center', color=color)
    else:
        ax.text(0.5, 0.5, '❓', fontsize=100, ha='center', va='center')  # Placeholder for unknown sentiment
    fig.tight_layout()

    # Convert plot to Tkinter canvas
    global canvas
    canvas = FigureCanvasTkAgg(fig, master=gui)
    canvas.draw()
    canvas.get_tk_widget().grid(row=15, column=2, pady=10)

# Function to speak text using pyttsx3
def speak_text(text):
    engine = pyttsx3.init()
    engine.say(text)
    engine.runAndWait()

# GUI setup
if __name__ == "__main__":
    gui = Tk()
    gui.config(background="light blue")  # Changed background color
    gui.title("Sentiment Analyzer")
    gui.geometry("600x700")  # Increased window size

    labelFont = ('Helvetica', 14, 'bold')  # Font style for labels
    textFont = ('Helvetica', 12)  # Font style for text area and entries

    # Labels and buttons with updated fonts and sizes
    enterText = Label(gui, text="How are you feeling today?", bg="light blue", font=labelFont)
    textArea = Text(gui, height=5, width=50, font=textFont)
    checkText = Button(gui, text="Check Sentiment (Text)", fg="Black", bg="Red", font=labelFont, command=detect_sentiment_text)
    checkAudio = Button(gui, text="Check Sentiment (Audio)", fg="Black", bg="Blue", font=labelFont, command=detect_sentiment_audio)
    checkMic = Button(gui, text="Check Microphone", fg="Black", bg="Yellow", font=labelFont, command=check_microphone)

    negative = Label(gui, text="Negative Sentiment:", bg="light blue", font=labelFont)
    neutral = Label(gui, text="Neutral Sentiment:", bg="light blue", font=labelFont)
    positive = Label(gui, text="Positive Sentiment:", bg="light blue", font=labelFont)
    overall = Label(gui, text="Overall Sentiment:", bg="light blue", font=labelFont)

    negativeField = Entry(gui, width=30, font=textFont)
    neutralField = Entry(gui, width=30, font=textFont)
    positiveField = Entry(gui, width=30, font=textFont)
    overallField = Entry(gui, width=30, font=textFont)

    clear = Button(gui, text="Clear", fg="Black", bg="Red", font=labelFont, command=clearAll)
    Exit = Button(gui, text="Exit", fg="Black", bg="Red", font=labelFont, command=gui.destroy)

    # Grid layout with updated row and column settings
    enterText.grid(row=0, column=0, columnspan=2, pady=10)
    textArea.grid(row=1, column=0, columnspan=2, padx=20, pady=10)
    checkText.grid(row=2, column=0, columnspan=2, pady=10)
    checkAudio.grid(row=3, column=0, columnspan=2, pady=10)
    checkMic.grid(row=4, column=0, columnspan=2, pady=10)
    negative.grid(row=5, column=0, sticky=W, pady=5, padx=20)
    neutral.grid(row=6, column=0, sticky=W, pady=5, padx=20)
    positive.grid(row=7, column=0, sticky=W, pady=5, padx=20)
    overall.grid(row=8, column=0, sticky=W, pady=5, padx=20)
    negativeField.grid(row=5, column=1, pady=5, padx=20)
    neutralField.grid(row=6, column=1, pady=5, padx=20)
    positiveField.grid(row=7, column=1, pady=5, padx=20)
    overallField.grid(row=8, column=1, pady=5, padx=20)
    clear.grid(row=9, column=0, pady=10)
    Exit.grid(row=9, column=1, pady=10)

    gui.mainloop()

