


sadfasdf

test


- asdf
  -  asdfsadf
  -  - [[la1🟣|]]==1== ➡️ ${forthRef("la1")}${backrefStat("la1")}*~Σ~*

1. asdfasdf
  2. asdfasdf
  3. | Header A | Header B |
|----------|----------|
| Cell A[[la1🟣|]]==2== ➡️ ${forthRef("la1")}${backrefStat("la1")}*~Σ~* | Cell B |





```space-lua
command.define {
  name = "Navigate: File Link Picker",
  key = "Alt-f",
  priority = 1,
  run = function()
    local FileLinks = getFileLinks()
    if not FileLinks or #FileLinks == 0 then
      editor.flashNotification("No File Links found.")
      return
    end
    
    local sel = editor.filterBox("🔍 Select", FileLinks, "Choose a File Link...", "a File Link to GoTo")
    if not sel then return end
    editor.navigate(sel.ref)
    editor.invokeCommand("Navigate: Center Cursor")
  end
}

function getFileLinks()
  return query[[
    from index.tag "link"
    where _.toFile
    select{
      name = _.snippet,
      description = string.format("%s @ %d", _.page, _.pos),
      ref = _.ref,
    }
    order by _.page, _.pos
  ]]
end
```
