import discord
import aiohttp
import json
import re
import base64  # For image encoding

TOKEN = "x"  # Replace with your bot's token
API_URL = "http://localhost:1234/v1/chat/completions"  # LM Studio API

RELAY_GUILD_ID = x  # Replace with the target server ID
RELAY_CHANNEL_ID = x  # Replace with the target channel ID

intents = discord.Intents.default()
intents.message_content = True
client = discord.Client(intents=intents)

conversation_history = [] 
PREFIXES = ["!", "!!"]


async def get_ai_response(user_input, image_data=None):
    # If image is provided, use multimodal format
    if image_data:
        image_base64 = base64.b64encode(image_data).decode("utf-8")
        # Using OpenAI-style multimodal message content
        message_content = [
            {"type": "text", "text": user_input},
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image_base64}"}}
        ]
        conversation_history.append({"role": "user", "content": message_content})
    else:
        conversation_history.append({"role": "user", "content": user_input})

    payload = {
        "model": "YOUR_LLM_FROM_LM_STUDIO",  # Put your LLM here from LM studio
        "messages": conversation_history,
    }

    async with aiohttp.ClientSession() as session:
        try:
            async with session.post(API_URL, json=payload) as response:
                if response.status == 200:
                    data = await response.json()
                    ai_response = data["choices"][0]["message"]["content"].strip()
                    ai_response = re.sub(r"<think>.*?<\/think>", "", ai_response, flags=re.DOTALL).strip()
                    conversation_history.append({"role": "assistant", "content": ai_response})
                    return ai_response
                else:
                    return f"Error: {response.status} - {await response.text()}"
        except Exception as e:
            return f"Error: {str(e)}"

async def relay_message(message):
    if message.author.bot or not any(message.content.startswith(prefix) for prefix in PREFIXES) and not message.reference:
        return

    relay_guild = client.get_guild(RELAY_GUILD_ID)
    if relay_guild:
        relay_channel = relay_guild.get_channel(RELAY_CHANNEL_ID)
        if relay_channel:
            channel_mention = message.channel.mention if isinstance(message.channel, discord.TextChannel) else "Direct Message"
            embed = discord.Embed(
                title="Relayed Message",
                description=f"**From:** {message.author.mention} in {channel_mention}\n\n**Message:**\n{message.content}",
                color=discord.Color.blue()
            )
            await relay_channel.send(embed=embed)

@client.event
async def on_ready():
    print(f'Logged in as {client.user}')

@client.event
async def on_message(message):
    if message.author.bot:
        return

    user_input = None
    image_data = None

    if any(message.content.startswith(prefix) for prefix in PREFIXES):
        user_input = message.content[len(next(prefix for prefix in PREFIXES if message.content.startswith(prefix))):].strip()

        # Check for image attachments
        if message.attachments:
            for attachment in message.attachments:
                if attachment.content_type and attachment.content_type.startswith("image/"):
                    image_data = await attachment.read()
                    break

        ai_response = await get_ai_response(user_input, image_data=image_data)
        response_chunks = [ai_response[i:i+2000] for i in range(0, len(ai_response), 2000)]
        for chunk in response_chunks:
            await message.channel.send(chunk)

        await relay_message(message)

client.run(TOKEN)

