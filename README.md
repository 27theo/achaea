# My Mudlet Packages for Achaea.

> :warning: See above for downloads. Use these, not the links in my forums posts. I'm trying to maintain both the dropbox files and this repository, but this repository will always be updated first. 

### riftCheck
- Check the contents of your rift against a list of all herbs, minerals, elixirs, salves, venoms, and inks.
- Easily identify which curatives, if any, you're nearly out or out of.
- Script contains configuration tables, easily modified to, in turn, modify the output.
  - Exclude or include certain categories of curative.
  - Only check important herbs, minerals, or venoms.
  - Change what herbs, minerals, and venoms count as important.
  - Changeable threshold values for colour coding.

Here's the [forums post](forums.achaea.com/discussion/7462/mudlet-rift-checker-easily-tell-what-youre-running-out-of) I announced it in.

```python
Usage:
    riftcheck -> run the script.
```

--------

### killCount
 - Kill tracker, for counting both:
   - Number of times you kill somebody/something.
   - Number of times you've been killed by them/it.
 - Allows for a simple search. 

Here's the [forums post](forums.achaea.com/discussion/7438/killcount-kill-tracker-for-mudlet) I announced it in.

```python
Usage:
    kc all            -> display all data.
    kc reset          -> reset all data.
    kc <name>         -> show killed by and deaths to name. substring search.
    kc add <name>     -> add name to the table.
    kc remove <name>  -> remove name from the table.
    
    kc name <k|d> <modifier> -> modify name's "killed by you" or "deaths to" count by the given modifier. this is for manual modifications.
                                e.g. "kc Shirim d +1" (this would increase the number of times you've killed Shirim by one.)
                                     "kc Cala k -4" (this would decrease the number of times you've died to Cala by four.)
                                     
    kc help           -> see this output.
```

--------

### daybox
 - Graphical time of day tracker.
 - An ASCII art alternative to the tracker built into Nexus.
 
 Here's the [forums post](forums.achaea.com/discussion/7403/graphical-day-night-tracker-mudlet-alternative) I announced it in.
 
--------

### RTrack
 - Easy GMCP tracking of players, afflictions, defences, and room items.
 - See forums post for more details.

> :warning: This script contains inaccuracies, and is no longer updated. It's here because it's still useful, and the mistakes are minor.

Here's the [forums post](forums.achaea.com/discussion/7397/rtrack-easy-gmcp-tracking-of-players-affs-defs-and-room-items) I announced it in.

```python
Usage:
    Settings are in the script "rt config [CONFIGURE HERE]" and "rt echo [CONFIGURE HERE]" for outputs.
    Players, afflictions, defences, and room items are kept in tables.
      rtr.players.list
      rtr.affs.list
      rtr.defs.list
      rtr.items.list
      rtr.items.ids
    Upon modifying any of the tables, RTrack will fire an event. Event names are detailed in the first config script.
```
