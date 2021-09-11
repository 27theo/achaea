# Mudlet Priest Offense

Hi all. I thought I'd make this scroll to help newcomers to the Priest class set up the basic scripts you need to get fighting. Setting up an offense is only the first step, of course - practice and familiarity with your class and others are equally if not more important. 

This isn't meant to be a scroll detailing _how_ to fight as a Priest (what afflictions to give, when to give them etc). If it ends up useful in that regard then great, but if it seems scarce on tactics and such, that's why.

------

### 0.1) PREREQUISITES

If you aren't transcendent in any of the three priest skills, I could be discussing abilities you don't have. Bear that in mind if you end up testing the aliases.

Assuming you have the abilities, you need to be somewhat familiar with what key attacks do. These include which zeal verses give afflictions and what, how chasten works, et cetera.

Before following this guide, you need to have a way of setting your target variable. An example alias might look like this:

```lua
-- Pattern: ^t (.+)$

send("st " .. matches[2])
target = matches[2]:lower():title()
-- It's always a good idea to make sure that the target variable contains the target's name in the right case.
-- This way, you can check if a name from a trigger == target without worrying about case.
```

I'm trying to keep it simple, but you'll still need some scripting knowledge. If you want to script your own profile, starting from zero experience is long and hard.

## 1) Aliases

### 1.1) The Very Basics

For the uninitiated, aliases are how almost everybody in Achaea send their attacks. A basic example might be an alias that sends the following:

```
setalias pk stand/wield mace shield/smite &tar/contemplate &tar
queue addclear eqbal pk
```

Take a look at the example alias. It starts with "stand/wield mace shield", or what's commonly called a PREALIAS. This can contain any number of things that cost no balance to do, like "take body" to pick up corpses, "touch stuff" to clear amnesia, or "take sovereigns/put sovereigns in pack" to pick up gold and store it away. What you include is up to you, but stand and wield are must-haves!

Then, the alias contains a smite. The &tar means that the smite will hit your game target, set with the SETTARGET command (or ST for short). 

It finishes off with a contemplate. This checks the mana of your opponent, but is only possible with the Mind Warden trait. You _must_ have this trait.

For those unfamiliar with queueing, "queue add eqbal <command>" tells Achaea to run your command when next have both equilibrium and balance. Here, we add the alias we've created to the queue.

### 1.2) Typical Attacks
 
Here are some attacks that I store in my pk alias, without my prealias.

```
stand/parry left leg/wield mace shield/recite guilt &tar rebuke darkness/smite &tar right leg chasten paralysis/contemplate &tar 
(guilt + paralysis)

stand/parry right leg/wield mace shield/recite penance &tar rebuke darkness/smite &tar left arm chasten body/contemplate &tar
(paralysis + chasten body)

stand/parry left leg/wield mace shield/recite penance &tar rebuke darkness/perform dazzle &tar/contemplate &tar 
(paralysis + stupidity)

stand/parry left leg/wield mace shield/recite penance &tar rebuke darkness/smite &tar left leg/angel push &tar/contemplate &tar 
(paralysis + prone)

stand/parry left leg/take handaxe/wield handaxe/throw handaxe at &tar gecko/contemplate &tar 
(slickness - need a handaxe proficiency)
```

Et cetera. Note that for every attack, I rebuke. This is ok to do because if you can't rebuke, you just don't. The base verse still goes through.

### 1.3) Setting Your Alias
 
In my opinion, the best way to set your attack alias is with a function. Here's an example.

```lua
-- NEW SCRIPT:

function combatAlias(str)
  if not target or target == "" then return end 
  -- This ensures that the function doesn't run if you have no target.
  
  sendAll("setalias pk touch stuff/stand/wield mace shield/" .. str .. "/contemplate &tar",
          "queue addclear eqbal pk")
end

-- NEW ALIAS:
--  pattern: ^sgp$

combatAlias("recite guilt &tar rebuke darkness/smite &tar chasten paralysis")

```

The function first checks if the target variable is set, and that you have a target. If you don't, it returns, exiting the function. Then, the function sends two commands to the game. First, it sets the alias "setalias pk ...", and then it queues it "queue addclear...".

This way, if you want to change your prealias, you only have to change it in one place - the function.

Here's an alternative method, using tables:

