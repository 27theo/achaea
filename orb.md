Orb management
=====

Alias to disable automatic orb management.
--

Pattern: `^auto orb (on|off)$`

Script:
```lua
ignoreOrb = matches[2] == "off"
```

First trigger: Reactivating the orb if you're in city.
--

Patterns (all perl regex type):
```pl
^\(Army\): \w+ says, "I've deactivated the city's orb of confinement\!"$
^\(Army\): \w+ says, "I've deactivated the city's Orb of Confinement\!"$
^\(Army\): \w+ says, "I have deactivated the Orb of Confinement\."
^\(Army\): \w+ says, "I have deactivated the orb of confinement\."
```

Trigger script:
```lua
if gmcp.Room.Info.area == "Targossas" and (not ignoreOrb) then
  echo("\n")
  cecho("<yellow>(going to reactivate orb in 5...)")
  tempTimer(5, function()
    send("queue addclear eqbal city improvement activate orb")
  end)
end
```

Second trigger: Reporting that you've activated the orb.
--

Pattern (exact match type):
```
You have activated the city's Orb of Confinement.
```

Trigger script:
```lua
send("art I have activated the Orb of Confinement")
```

Third trigger: Reporting that you've deactivated the orb.
--

Pattern (exact match type):
```
You have deactivated the city's Orb of Confinement.
```

Trigger script:
```lua
send("art I have deactivated the Orb of Confinement")
```
