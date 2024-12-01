# AI-chatbot-on-botpress-with-Meta
To integrate your chatbot created on Botpress using OpenAI into Facebook, Instagram, and WhatsApp for your gym, follow the steps below. This code will illustrate the integration process with the help of Botpress APIs and respective social media platform APIs.
Prerequisites:

    Botpress Chatbot: You must already have your Botpress instance running.
    Meta Developer Account: Access to the Meta for Developers dashboard.
    WhatsApp Business API: Registered WhatsApp business account.
    Facebook Page: Admin access to your gym's page.

Step 1: Meta API Setup

    Visit the Meta for Developers website.
    Create a new app and set up Messenger for Facebook and Instagram integration.
    Obtain the following:
        Access Token
        Verify Token
        Webhook URL

Step 2: Configure Webhooks in Meta API

    Configure a webhook URL to point to your Botpress instance:
        URL: https://your-botpress-url/hooks/facebook
        Ensure it can receive POST requests.

    Select events like messages, message_reactions, etc., for Messenger and Instagram.

Step 3: Integration Code (Python Example)

Here is an example Python script to handle the integration:

from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# Meta API Credentials
ACCESS_TOKEN = "your_meta_access_token"
VERIFY_TOKEN = "your_verify_token"
BOTPRESS_URL = "https://your-botpress-instance/api/v1/bots/<your-bot-id>/converse"

# Facebook/Instagram Webhook Verification
@app.route('/webhook', methods=['GET'])
def verify_webhook():
    if request.args.get('hub.verify_token') == VERIFY_TOKEN:
        return request.args.get('hub.challenge')
    return "Verification token mismatch", 403

# Receive messages from Meta (FB, IG, WhatsApp)
@app.route('/webhook', methods=['POST'])
def handle_messages():
    data = request.json
    for entry in data.get('entry', []):
        for messaging_event in entry.get('messaging', []):
            sender_id = messaging_event['sender']['id']
            if 'message' in messaging_event:
                user_message = messaging_event['message']['text']
                # Send the message to Botpress
                response = requests.post(
                    f"{BOTPRESS_URL}/{sender_id}",
                    json={"type": "text", "text": user_message},
                    headers={"Authorization": f"Bearer {ACCESS_TOKEN}"}
                )
                bot_response = response.json().get('responses', [{}])[0].get('text', "I couldn't understand that.")
                # Send bot response back to user
                send_message(sender_id, bot_response)
    return "OK", 200

def send_message(recipient_id, message_text):
    """Send message to the user via Facebook Messenger."""
    url = f"https://graph.facebook.com/v12.0/me/messages?access_token={ACCESS_TOKEN}"
    response = requests.post(
        url,
        json={
            "recipient": {"id": recipient_id},
            "message": {"text": message_text}
        }
    )
    return response.json()

if __name__ == "__main__":
    app.run(port=5000, debug=True)

Step 4: Botpress Side Configuration

    Make sure the Botpress bot supports external channels:
        Configure an external webhook for incoming messages from Facebook/Instagram/WhatsApp.
        Use the https://your-botpress-instance/hooks/facebook endpoint.

    Test the flow to ensure the bot is correctly responding.

Step 5: WhatsApp Integration (Optional)

To integrate WhatsApp:

    Use the Twilio API for WhatsApp or Meta's WhatsApp Cloud API.
    Replace the send_message function with the appropriate API calls for WhatsApp.

Notes

    Security: Secure your tokens and restrict the webhook URL.
    Scalability: Consider deploying this script on a production-grade server or cloud service.
    Testing: Test extensively to ensure the bot behaves as expected across all channels.

This approach helps manage and scale your Botpress chatbot to serve customers on multiple platforms seamlessly.
