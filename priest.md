# Scripting a basic manual priest offence (Mudlet)

This is a summary of the techniques with which I recommend making a starter priest offence. 

The final result is:
1. Simple to adapt over time.
2. Efficient.
3. Easy to incorporate some automation into.

Note: This is not a guide on _how_ to fight as a Priest. Ask around in-game for guides on strategy and kill paths.

------

### Pre-requisites

1. Transcendence in all three of the Priest class skills, or at the very least absolve in spirituality, inquisition in devotion, and purge in zeal.
2. Basic familiarity with priest attacks. Which afflictions verses give, how chasten works, etc.
4. The mind warden trait is tremendously helpful.
5. A server-side command separator[^1], set with `CONFIG COMMANDSEPARATOR`. I will use `|` in examples.
6. A target variable, and an alias that sets it.

Here's an example of a target-setting alias.
```lua
-- New alias: Set target
-- Pattern: ^t (.+)$

send("st " .. matches[2]) -- Set your target game-side
target = matches[2]:lower():title() -- It is good convention that your target variable is case-sensitive.
```

You don't necessarily need basic scripting knowledge to follow this guide, but a little bit goes a long way when it comes to understanding the code, even if vaguely.

Online Lua/Mudlet documentation is useful, but the [official Achaea discord](https://discord.gg/2v2upFTj8G) is likely the best place to seek coding help. 

## Attacks

### Queueing

Queueing is the method by which you tell the game to send a command when you meet certain conditions.

These conditions are best explained by `HELP QUEUEING`, in game. 

Here are some of the most common queue conditions:

```lua
-- Commands will fire when we have eq, bal, and are unhindered.
queue addclear free kill rabbit
```

```lua
-- Commands will fire when we have class balance and no stun.
-- This is used by classes with a tertiary balance that can be used without eq and bal (not priest).
queue addclear c!t recite epic at rabbit
```

```lua
-- Commands will fire when we have eq, bal, class bal, and are unhindered.
queue addclear full recite epic at Romaen|kill Romaen
```

In the vast majority of cases, this last queue format is ideal for priest. 

### Pre-queue

Sometimes referred to as a pre-alias, the pre-queue contains commands that we want to run just after we get balance back, but just before we attack.

We have no pre-queue in above examples.

Actions that people tend to include in their pre-queue:

1. `touch stuff` to clear amnesia.
2. `concentrate` to cure hidden disrupt.
3. `parry X` to parry a limb.
4. `wield weapon shield` to re-wield or change weapons.

With those, a command would normally[^2] take this format:

```
queue addclear full touch stuff|concentrate|parry right leg|wield mace111111 shield111111|<atk>
```

Some people grab gold, grab corpses, wear armour, etc.

At the time of writing, the vast majority still include stand in the pre-queue. This is no longer necessary[^3].

### Example attacks

Here are some example attacks that our alias set-up will send.

We queue:
1. The pre-queue.
2. The attack.
3. Contemplate.

```
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite guilt Romaen|smite Romaen right leg chasten paralysis|contemplate Romaen
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite penance Romaen|smite Romaen chasten mind|contemplate Romaen
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite penance Romaen|smite Romaen right leg chasten body|contemplate Romaen
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite purge Romaen|smite Romaen right leg|angel push Romaen|contemplate Romaen
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite penance Romaen|perform dazzle Romaen|contemplate Romaen
```

## Scripting the queue

### Setting the queue

We want one [function](https://www.lua.org/pil/5.html) to queue all of our attacks.

This is to be placed in a new script.

```lua
function queue(attacks)
  local prequeue = "touch stuff|parry right leg|wield mace111111 shield111111"
  local sep = "|"
  send("queue addclear full " .. prequeue .. sep .. table.concat(attacks, sep) .. sep .. "contemplate " .. target)
end
```

With `function queue(attacks)`, we define our function, specifying that it takes one [argument](https://www.idtech.com/blog/what-is-an-argument-in-programming#:~:text=Argument%20definition,argument%2C%20also%20called%20a%20parameter.), a table of attacks.

We create a local (temporary) variable `prequeue` that contains the pre-queue for neatness.

We create another local variable `sep` that contains our command separator, also for neatness.

In the send:
* We start the queue commmand
* Append the pre-queue
* Append the table of attacks, concatenated using [`table.concat`](https://wiki.mudlet.org/w/Manual:Table_Functions#table.concat), using `sep` as the separator.
    - `table.concat({"one", "two", "three"}, "+")` would result in the string `"one+two+three"`.
* Append a contemplate at the end

All split up by our command separator.

Adding some automation to this function can go a long way. A few notable improvements include:
* Automatic parry selection.
* Wield detection/switching weapons based on arm breaks.
* Pre-queue adjustment under stupidity.

### Using the `queue` function

We will use the queue function like so:

```lua
queue({"recite penance " .. target, "smite " .. target .. " right leg chasten body"})
-- Sends: queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite penance Romaen|smite Romaen right leg chasten body|contemplate Romaen
```

```lua
queue({"recite purge " .. target, "smite " .. target .. " right leg", "angel push " .. target})
-- Sends: queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite purge Romaen|smite Romaen right leg|angel push Romaen|contemplate Romaen
```

## Aliases

### Main attack alias

We can use [regex](https://www.tutorialspoint.com/perl/perl_regular_expressions.htm) and [tables](https://www.tutorialspoint.com/lua/lua_tables.htm) to make a very powerful alias.

Create a new alias.

```lua
-- New alias.
-- Pattern: ^(\w)(hh|ra|tt|la|rl|ll|nn)$

-- The combo table contains affliction combinations.
-- The key of each entry is the letter that corresponds to the combination. So "g" corresponds to its own sub-table.
-- The first entry of the sub-table is the zeal verse, and the second is the chasten selection. For "g", the zeal verse is "guilt", and the chasten selection is "paralysis".
local combo = {
  g = {"guilt", "paralysis"},
  s = {"ash", "paralysis"},
  t = {"burn", "paralysis"},
  j = {"condemnation", "paralysis"},
  b = {"purge", "paralysis"}, 
  -- I use b for purge (for burning) because I use p for paralysis in my break/angel push alias, where purge/burning needs to be an option.
  
  o = {"penance", "body"},
  m = {"penance", "mind"},
  
  a = {"penance", "asthma"},
  c = {"penance", "clumsiness"},
  l = {"penance", "healthleech"}, -- h is taken
  n = {"penance", "nausea"},
  
  e = {"penance", "epilepsy"},
  d = {"penance", "dizziness"},
  f = {"penance", "confusion"}, -- c is taken
  r = {"penance", "recklessness"},
}

-- The limb table extends the shorthand limb selection into its full name.
-- "nn" will translate to an untargetted smite, hitting no limb.
local limbs = {
  hh = "head",
  tt = "torso",
  ra = "right arm",
  la = "left arm",
  rl = "right leg",
  ll = "left leg",
  nn = "",
}

-- The rebuke could be chosen in the alias pattern, ^(\w[mo]?)(hh|ra|tt|la|rl|ll|nn)(d|e|c)$, but it's more effort than it's worth.
-- Alternatively, the rebuke could be automatically chosen using an affliction tracker program, like AK. If they don't have masochism, rebuke evil, etc.
-- However, remember that you might not want to rebuke after using angel push. A little problem each solve differently!

-- At the end of the day, always using darkness is absolutely fine.
local rebuke = "darkness"

if combo[matches[2]] then -- Check if the first part (one/two letters) of the alias is in the combo table.
  local zeal = combo[matches[2]][1] -- Pick the first entry in the chosen sub-table as the zeal verse.
  local chasten = combo[matches[2]][2] -- Pick the second entry in the chosen sub-table as the chasten selection.
  queue({ -- We can give each entry in the attack table its own line for readability.
    "recite " .. zeal .. " target rebuke " .. rebuke,
    "smite " .. target .. " " .. limb .. " chasten " .. chasten
  })
end
```

### Use cases

The first letter in the pattern specifies the affliction combination. 

Some players would specify each affliction with its own letter, but in practice the majority of priest attacks contain either penance or paralysis, the other affliction being the one you want to stick.

So, we can map one letter to two afflictions using the combo table. `g` to guilt and paralysis, `m` to penance and mind, etc. In my experience, this is wildly preferable.

`ghh` translates to
```
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite guilt Romaen|smite Romaen head chasten paralysis|contemplate Romaen
-- Guilt zeal, head limb, paralysis chasten.
```

`pra` translates to
```
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite purge Romaen|smite Romaen right arm chasten paralysis|contemplate Romaen
-- Purge zeal, right arm limb, paralysis chasten.
```

`rnn` translates to
```
queue addclear full touch stuff|parry right leg|wield mace111111 shield111111|recite penance Romaen|smite Romaen chasten recklessness|contemplate Romaen
-- Penance zeal, no limb, recklessness chasten.
```

It will likely take time to get used to the affliction combinations, but it's worth it. Practice makes perfect.

### More aliases

You'll want an alias for angel push, for when you want to prone.

```lua
-- New alias.
-- Pattern: ^(\w+)p(hh|ra|tt|la|rl|ll|nn)$

local aff = {
  p = "penance",
  b = "purge",
}

local limbs = {
  hh = "head",
  tt = "torso",
  ra = "right arm",
  la = "left arm",
  rl = "right leg",
  ll = "left leg",
  nn = "",
}

if aff[matches[2]] then 
  local zeal = aff[matches[2]]
  queue({ 
    "recite " .. zeal .. " target, -- We don't want to rebuke, to save for wrath/sap.
    "smite " .. target .. " " .. limb, -- No chasten, so that we can angel push.
    "angel push " .. target
  })
end
```

And you'll want aliases for devotion attacks, like dazzle:

```lua
-- New alias.
-- Pattern: ^da$

queue({
  "recite penance " .. target .. " rebuke darkness",
  "perform dazzle " .. target
})
```

And inquisition, in which we need to select an affliction:

```lua
-- New alias.
-- Pattern: ^(g|s|p|b)inq$

local aff = {
  g = "guilt",
  s = "ash",
  p = "penance",
  b = "purge"
}

queue({
  "recite " .. aff[matches[2]] .. " " .. target .. " rebuke darkness",
  "perform inquisition " .. target
})
```

You can (and arguably should) also use the `queue` function for other commands you'll use in fights.

```lua
-- New alias.
-- Pattern: ^fly$

queue({"fly"})
```

The possibilities are endless. 

-----

If you have any questions, I will respond on Discord at `theo#7545`. 


[^1]:NOT a client-side command separator, as is supported by Mudlet. Your client-side and server-side command separators work differently, and in turn should be different.
[^2]:Some people eliminate as much of the pre-queue as possible with stupidity, to decrease the chance of a proc that disrupts your offense (the SECRETS command can fire, most infamously).
[^3]:There used to be a short delay in between regaining balance and server-side curing automatically standing. Short, but long enough to matter. This is no longer the case as of November 2022.





