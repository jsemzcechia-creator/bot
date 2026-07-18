from discord.ext import commands
from discord import app_commands
import datetime
import requests
import os

# Importujeme kód pro tikety a web server
from keep_alive import keep_alive
from ticket_system import TicketView

# --- KONFIGURACE ---
FIVEM_CFX_CODE = "ZDE_VLOZ_SVOJ_CFX_KOD"  # Sem dej svůj kód z cfx.re URL (např. "abc123")
ANTI_RAID_MIN_AGE_DAYS = 3  # Minimální věk účtu (ve dnech) pro vstup na server

class Bot(commands.Bot):
    def __init__(self):
        intents = discord.Intents.default()
        intents.message_content = True
        intents.members = True  # Nutné mít zapnuté v Discord Developer Portálu!
        super().__init__(command_prefix="!", intents=intents)

    async def setup_hook(self):
        await self.tree.sync()
        self.add_view(TicketView())

bot = Bot()

# Spuštění webu na pozadí
keep_alive()

# --- FUNKCE PRO FIVEM STATUS ---
def get_fivem_status(cfx_code):
    headers = {'User-Agent': 'Mozilla/5.0'}
    try:
        url = f"https://servers-frontend.fivem.net/api/servers/single/{cfx_code}"
        response = requests.get(url, headers=headers, timeout=5)
        if response.status_code == 200:
            data = response.json()
            players_count = len(data['Data']['players'])
            max_players = data['Data']['vars']['sv_maxclients']
            server_name = data['Data']['hostname']
            return True, players_count, max_players, server_name
    except Exception:
        pass
    return False, 0, 0, ""

# --- SLASH PŘÍKAZY ---

@bot.tree.command(name="setup", description="Vytvoří ticketovací panel v tomto kanálu.")
@app_commands.checks.has_permissions(administrator=True)
async def setup_tickets(interaction: discord.Interaction):
    embed = discord.Embed(
        title="🎫 Centrum Podpory & Ticketů",
        description=(
            "Potřebuješ pomoct, nahlásit hráče nebo kontaktovat vedení?\n"
            "Vyber si příslušnou kategorii z menu níže.\n\n"
            "⚠️ **Provozní doba:** Pondělí - Pátek (od 07:00)."
        ),
        color=discord.Color.green()
    )
    embed.set_thumbnail(url="https://images.cfx.re/thumbnails/fivem.png")
    
    await interaction.response.send_message("Panel byl úspěšně vytvořen!", ephemeral=True)
    await interaction.channel.send(embed=embed, view=TicketView())

@bot.tree.command(name="status", description="Zobrazí aktuální stav FiveM serveru.")
async def fivem_status(interaction: discord.Interaction):
    await interaction.response.defer()
    online, players, max_players, name = get_fivem_status(FIVEM_CFX_CODE)
    
    if online:
        embed = discord.Embed(
            title="🟢 Server je ONLINE!",
            description=f"**Název:** {name[:100]}...\n**Hráči:** `{players} / {max_players}`",
            color=discord.Color.green()
        )
    else:[gemini-code-1784293638223.py](https://github.com/user-attachments/files/30144161/gemini-code-1784293638223.py)
        embed = discord.Embed(
            title="🔴 Server je OFFLINE!",
            description="Nepodařilo se připojit k serveru. Zkontroluj, zda běží.",from flask import Flask
from threading import Thread

app = Flask('')

@app.route('/')
def home():
    return "Bot je online na Discloudu!"

def run():
    app.run(host='0.0.0.0', port=8080)

def keep_alive():
    t = Thread(target=run)
    t.start()
            color=discord.Color.red()
        )
    await interaction.followup.send(embed=embed)

# --- ANTI-RAID OCHRANA ---

@bot.event
async def on_member_join(member: discord.Member):
    now = datetime.datetime.now(datetime.timezone.utc)
    account_age = now - member.created_at

    if account_age.days < ANTI_RAID_MIN_AGE_DAYS:
        try:
            await member.send(
                f"❌ Byl jsi automaticky vyhozen ze serveru **{member.guild.name}**.\n"
                f"Důvod: Tvůj účet na Discordu je příliš nový ({account_age.days} dní). "
                f"Ochrana vyžaduje věk účtu alespoň {ANTI_RAID_MIN_AGE_DAYS} dny."
            )
        except Exception:
            pass
        await member.kick(reason=f"Anti-Raid: Nový účet ({account_age.days} dní).")

# Spuštění přes token z proměnných
bot.run(os.environ.get('DISCORD_TOKEN'))