```lua
-- NEW SCRIPT:

function combatAlias(tab)
  if not target or target == "" then return end
  
  sendAll("setalias pk touch stuff/stand/wield mace shield/" .. table.concat(tab, "/") .. "/contemplate &tar",
          "queue addclear eqbal pk")
end

-- NEW ALIAS:
--  pattern: ^sgp$

combatAlias({"recite guilt &tar rebuke darkness", "smite &tar chasten paralysis"})

```

This is very similar to the other combatAlias function, but it takes a table as an argument. A table is a collection of values, like so:

```lua
{"first value", "second value", "and the third"}
```

This function takes the table as an input, and concatenates (table.concat) it into a string, joining it with a "/" character. Therefore, the results of both the example aliases will be the same.

**This (second version) is the combatAlias function I'll be using in examples. I prefer it for reasons I'll detail below.**

### 1.4) The Artful Master Alias

The following technique is how I like to send attacks. As opposed to creating individual aliases, each with their own combatAlias(blah), many aliases can be combined into one.

```lua
-- NEW ALIAS:
--  pattern: ^(\w\w?)(hh|tt|ra|la|rl|ll|nn)$



local combo = {
  g = {"guilt", "paralysis"},
  s = {"ash", "paralysis"},
  t = {"burn", "paralysis"},
  j = {"condemnation", "paralysis"},
  
  o = {"penance", "body"},
  m = {"penance", "mind"},
  
  a = {"penance", "asthma"},
  c = {"penance", "clumsiness"},
  l = {"penance", "healthleech"},
  n = {"penance", "nausea"},
  
  e = {"penance", "epilepsy"},
  d = {"penance", "dizziness"},
  f = {"penance", "confusion"},
  r = {"penance", "recklessness"},
  
  gm = {"guilt", "mind"},
  sm = {"ash", "mind"},
  tm = {"burn", "mind"},
  jm = {"condemnation", "mind"},
  
  go = {"guilt", "body"},
  so = {"ash", "body"},
  to = {"burn", "body"},
  jo = {"condemnation", "body"},
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

if combo[matches[2]] then
  combatAlias({"recite " .. combo[matches[2]][1] .. " &tar rebuke darkness",
               "smite &tar " .. limbs[matches[3]] .. " chasten " .. combo[matches[2]][2]})
end

```

Let's talk about the pattern, first. This will match any one or two letters, followed by hh, tt, ra, la, rl, ll, or nn. The last two letters correspond to limbs, as shown in the limbs table. "nn" means no limb, so the smite will just be an untargetted one.

The first letter, or the first two letters, correspond to the afflictions you'll deliver. These are converted using the combo table.

Take a look at "g", for example. You can see it at the top of the combo table, and the afflictions it corresponds to are guilt and paralysis. A standard guilt smite.

The first word after each combination is what will be recited at the target, or the verse. So, for "a", penance will first be recited, afflicting paralysis. It's important to use the verse name, not the affliction name - recite ash, for example, as opposed to recite spiritburn (the affliction that the ash verse gives), which will not work.

The second word in the table for each combination is what will be chastened. Looking at "a" again, we can see that we'll chasten asthma.

So, if I were to use the alias "arl", I would:
 - recite penance rebuke darkness
 - smite &tar right leg chasten asthma

Or, if I were to "tmnn", I would:
 - recite burn rebuke darkness
 - smite &tar chasten mind (no limb!)
 
The reason I'd advise an alias like this is because it squishes 23 into one. If you were to make individual ones for each limb, you'd have 161. It also makes adding new affliction combinations extremely easy. Just add them to the combo table. 

Of course, this just covers standard attacks. Some examples of others I have aliases for include:
 - bp(limb), which breaks limb and prones opponent
 - da, ano, wea, for PERFORM DAZZLE, PERFORM FASTING, PERFORM SLOTH


### 1.5) Advanced Alias Function

Say, for example, I'm using the above master alias, but have another script controlling whether or not I want to rebuke. Perhaps if the target has broken legs, or if I'm currently breaking those legs, I don't want to rebuke so that I have enough prayer length for wrath.

For our example, the script will change the `noRebuke` variable to false if I want to rebuke, and true if I don't. I can change my combatAlias function to account for this.

