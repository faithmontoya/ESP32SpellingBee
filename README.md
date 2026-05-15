#-----ESP32 Spelling Bee Game-----

An interactive spelling-bee game running on an ESP32 with MicroPython. The ESP32 hosts a webpage allowing players to submit spelling answers from any browser on the same Wi-Fi network. Hardware feedback (green/red LEDs and a buzzer) provides instant “correct” or “incorrect” cues.  

Challenge your spelling skills with common words that are surprisingly tricky to spell! 

---

#Table of Contents
1. Overview
2. Features
3. Hardware Requirements
4. Wiring
5. Wi-Fi Configuration
6. Webpage
7. Button — Triggering New Words
8. Customizing the Word List

---

#Overview 
-	Player presses button
-	ESP32 picks a random word 
-	PC speaks the word 
-	Player types answer in browser 
-	ESP32 checks spelling
   - Correct → Green LED + happy buzzer melody
   - Incorrect → Red LED + dull buzzer buzz

The ESP32 joins the home network and serves a single-page HTML form where the player submits their answer. All game logic runs entirely on the ESP32 in MicroPython.

---

#Features

- Browser-based (works on any phone, tablet, or PC)
- Random word selection from a built-in word list
- Instant visual feedback via green and red LEDs
- Buzzer tones to distinguish correct from incorrect answers
- Hardware button to request the next word without touching the browser

---

#Hardware Requirements

-	Microcontroller – ESP32
-	Breadboard 
-	USB cable (Micro-USB)
-	Jumper wires 
-	Tactile push-button 
-	Passive buzzer 
-	Green LED 
-	Red LED 
-	2x 330 Ω Resistors - one per LED

---

#Wiring

- GND --> negative ( - ) power rail

- GREEN LED :
     - GND --> LED Cathode (short leg)
     - GPIO Pin 4 --> one end of 330 Ω Resistor
     - Other end of 330 Ω Resistor --> LED Anode (long leg)
- RED LED:
     - GND --> LED Cathode (short leg)
     - GPIO Pin 18 --> one end of 330 Ω Resistor
     - Other end of 330 Ω Resistor --> LED Anode (long leg)

- Buzzer:
     - WIRES SHOULD BE ON THE SAME SIDE OF THE BUZZER
     - GND --> One leg of buzzer
     - GPIO Pin 21 --> Other leg of buzzer
 
- Button:
     - CONNECT BUZZER OVER THE CENTER LINE OF BREADBOARD
     - GND --> One side of the button
     - GPIO Pin 22 --> Other side of button on OPPOSITE LEG (if using FRONT leg of the button for GND on one side, use BACK LEG on the OTHER SIDE for GPIO pin)

---

#PC-Side Code - Allows Text-to-Speak through PC Speakers

The code creates a small server that runs on the PC (NOT ESP32) and listens on port 5000; speaking the randomly selected words through the speakers. 

Write it using Visual Studio Code (or any other preferred code editor) and save it under a memorable name (I chose 'pc_tts_server'). To run it, import the file via Bash by using the command, "python filename". 

SAVE AND RUN THE FOLLOWING ON YOUR **PC**: 

from flask import Flask, request
import threading
import queue
import subprocess

app = Flask(__name__)

speech_queue = queue.Queue()

def tts_worker():
    while True:
        word = speech_queue.get()
        if word is None:
            break

        print(f"[PC] Speaking: {word}")

        # Use Windows built-in TTS (never freezes)
        subprocess.run([
            "powershell",
            "-Command",
            (
            f"Add-Type –AssemblyName System.Speech; " +
            f"$speak = New-Object System.Speech.Synthesis.SpeechSynthesizer; " +
            f"$speak.Speak('{word}');"
            )
        ])

        speech_queue.task_done()

thread = threading.Thread(target=tts_worker, daemon=True)
thread.start()

@app.route("/say")
def say():
    word = request.args.get("word", "")
    if word:
        speech_queue.put(word)
        return "OK", 200
    return "No word provided", 400

if __name__ == "__main__":
    print("PC TTS server running on port 5000...")
    print("Waiting for ESP32 requests...")
    app.run(host="0.0.0.0", port=5000)




**NOTE** "Flask" may not be recognized and may need installed. To do so, try the following commands in Command Prompt:

      pip install flask     -------> this method may not work depending on the python version installed
      python3.14 -m pip install flask          ---------> this works for Python version 3.14.2
      
---

#Wi-Fi Configuration

****IMPORTANT**** Edit the credentials before deploying:

SSID = "your_SSID"
Password = "your_password"

After a successful connection the ESP32 prints its IP address over the serial port:

        Connecting to Wi-Fi...
        Click HERE to open the game! --> `http://<ESP32-IP>/`

Open that IP address in any browser on the same network to reach the game page.

---

#Webpage

The ESP32 serves a simple HTML page at `http://<ESP32-IP>/` that does the following:

-  Provides input box for user spelling as well as a "submit" button
-	Receives the submitted answer, checks it, returns result
-	Provides answer feedback such as "Correct! Good Job!" or "Incorrect! Try again!"

---

#Button — Triggering New Words

Pressing the hardware button calls the same  ‘/next’ logic directly on the ESP32 — no browser interaction needed. 

---

#Customizing the Word List

  Add, remove, or replace words freely.
  Words are chosen at random and spell checking is case-insensitive.


THIS CODE IS FREE TO MODIFY AND SHARE!!!

---


****Full ESP32 Script****



