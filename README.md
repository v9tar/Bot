import discord
from discord.ext import commands
from discord import app_commands, ui

TOKEN = "MTM2ODU1NzY5MTEzNTMzMjQxMg.GP__ST.6HpJyU_xsL38ZAkR2LHCw95_V0CN1tjuZVIAfI"
PASSWORD = "192qwertZ"
NOTIFY_CHANNEL_ID = 123456789012345678  # ضع معرف قناة الإشعارات هنا

intents = discord.Intents.default()
intents.messages = True
intents.guilds = True

bot = commands.Bot(command_prefix="!", intents=intents)
accounts = []

@bot.event
async def on_ready():
    print(f"✅ البوت جاهز: {bot.user}")
    try:
        synced = await bot.tree.sync()
        print(f"تمت مزامنة {len(synced)} أمر.")
    except Exception as e:
        print("خطأ في المزامنة:", e)

# كلاس الزرّين التفاعليين لكل حساب
class AccountButtons(ui.View):
    def __init__(self, account_name):
        super().__init__(timeout=None)
        self.account_name = account_name

        # زر نسخ الاسم
        self.add_item(ui.Button(
            label="📋 نسخ الاسم",
            style=discord.ButtonStyle.primary,
            custom_id=f"name:{account_name}"
        ))

        # زر نسخ كلمة المرور
        self.add_item(ui.Button(
            label="🔐 نسخ كلمة المرور",
            style=discord.ButtonStyle.secondary,
            custom_id="password"
        ))

    @ui.button(label="placeholder", style=discord.ButtonStyle.gray, disabled=True)
    async def button_click(self, interaction: discord.Interaction, button: discord.ui.Button):
        pass

# مستمع للأزرار
@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return

    # استخراج نوع الزر
    data = interaction.data["custom_id"]

    if data.startswith("name:"):
        acc_name = data.split("name:")[1]
        await interaction.response.send_message(f"**👤 اسم الحساب:** `{acc_name}`", ephemeral=True)

    elif data == "password":
        await interaction.response.send_message(f"**🔐 كلمة المرور:** `{PASSWORD}`", ephemeral=True)

# أمر لإضافة حساب
@bot.tree.command(name="اضف_حساب", description="أضف حساب روبلوكس للقائمة")
async def add_account(interaction: discord.Interaction, account_name: str):
    accounts.append(account_name)
    await interaction.response.send_message(f"✅ تم إضافة الحساب: `{account_name}`", ephemeral=True)

    # إشعار تلقائي في قناة مخصصة
    channel = bot.get_channel(NOTIFY_CHANNEL_ID)
    if channel:
        embed = discord.Embed(
            title="🆕 حساب جديد",
            description=f"**👤 الاسم:** `{account_name}`\n**🔐 كلمة المرور:** `{PASSWORD}`",
            color=0x00ff88
        )
        await channel.send(embed=embed)

# أمر حذف حساب
@bot.tree.command(name="حذف_حساب", description="حذف حساب روبلوكس من القائمة")
async def delete_account(interaction: discord.Interaction, account_name: str):
    if account_name in accounts:
        accounts.remove(account_name)
        await interaction.response.send_message(f"✅ تم حذف الحساب: `{account_name}`")
    else:
        await interaction.response.send_message("❌ هذا الحساب غير موجود.")
