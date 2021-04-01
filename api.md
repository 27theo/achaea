# Easy API info

## The Code

1. **Copy the following into a new script:**

```lua
function apiGetInfo(name)
  name = name:trim():lower():title()
  downloadFile(getMudletHomeDir().."/char.json", "http://api.achaea.com/characters/"..name..".json")
end

function apiReceivedInfo(_, file)
  if not file:find("char.json") then return end
  local f = io.open(file)
  if f then 
    s = f:read("*l"):trim()
    local t = yajl.to_value(s)
    echo("\n")
    display(t)
    echo("\n")
    io.close(f)
  else
    cecho("<red>(!) <white>API download failed.\n")
  end
end

registerAnonymousEventHandler("sysDownloadDone", "apiReceivedInfo")
```

2. **Make a new alias:**

```lua
-- Pattern: ^api (\w+)$
apiGetInfo(matches[2])
```

## Usage

```
  api <name> 
    downloads info for given name and displays it.
```
