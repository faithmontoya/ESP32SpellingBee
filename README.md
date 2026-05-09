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
-	1x 10 kΩ pull-down Resistor (for button)

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
