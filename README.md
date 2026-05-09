#-----ESP32 Spelling Bee Game-----

An interactive spelling-bee game running on an ESP32 with MicroPython. The ESP32 hosts a webpage allowing players to submit spelling answers from any browser on the same Wi-Fi network. Hardware feedback (green/red LEDs and a buzzer) provides instant “correct” or “incorrect” cues.  

Challenge your spelling skills with common words that are surprisingly tricky to spell! 

---

#Table of Contents
1. Overview
2. Features
3. Hardware Requirements
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
   - Wrong → Red LED + dull buzzer buzz

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
-	11x Jumper wires 
-	Tactile push-button 
-	Passive buzzer 
-	Green LED 
-	Red LED 
-	2x 330 Ω Resistors - one per LED

---

#Wi-Fi Configuration

****IMPORTANT**** Edit the credentials before deploying:

SSID = "your_SSID"
Password = "your_password"

After a successful connection the ESP32 prints its IP address over the serial port:

        Connecting to Wi-Fi...
        Connected to (SSID)
        Click HERE to open the game! --> `http://<ESP32-IP>/`

Open that IP address in any browser on the same network to reach the game page.

---

#Webpage

The ESP32 serves a single HTML page at `http://<ESP32-IP>/` that does the following:

-	Submits the current word prompt to the web page
-	Receives the submitted answer, checks it, returns result 
-	Selects the next random word (also triggered by the button)


---

#Button — Triggering New Words

Pressing the hardware button calls the same  ‘/next’ logic directly on the ESP32 — no browser interaction needed. 

---

#Customizing the Word List

  Add, remove, or replace words freely.
  Words are chosen at random and spell checking is case-insensitive.


THIS CODE IS FREE TO MODIFY AND SHARE!!!


------------------------------------------------FULL ESP32 SPELLING BEE SCRIPT: ---------------------------------------------------


import network
import socket
import machine
import time
import ure
import urequests

#-----Hardware-----

button = machine.Pin(22, machine.Pin.IN)
buzzer = machine.Pin(21, machine.Pin.OUT)
green_led = machine.Pin(4, machine.Pin.OUT)
red_led = machine.Pin(12, machine.Pin.OUT)

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
]

current_word = None
last_result = ""
last_correct_spelling = ""
last_message = ""

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

    print("Connected to DecoDeco")
    return wlan.ifconfig()[0]

ip = connect_wifi()

#-----Button Control-----

def wait_for_button():
    global current_word, last_result, last_correct_spelling, last_message
    if button.value() == 0:  # pressed
        time.sleep_ms(20)
        if button.value() == 0: #nothing lights up if nothing is pressed
            green_led.value(0)
            red_led.value(0)

            # Pick a new word
            import urandom #for random selection 
            current_word = WORDS[urandom.getrandbits(8) % len(WORDS)] #recieve a random word
            last_result = ""
            last_correct_spelling = ""
            last_message = ""
            
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

# -----------------------------
# Web Server
# -----------------------------
addr = socket.getaddrinfo("0.0.0.0", 80)[0][-1]
s = socket.socket()
s.bind(addr)
s.listen(1)
print("Click HERE to open the game! --> http://%s/" % ip)

#-----Main Loop-----

while True:
    wait_for_button()

    try:
        conn, addr = s.accept()
        request = conn.recv(1024).decode() #read the data and convert into string response

        # Extract answer from URL
        match = ure.search("answer=([^& ]+)", request)
        if match and current_word:
            user_answer = match.group(1).replace("%20", " ").lower() #return answer, add a space for URL, allow case-insenitivity
            correct = (user_answer == current_word.lower()) #results in True or False

            if correct:
                green_led.value(1)
                red_led.value(0)
                
            else:
                green_led.value(0)
                red_led.value(1)
               

            last_correct_spelling = print(f"Correct spelling: {current_word}")

        response = webpage()
        conn.send("HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n") #add 200 OK to verify successful connection
        conn.sendall(response) #send the response from the webpage
        conn.close() #close the connection

    except Exception as e:
        print("Error:", e)