```lua
function combatAlias(tab)
  if not target or target == "" then return end
  
  if noRebuke then -- (if not meant to be rebuking)
    for k, v in pairs(tab) do 
      tab[k] = v:gsub("rebuke %a+", "")
    end
  end
  
  sendAll("setalias pk touch stuff/stand/wield mace shield/" .. table.concat(tab, "/") .. "/contemplate &tar",
          "queue addclear eqbal pk")
end
```

There. Our combatAlias function now removes any rebukes from the attack if noRebuke is true. Because we're using the table version of combatAlias, we can easily iterate through the commands, and check the contents of all of them.

I've found a lot of uses for these substitutions. Here's a snippet from my function:

```lua
-- My prea variable contains my prealias. By default, it's "stand/parry left leg/wield mace shield447030/take sovereigns/put sovereigns in pack/take body/".
if aff.list.damagedleftarm or aff.list.brokenleftarm or aff.list.mangledleftarm then
  prea = prea:gsub("wield mace shield447030", "wield shield447030 mace")
end
```

My prea variable contains my prealias. If I can't wield my mace because of limb damage on my left arm, I'll swap out "wield mace shield" for "wield shield mace", therefore wielding my mace in my right hand, with which I can keep on smiting.

Here's another:

```lua
if table.contains(rtr.items.list, "a throwing axe") then 
  prea = prea .. "take axe/"
end
```

If there's a throwing axe in my room (which I probably threw), then I'll pick it up as part of my prealias. 

```lua
local select = {"right leg/", "left leg/"}
prea = prea .. "parry " .. select[math.random(1, #select)]
```

Random parry.

The possibilities are endless, and this is very useful. Monoliths, breaking shields, tentacle, stopping earring with a prone, et cetera.

## 2) Gags

These aliases are noisy. Here are lines that I gag, for your ease of use.

```lua
-- Regex type: (blue)
^Alias "pk" will now execute: ".+"$
^\[System\]: Added (?:QUEUE ADDCLEAR CLASS )?PK to your \w+ queue\.$
^\[System\]: Queued (\w+) commands cleared\.$

-- Script:
deleteFull()
```

```lua
-- Exact match: (green)
You hold no "sovereigns".
I see no "sovereigns" to take.
I see no "body" to take.
You are not fallen or kneeling.
You may not focus on contemplating the mental state of someone again quite yet.
You hold no "monolith".
You already possess equilibrium.


-- Start of line: (red)
You are already wielding

-- Script:
deleteLine()
```

## 3) Triggers

You need to be able to tell what % an opponent's mana is at, because our mana damage works in percentages, and you must notice when the opponent's mana is below 50% so that you can absolve. The following will 

```lua
-- Regex type: (blue)
^(\w+)\'s mana stands at (\d+)\/(\d+)\.$

-- Script:
if target == matches[2] then
  local mp = matches[3]
  local max = matches[4]
  cecho(" <firebrick>(<white>" .. math.floor((mp/max)*1000)/10 .. "%<firebrick>)")
  if mp/max < 0.5 then 
	cecho("\n    <cyan>--- ABSOLVE READY! ABSOLVE READY! ---")
  end
end
```

Good idea to highlight when your target cures guilt, too.

```lua
-- Regex type: (blue)
^(\w+) straightens, as if some great burden had been lifted from \w+ shoulders\.$

-- Script:
if target == matches[2] then 
  selectCurrentLine()
  fg("white")
  bg("ansi_yellow")
  resetFormat()
  deselect()
end
```

Triggers are relatively simple to set up, and how may use or if you really use any at all is up to you, so I'll leave it at that.

## 4) External Scripts

There are two main scripts that I use alongside my manual functions.

 1. AK Affliction Tracking

I don't use this to automate my affliction choice, because I like having full control. I do, however, display my target's afflictions in a box to the right of the main console, along with the probability that they have them. 

 2. My Limb Counter (shameless plug)

In my opinion, some form of limb counter is a must. You can use AK for limb tracking, but it's overcomplicated and doesn't work with the in-game limb damage percentages.

I display limb data in the same console that I do afflictions. [Here is a visual demonstration which I uploaded to YouTube](https://www.youtube.com/watch?v=2fTlgb2yq3Q).

------

I hope this is somewhat helpful. I'd like to reiterate that this is just a starting guide on how to set up your profile, and is by no means some form of guarantee that if you make these aliases and copy this code that you'll start winning fights. This is just what I do, which I decided to share in case it helps somebody. If you have any questions, shout me in game (Romaen) or on discord (theo#7545). 
