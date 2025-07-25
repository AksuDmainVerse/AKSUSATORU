import os
import discord
import openai

intents = discord.Intents.default()
intents.messages = True
intents.message_content = True
client = discord.Client(intents=intents)

# Load from environment
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
DISCORD_BOT_TOKEN = os.getenv("DISCORD_BOT_TOKEN")
openai.api_key = OPENAI_API_KEY

# ----- Dual Personality System -----
def generate_prompt(username, message, gender):
    if gender == "boy":
        personality = "You are Sigma Aksu, a savage, confident, bold MLBB coach. Use Hinglish. Be short, punchy, and motivational. End with 🔥 if needed."
    elif gender == "girl":
        personality = "You are Charming Aksu, romantic and emotional. You flirt kindly, speak in Hinglish, and always encourage and comfort. End with ❤️."
    else:
        personality = "You are Aksu, an MLBB bot that replies in a friendly tone in Hinglish."
    
    return f"{personality}\nUser: {message}\nAksu:"

# --- Gender guess from message (simplified) ---
def detect_gender(message):
    if any(word in message.lower() for word in ["baby", "jaan", "love", "miss you", "romantic"]):
        return "girl"
    elif any(word in message.lower() for word in ["bro", "bhai", "game", "rank", "savage"]):
        return "boy"
    return "neutral"

# --- On message received ---
@client.event
async def on_message(message):
    if message.author == client.user:
        return
    
    if message.content.startswith("/aksu"):
        user_input = message.content.replace("/aksu", "").strip()
        gender = detect_gender(user_input)
        prompt = generate_prompt(message.author.name, user_input, gender)

        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "system", "content": prompt}],
            max_tokens=100
        )

        await message.channel.send(response.choices[0].message["content"])

# Bot Ready
@client.event
async def on_ready():
    print(f"Aksu is online as {client.user}")

# Run bot
client.run(DISCORD_BOT_TOKEN)
