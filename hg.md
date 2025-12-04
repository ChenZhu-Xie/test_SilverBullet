


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
  name = "Navigate: Table Picker",
  key = "Ctrl-Shift-t",
  priority = 1,
  run = function()
    local tables = getTables()
    if not tables or #tables == 0 then
      editor.flashNotification("No tables found.")
      return
    end

    local sel = editor.filterBox("Jump to", tables, "Select a Table...", "Page @ Pos where the Table locates")
    if not sel then return end
    editor.navigate(sel.ref)
    editor.invokeCommand("Navigate: Center Cursor")
  end
}

function getTables()
  local rows = query[[
    from index.tag "table"
    select {
      name = string.format("%s @ %d", _.page, _.pos),
      ref      = _.ref,
      tableref = _.tableref,
    }
    order by _.page, _.pos
  ]]

  local out, seen = {}, {}
  for _, r in ipairs(rows) do
    local key = r.tableref
    if not seen[key] then
      seen[key] = true
      table.insert(out, r)
    end
  end
  return out
end
```
