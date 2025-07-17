import speech_recognition as sr
import pyttsx3
import random

# Define multiple customer profiles
profiles = [
 {
        "policy_holder_name": "Mr. Pratik Jadhav",
        "product_name": "Secure Plus Plan",
        "policy_number": "VE123456",
        "policy_start_date": "8th July 2025",
        "total_premium_paid": "₹15,000",
        "outstanding_amount": "₹5,000",
        "premium_due_date": "8th June 2026",
        "sum_assured": "₹2,00,000",
        "fund_value": "₹26,500"
        }
]

current_profile = random.choice(profiles)

# Initialize TTS engine
engine = pyttsx3.init()
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[1].id)
engine.setProperty('rate', 170)
engine.setProperty('volume', 1.0)

# Common affirmatives
affirmatives = ["yes", "yeah", "yup", "okay", "sure", "fine", "alright", "hm", "hmm", "ok", "yep", "speaking"]
negatives = ["no", "not now", "busy", "later", "can't", "don't"]

# Speak function
def speak(text):
    print(f"Veena: {text}")
    engine.say(text)
    engine.runAndWait()

# Listen function
def listen(prompt=""):
    r = sr.Recognizer()
    with sr.Microphone() as source:
        if prompt:
            speak(prompt)
        print("You can speak now, I am listening...")
        r.adjust_for_ambient_noise(source)
        try:
            audio = r.listen(source, timeout=8, phrase_time_limit=10)
            response = r.recognize_google(audio)
            print("You:", response)
            return response.lower()
        except:
            speak("Sorry, I didn’t understand. Can you repeat?")
            return listen()

# Utility function to check affirmatives

def is_affirmative(response):
    return any(word in response for word in affirmatives)

def is_negative(response):
    return any(word in response for word in negatives)

# Main conversation logic
def start_conversation(profile):
    speak(f"Hello and very Good Morning Sir, May I speak with {profile['policy_holder_name']}?")
    response = listen()

    if is_affirmative(response):
        speak("My name is Veena and I am an Executive calling on behalf of ValuEnable Life Insurance Co. Ltd, this is a service call with regards to your life insurance policy.")
        speak("Is this the right time to speak to you regarding the renewal of your policy?")
        response = listen()

        if is_affirmative(response):
            branch_2(profile)
        else:
            speak(f"May I know your relationship with {profile['policy_holder_name']}?")
            relation = listen()
            speak(f"Do you handle {profile['policy_holder_name']}'s life insurance policy number {profile['policy_number']}? Are you aware of the details of this policy?")
            confirm = listen()
            if is_affirmative(confirm):
                speak("It will take just 2 minutes of your time. Can we discuss it right now or should I reschedule your call at a better time?")
                response = listen()
                if is_affirmative(response):
                    branch_2(profile)
                else:
                    branch_3()
            else:
                branch_3()

    elif is_negative(response):
        branch_3()
    else:
        speak("Sorry, I didn't get that. Can I speak to the policyholder?")
        listen()

# Branch 2.0 - Confirm Policy

def branch_2(profile):
    speak(f"Let me start by confirming your policy details. Your policy is ValuEnable Life {profile['product_name']} insurance policy number is {profile['policy_number']}, started on {profile['policy_start_date']}, and you've paid {profile['total_premium_paid']} so far. The premium of {profile['outstanding_amount']} due on {profile['premium_due_date']} is still pending, and your policy is currently in Discontinuance status, with no life insurance cover.")
    speak("Could you please let me know why you haven’t been able to pay the premium?")
    reason = listen()

    if any(word in reason for word in ["money", "financial", "no job", "problem"]):
        branch_7(profile)
    elif "already paid" in reason or "paid" in reason:
        branch_6()
    elif "no bond" in reason or "don’t have bond" in reason:
        branch_4()
    elif is_negative(reason):
        branch_8(profile)
    else:
        speak(f"The due date for renewal premium payment for your policy was on {profile['premium_due_date']}, the grace period is over. Would you like to know the policy benefits if you resume payments?")
        response = listen()
        if is_affirmative(response):
            speak("You get maximum allocation in fund, better returns, loyalty units, and tax savings under section 80C and 10(10D). Does this help you make a better decision?")
            response = listen()
            if is_affirmative(response):
                branch_5()
            else:
                branch_9()

# Branch 3.0 - Reschedule

def branch_3():
    speak("When would be a convenient time to call you again? Please tell a time and date.")
    listen()
    speak("Thank you, I will arrange a call back at the given time.")
    branch_9()

# Branch 4.0 - No Bond

def branch_4():
    speak("You can download the policy bond on WhatsApp. Send a message from your registered number to 8806727272.")
    branch_9()

# Branch 5.0 - Ready to Pay

def branch_5():
    speak("May I know how you plan to make the payment? Cash, cheque or online?")
    mode = listen()
    if "cheque" in mode or "cash" in mode:
        speak("You can also pay online via debit card, credit card, net banking, PhonePe, WhatsApp, or Google Pay.")
    elif "branch" in mode:
        speak("You can pay from home. I can assist with digital payment.")
    else:
        speak("Noted. I’ll send you a payment link for easy processing.")
    branch_9()

# Branch 6.0 - Already Paid

def branch_6():
    speak("Thank you for making the payment. May I know when you made it?")
    date = listen()
    speak("Where did you make the payment, online, cheque, or cash?")
    method = listen()
    speak("Please provide the transaction ID or cheque number.")
    listen()
    branch_9()

# Branch 7.0 - Financial Issue

def branch_7(profile):
    speak("I understand your concern. You can pay by credit card, EMI, or monthly mode. Can you arrange the premium to continue benefits?")
    listen()
    branch_9()

# Branch 8.0 - Rebuttals

def branch_8(profile):
    speak(f"If premiums stop before 5 years, policy discontinues with 4-4.5% return only. You lose sum assured of {profile['sum_assured']}. If continued, maturity value is {profile['fund_value']}. Will you pay now?")
    response = listen()
    if is_affirmative(response):
        branch_5()
    else:
        branch_9()

# Branch 9.0 - Closure

def branch_9():
    speak("For help, call 1800 209 7272 or WhatsApp us on 8806 727272. Thank you for your time. Have a great day!")

# Start bot
start_conversation(current_profile)
