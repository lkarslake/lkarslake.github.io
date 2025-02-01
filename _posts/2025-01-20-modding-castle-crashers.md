---
title: "Modding Castle Crashers Remastered PC's Game Logic: Experience Distribution"
date: 2025-01-20 00:00:00 +1000   
categories: [Modding]
tags: [Castle Crashers, Modding]
author: lkarslake
description: "A short guide on how I modded the Castle Crashers' game logic"
---
## The Goal
For the longest time I wanted to mod Castle Crashers. I love the game but
I was always frustrated with how the experience was distributed in
multiplayer. By default, the game distributes 1 experience point for each
hit to the hitting player (whether it is a hit from their weapon, magic,
bow or pet). The reason why this is a problem is that certain builds 
tend to do a larger AoE than others. This causes a significant level
lead for players who choose these builds. Not only does
this disincentivize playing other builds with less AoE capabilities, it
is not great being more susceptible to downing and having no chance at
face the princess. To fix this I decided to learn how to mod the game to
change the experience system to a shared distribution system.

## The Research
I knew very little about game modding, so I did what any
reasonable person would do and consulted DR. Google. After digging around
the surface I was quickly becoming discouraged- no official modding API,
some people claiming you had to use CheatEngine scripts (which I think was
an older method), Gamebanana mods with no source code, etc. It felt a little hopeless,  that was until
I stumbled upon the [Castle Crashers Wiki Discord](https://discord.gg/VVDrtWq).
I dug through the modding text channels which lead me to the [Dodecafunctional Reorbanizations Discord](https://discord.gg/Y5DP25g2BT), this is where I really
struck gold.

## The Method
> I don't cover how to decrypt the MAIN.swf but the
unencrypted file as well as Ethteck's Decryption tool can be found via the resources mentioned.
{: .prompt-info}

From the getting-started channel, I was introduced to the main tools:
- [JPEXS Free Flash Decompiler](https://github.com/jindrapetrik/jpexs-decompiler)
- MAIN.swf
- Ethteck’s Decryption Script

Castle Crashers' game logic is written in ActionScript2 stored within the MAIN.swf file under scripts/frame 2/DoAction. The MAIN.swf is in an encrypted state
as main.pak in the "\Steam\steamapps\common\CastleCrashers\data\game" directory for the Steam version.

First I opened the MAIN.swf with the JPEXS Free Flash Decompiler. This tool decompiled the file into editable source code. Next, in the 
left-side panel, I navigated to MAIN.swf/scripts/frame 2/DoAction. Then by pressing the edit ActionScript button in the center panel, I could edit the ActionScript.

I used various keywords and knowledge of the game to find what I needed to both edit and use. I tried "exp", "xp", "experience", etc until I found the function "f_Exp". This function is responsible for calculating the experience gain, granting the experience to the player and determining any level ups. This was a good start, but it was not responsible for the experience distribution- which I knew was from hitting enimies. So I continued searching to find where
in the code the "f_Exp" was called. This lead me to the function "f_Damage". This
function was not only responsible for the damage calculations but also for the
experience distributed on hit.
```as
function f_Damage(zone, damage_pow, damage_type, damage_flags, recoil_x, recoil_y)
{
  ...
  if(zone.health > 0)
     {
        if(zone.hitby)
        {
           if(zone.hitby.human)
           {
              f_Exp(zone.hitby,1);
           }
           else if(zone.hitby.owner.human)
           {
              f_Exp(zone.hitby.owner,1);
           }
        }
     }
 ...
}
```
This was great, I was already half way there. The "f_Exp" is called when the
enemy is hit by you (zone.hitby.player) or your pet (zone.hitby.owner.human). Now if I can edit this to distribute to all players instead, I would be done. There are only two problems.
1. How do I reference all players?
2. How do I keep the total experience distributed the same?

### 1. How do I reference all player?
To distribute to all players I had to find some way to iterate over all player objects. So I went back through the code and searched for "player". Though this  I managed
to find an "active_players" global variable which stores the number of players
in the game session. This is good because I now have a stop condition for a for
loop. I use this variable to search for any code that already iterates over players. Although I didn't find any code like that I did find the function "f_PlayerArray". This function initialises the object "playerArrayOb" that stores the instance of all player objects. Exactly what I wanted. To reference a specific player, player 1-4, I have to call "playerArrayOb["p_pt" + i]"  where i is the player's number from 1-4. I had all I needed so I  went back to the "f_Damage" function and updated it to the following:
```as
function f_Damage(zone, damage_pow, damage_type, damage_flags, recoil_x, recoil_y)
{
  ...
  if(zone.hitby)
      {
         if(zone.hitby.human | zone.hitby.owner.human)
         {
            for (var i = 1; i <= active_players; i++) 
            {
               f_Exp(playerArrayOb["p_pt" + i], 1);
            }
         }
      }  
  ...
}
``` 
This code change now distributes the experience to all players independent on
who does the damage. However, this has a problem; now everyone is getting 1 exp-
thus the new total experience will be number of players * original total exp.
This will likely cause an unintended power creep. So how do I keep the total
experience distributed the same?

### 2. How do I keep the total experience distributed the same?
My initial thought was, "Oh I can just change the experience given from 1 to
1 / number of active players." This seemed like a good solution but I didn't
know if the parameter supported floating point numbers, and I wanted to avoid
introducing unpredictable floating-point errors. So that was out. The next thing I considered was to change experience required per level dynamically depending on the number of players. However, this would've also caused unintended problems, because the experience required is bound to the character itself. This means that if you were to use this character in different lobbies where the number of players change then the XP table for the character would be completely messed up. Instead I had a different idea. Since the experience distribution was only dependent on the number of active players, I would define a global variable at the same time when "active_players" was defined called "total_hits". Then when the game would grant exp, it would first check to see if the number of hits is divisible by the number of active players then give all of the players 1 exp. This way the total experience would remain the same regardless of the number of players. To achieve this I edited the previous logic as follows:
```as
function f_Damage(zone, damage_pow, damage_type, damage_flags, recoil_x, recoil_y)
{
  ...
  if(zone.hitby)
      {
         if(zone.hitby.human | zone.hitby.owner.human)
         {
            total_hits = total_hits + 1;
            if(total_hits % active_players == 0)
            {
               for (var i = 1; i <= active_players; i++) 
               {
                 f_Exp(playerArrayOb["p_pt" + i], 1);
               }
            }
         }
      } 
  ...
}
```
```as
function f_PlayerArray()
{
   ...
   active_players = 0;
   total_hits = 0;
   ...
}
```
With these changes made, I saved the file. The next step was to use the Ethteck's Decryption Script to encrypt the MAIN.swf back into the main.pak file. To do this, I located the directory in which I unpacked the script in my terminal. Then I ran the following command.
```bash
python3 decrypt_asset.py --encrypt /path/to/MAIN.swf output
```
This generated the main.pak into a folder named output in the same directory. All that was left to do was to install the mod by replacing the main.pak in the game directory "\Steam\steamapps\common\CastleCrashers\data\game"  with the main.pak generated.
