### Python Programmer
**The displayed codes names are from projects made by myself.**


## Code Examples
```py
import discord
from discord.ext import commands, tasks
import pymongo
from ezcord import emb

class DM_Queuing(commands.Cog):
    def __init__(self, bot):
        self.bot : discord.Bot = bot
        self.MongoClient: pymongo.MongoClient = bot.MongoClient
        self.DB = self.MongoClient.get_database("Patrolly")
        self.collection = self.DB.get_collection("User_DM_Queue")

    @commands.Cog.listener()
    async def on_ready(self):
        if not self.dm_users.is_running():
            self.dm_users.start()

    @tasks.loop(seconds=5)
    async def dm_users(self):
        user_data = self.collection.find_one_and_delete({})
        
        if user_data:
            user_id = user_data.get("UserID")
            message = user_data.get("Message", "Default message")

            if message == "Bot_Joined":
                GuildName = user_data.get("GuildName")
                embed_title = "Hello! I am Patrolly!"
                embed_description = (
                    f"Thank you for adding Patrolly to {GuildName}, a lightweight Discord bot for ERLC. "
                    "Remote server management made easy with Patrolly.\n\n"
                    "Please make sure to run `/set-erlc-token` to set your ERLC token. "
                    "We are actively working on adding a dashboard to this bot!"
                )

            user = self.bot.get_user(user_id)
            if user:
                try:
                    await emb.info(target=user, txt=embed_description, title=embed_title)
                except Exception as e:
                    print(f"Error sending DM to {user.name}#{user.discriminator}: {e}")

    async def send_message(self, user, message, embed):
        try:
            if embed:
                await user.send(embed=embed)
            else:
                await user.send(message)
        except discord.Forbidden:
            print(f"Could not send DM to {user.name}#{user.discriminator} (Forbidden)")
        except discord.HTTPException as e:
            print(f"Error sending DM to {user.name}#{user.discriminator}: {e}")

    @dm_users.before_loop
    async def before_dm_users(self):
        await self.bot.wait_until_ready()

def setup(bot):
    bot.add_cog(DM_Queuing(bot))
```

Example 2

```py
import discord
from discord.ext import commands, tasks
import pytz
from datetime import datetime
import os

class BotAnalytics(commands.Cog):
    def __init__(self, bot):
        self.bot = bot
        self.analytics_channel_id = os.environ.get("ANALYTICS_CHANNEL_ID")
        self.MAIN_SERVER = os.environ.get("MAIN_SERVER")

        self.commands_executed = 0
        self.buttons_pressed = 0
        self.bot_added = 0
        self.bot_removed = 0
        self.errors_logged = 0

    @commands.Cog.listener()
    async def on_ready(self):
        print("Bot is ready.")
        if not self.analytics_report.is_running():
            self.analytics_report.start()

    def cog_unload(self):
        self.analytics_report.cancel()

    @commands.Cog.listener()
    async def on_interaction(self, interaction: discord.Interaction):
        if interaction.type == discord.InteractionType.application_command:
            self.commands_executed += 1

        elif interaction.type == discord.InteractionType.component:
            if interaction.data and interaction.data.get("component_type") == 2: 
                self.buttons_pressed += 1
    @commands.Cog.listener()
    async def on_guild_join(self, guild):
        self.bot_added += 1

    @commands.Cog.listener()
    async def on_guild_remove(self, guild):
        self.bot_removed += 1

    @commands.Cog.listener()
    async def on_command_error(self, ctx, error):
        self.errors_logged += 1

    @tasks.loop(seconds=60)
    async def analytics_report(self):
        berlin_tz = pytz.timezone("Europe/Berlin")
        now = datetime.now(berlin_tz)
        if now.hour == 0 and now.minute == 0:
            report_message = (
                "📋 › **Daily Bot Analytics Report**\n\n"
                "This message provides an overview of bot activity in the last **24 hours**.\n\n"
                f"📌 › **Commands Executed:** {self.commands_executed}\n"
                f"📌 › **Buttons Pressed:** {self.buttons_pressed}\n\n"
                f"➕ › **Bot Added to Servers:** {self.bot_added}\n"
                f"➖ › **Bot Removed from Servers:** {self.bot_removed}\n\n"
                f"⚠️ › **Errors Logged:** {self.errors_logged}\n\n"
                "_As of now, this data has been **permanently deleted** from the database._"
            )

            if not self.MAIN_SERVER or not self.analytics_channel_id:
                print("Error: Missing environment variables for MAIN_SERVER or ANALYTICS_CHANNEL_ID.")
                return

            guild = self.bot.get_guild(int(self.MAIN_SERVER))
            if guild is None:
                print(f"Error: Could not find guild with ID {self.MAIN_SERVER}.")
                return

            channel = guild.get_channel(int(self.analytics_channel_id))
            if channel is None:
                print(f"Error: Could not find channel with ID {self.analytics_channel_id}.")
                return

            await channel.send(report_message)

            self.commands_executed = 0
            self.buttons_pressed = 0
            self.bot_added = 0
            self.bot_removed = 0
            self.errors_logged = 0

    @analytics_report.before_loop
    async def before_analytics_report(self):
        await self.bot.wait_until_ready()

def setup(bot):
    bot.add_cog(BotAnalytics(bot))
```

**I am aware that some softwares use a simpler system, by minimizing the variable ammount. I am able to do that depending on the preference**
