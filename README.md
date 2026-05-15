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


#ESP32 Spelling Bee Game
#Challenge your spelling skills with common words that are surprisingly tricky to spell!

#-----Libraries-----

import network
import socket
import machine
import time
import ure
import urequests
import uio
import sys

#-----Hardware-----

button = machine.Pin(22, machine.Pin.IN, machine.Pin.PULL_UP)
buzzer = machine.Pin(21, machine.Pin.OUT)
green_led = machine.Pin(4, machine.Pin.OUT)
red_led = machine.Pin(18, machine.Pin.OUT)

#-----Sound Functions-----

def tone(freq, duration):
    if freq == 0:
        buzzer.value(0)
        time.sleep_ms(duration)
        return
    pwm = machine.PWM(buzzer)
    pwm.freq(freq)
    pwm.duty(512)
    time.sleep_ms(duration)
    pwm.deinit()

def happy_sound():
    melody = [600, 800, 1000]
    for f in melody:
        tone(f, 150)
        time.sleep_ms(50)

def sad_sound():
    melody = [800, 600, 400]
    for f in melody:
        tone(f, 200)
        time.sleep_ms(50)

#-----Word List-----

WORDS = [
  "Fahrenheit",
  "Definitely",
  "Restaurant",
  "Vacuum",
  "Separate",
  "Maintenance",
  "Yacht",
  "Necessary",
  "Bureaucracy",
  "Onomatopoeia",
  "Mischievous",
  "Psychology",
  "Receipt",
  "License",
  "Separators",
  "Pneumonia",
  "Croissant",
  "Accommodate",
  "Nauseous"
  
]

current_word = None
last_result = ""
last_correct_spelling = ""
last_message = ""

#-----Button Control-----
#Used to ensure main loop doesn't waste time checking button state; prevents delay in performance

button_pressed = False #start button as unpressed 

def handle_button(pin):
    global button_pressed #allows use for OUTSIDE of main loop
    button_pressed = True

button.irq(trigger=machine.Pin.IRQ_FALLING, handler=handle_button) #Button(pin) will change from high to low state after being pressed

#-----Connect to Wifi-----

SSID = "DecoDeco"
Password = "@liensRr3al?"

def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(SSID, Password)

    print("Connecting to Wi-Fi...")
    while not wlan.isconnected():
        time.sleep(0.2)

    print("Connected:", wlan.ifconfig())
    return wlan.ifconfig()[0]

ip = connect_wifi()

#-----PC Text-To-Speech-----

PC_IP = "192.168.68.51"

def speak_on_pc(word):
    try:
        url = f"http://{PC_IP}:5000/say?word={word}"
        print("Sending new word to PC...")
        #fire-and-forget/ send request to the PC "say" url, then continue without waiting for response
        urequests.get(url)
    except:
        pass

#-----HTML Page-----

def webpage():
    return f"""
<!DOCTYPE html>
<html>
<head>
<title>ESP32 Spelling Bee</title>
<style>
body {{ font-family: Arial; margin: 40px; }}
input {{ font-size: 20px; padding: 8px; }}
button {{ font-size: 20px; padding: 8px; }}
</style>
</head>
<body>
<h1>Welcome to the ESP32 Spelling Bee!</h1>

<p>Press the button on the ESP32 to hear your word!</p>
<p>Good luck!</p>

<form action="/" method="get">
    <input type="text" name="answer" placeholder="Type your answer" autofocus>
    <button type="submit">Submit</button>
</form>

<h2>{last_result}</h2>
<p>{last_correct_spelling}</p>
<p>{last_message}</p>

</body>
</html>
"""

#-----Web Server-----

addr = socket.getaddrinfo("0.0.0.0", 80)[0][-1]
s = socket.socket()
s.bind(addr)
s.listen(1)
s.settimeout(0.1) #Wait for client with a timeout to prevent hanging
print("Click HERE to open the game! --> http://%s/" % ip)

#-----Main Loop-----

try:
    while True:

        #Add button control
        if button_pressed:
            button_pressed = False #Reset the button; won't trigger until pressed again

            import urandom
            current_word = WORDS[urandom.getrandbits(8) % len(WORDS)] #Choose a new word
            print("Finding new word...")

            speak_on_pc(current_word) #PC announces the word through speakers
            print("Delivered! Happy spelling!")

        #Connect to the webserver
        try:
            conn, addr = s.accept() #Accept the connection
            request = conn.recv(1024).decode() #Read the data; convert it into a string response
            
            #Recieve the answer from webpage URL
            match = ure.search("answer=([^& ]+)", request)

            if match is not None and current_word:
                user_answer = match.group(1).replace("%20", " ").lower() #Replace URL spaces and make string case-insensitive
                correct = user_answer.strip().lower() == current_word.strip().lower() #Answers correct if spelling matches regardless of capitalization

                if correct:
                    print("-" *40) #Separator lines for cleaner printout appearance
                    print("HOORAY! Great spelling! :)") #Correct answer message
                    print("-" *40)
                    green_led.value(1) #Green light turns on
                    red_led.value(0)
                    happy_sound() #Happy buzzer sound plays
                    time.sleep(5)
                    green_led.value(0) #Green LED turns off after 5 seconds
                   
                    #Display 'correct' output on webpage
                    last_result = "Correct!" 
                    last_message = "Good job!"

                else:
                    print("-" *40) #Separator lines
                    print("Aww, man :( better luck next time!") #Incorrect answer message
                    print("Correct spelling:", current_word, "|", "Your spelling:", user_answer) #Display correct spelling compared to user input
                    print("-" *40)
                    green_led.value(0)
                    red_led.value(1) #Red LED turns on
                    sad_sound() #Sad buzzer sound plays
                    time.sleep(5)
                    red_led.value(0) #Red LED turns off after 5 seconds
                    
                    #Display 'incorrect' output on webpage
                    last_result = "Incorrect!"
                    last_message = "Try again!"

                last_correct_spelling = f"Correct spelling: {current_word}" #Show correct spelling on HTML page
                current_word = None #Prevent re-grading

            conn.send("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n") #Return 200(OK) to indicate successful HTTP response
            conn.sendall(webpage()) #Send response from the webpage
            conn.close() #Close the connection

        except OSError:
            pass

        time.sleep_ms(5)

except Exception as e:
    print("CRASHED WITH ERROR:")
    print(e)



