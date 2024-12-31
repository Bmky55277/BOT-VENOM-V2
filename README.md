from flask import Flask, request
from twilio.twiml.messaging_response import MessagingResponse
import sqlite3
from datetime import datetime

app = Flask(__name__)

# Initialisation de la base de données
def init_db():
    conn = sqlite3.connect('bot_database.db')
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS user_messages (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            phone_number TEXT,
            message TEXT,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    conn.commit()
    conn.close()

# Enregistrer les messages utilisateur
def save_message(phone_number, message):
    conn = sqlite3.connect('bot_database.db')
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO user_messages (phone_number, message) VALUES (?, ?)
    ''', (phone_number, message))
    conn.commit()
    conn.close()

# Gérer la logique de réponse
def get_response(message):
    message = message.lower()
    if "bonjour" in message or "salut" in message:
        return "Bonjour ! Comment puis-je vous aider aujourd'hui ? Tapez 'aide' pour plus d'options."
    elif "aide" in message:
        return ("Voici ce que je peux faire :\n"
                "1. Répondre à des questions simples.\n"
                "2. Enregistrer vos signalements.\n"
                "3. Tapez 'bug' pour signaler un problème.")
    elif "bug" in message:
        return "Merci d'avoir signalé un bug. Veuillez fournir une description détaillée."
    elif "merci" in message:
        return "Avec plaisir ! Si vous avez d'autres questions, n'hésitez pas."
    else:
        return "Je suis désolé, je ne comprends pas votre message. Tapez 'aide' pour voir ce que je peux faire."

# Endpoint principal pour WhatsApp
@app.route('/webhook', methods=['POST'])
def webhook():
    incoming_msg = request.values.get('Body', '').strip()  # Message utilisateur
    sender = request.values.get('From', '')  # Numéro de téléphone de l'expéditeur
    
    # Sauvegarder le message
    save_message(sender, incoming_msg)
    
    # Générer une réponse
    response = get_response(incoming_msg)
    
    # Créer une réponse Twilio
    twilio_response = MessagingResponse()
    twilio_response.message(response)
    return str(twilio_response)

if __name__ == "__main__":
    # Initialiser la base de données
    init_db()
    print("Le bot fonctionne !")
    app.run(debug=True, host='0.0.0.0', port=5000)
