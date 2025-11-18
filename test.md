
















some random text then suddenly insert a [[your label 1🔵|]]${backrefStat("your label 1")}*~Σ~* 🔙 ${backRefs("your label 1")} keep input any random words

lkasdflksdajflkj [[your label 1🟣|]]==1== ➡️ ${forthRef("your label 1")}${backrefStat("your label 1")}*~Σ~* asdlkfsadlkfsadjkf

l;askdjflksjadf [[your label 1🟣|]]==2== ➡️ ${forthRef("your label 1")}${backrefStat("your label 1")}*~Σ~* but you will adapt to it and like it if you .... keep using this.... or understand the mechanism behind



```
function usrPrompt(hinText)
  local input = editor.prompt(hinText, "")
  if not input then
    editor.flashNotification("Cancelled", "warn")
  end
  return input
end

local suffixFlabel = "🔵" -- 🐳💧 ➖ 🗨
local suffixBlabel = "🟣" -- 🐈‍⬛🍆 ➕ 🗯
local F = " ➡️ " -- >> » 🔜 🢧
local B = " 🔙 " -- << « 🔙 🡄

-- =========== Forth Anchor + Back Refs ==================

local function tableBack(Flabel)
  local aspiringPageBack = Flabel .. suffixBlabel
  return query[[
    from index.tag "link"
    where toPage and toPage:find(aspiringPageBack, 1, true) -- no Regex
    order by _.thBlabel
    select {ref=_.ref, thBlabel=_.thBlabel}
  ]]
end

function backrefStat(Flabel)
  return #tableBack(Flabel)
end

function backRefs(Flabel)
  -- local str = template.each(tableBack(Flabel), template.new[==[​[[${_.ref}]]​^${_.thBlabel}^​]==])
  local str = template.each(tableBack(Flabel), template.new[==[​[[${_.ref}]]​==${_.thBlabel}==​]==])
  if #str == 0 then return "No BackRef" end
  return str
end

command.define {
  name = "insert: Forthanchor + Backrefs",
  key = "Ctrl-,",
  run = function()
    local Flabel = usrPrompt('Enter: label (to be Referred)')
    if not Flabel then return end
    local aspiringPageForth = Flabel .. suffixFlabel
    local forthAnchor = "[[" .. aspiringPageForth .. "||^|]]"
    local backrefStat = '${backrefStat("' .. Flabel .. '")}*~Σ~*'
    local backRefs = '${backRefs("' .. Flabel .. '")}'
    local fullText = forthAnchor .. backrefStat .. B .. backRefs
    editor.insertAtPos(fullText, editor.getCursor(), true)
  end
}

-- =========== Back Anchor + Forth Ref ==================

local function tableForth(Flabel)
  local aspiringPageForth = Flabel .. suffixFlabel
  return query[[
    from index.tag "link"
    where toPage == aspiringPageForth
    select {ref=_.ref}
  ]]
end

function forthRef(Flabel)
  local str = template.each(tableForth(Flabel), template.new[==[​[[${_.ref}]]​]==])
  if #str == 0 then return "No such Anchor" end
  return str
end

command.define {
  name = "insert: Backanchor + Forthref",
  key = "Ctrl-.",
  run = function()
    local Flabel = usrPrompt('Jump to: label')
    if not Flabel then return end
    local aspiringPageBack = Flabel .. suffixBlabel
    local backAnchor = "[[" .. aspiringPageBack .. "||^|]]"
    -- local thBlabel = "^" .. (tableBack(Flabel)).length + 1 .. "^"
    local thBlabel = "==" .. #tableBack(Flabel) + 1 .. "=="
    local forthRef = '${forthRef("' .. Flabel .. '")}'
    local backrefStat = '${backrefStat("' .. Flabel .. '")}*~Σ~*'
    local fullText = backAnchor .. thBlabel .. F .. forthRef .. backrefStat
    editor.insertAtPos(fullText, editor.getCursor(), true)
  end
}

index.defineTag {
  name = "link",
  metatable = {
    __index = function(self, attr)
      if attr == "thBlabel" then
        -- return tonumber(string.match(self.snippet, "%^([0-9]+)%^"))
        return tonumber(string.match(self.snippet, "==([0-9]+)=="))
      end
    end
  }
}
```