This is where you configure SilverBullet to your liking. See [[^Library/Std/Config]] for a full list of configuration options.

```space-lua
config.set("plugs", {
  -- Add your plugs here (https://silverbullet.md/Plugs)
  -- Then run the `Plugs: Update` command to update them
})
```

# test1

${widgets.linkedMentions()}

# test2

```space-lua
local objects = {
    {tag = "mytask", ref="task1", content = "Buy groceries"},
    {tag = "mytask", ref="task2", content = "Write docs"}
}
index.indexObjects("my page", objects)
```

${index.queryLuaObjects("mytask", {})}

${query[[from index.tag "mytask"]]}

${index.getObjectByRef("my page", "mytask", "task1")}


# test3

```
-- priority: -1
myObjects = myObjects or {}

function addPosAnchor(anchorName)
  if not anchorName or anchorName == "" then
    editor.flashNotification("Missing anchor name", "warn")
    return
  end

  -- 检查是否已经存在该 anchorName
  local exists = false
  for _, obj in ipairs(myObjects) do
    if obj.name == anchorName then
      exists = true
      break
    end
  end

  -- 若不存在则插入新对象
  if not exists then
    local newObj = {
      tag = "posAnchor",
      name = anchorName,
      ref = editor.getCurrentPage() .. "@" .. editor.getCursor()
    }
    table.insert(myObjects, newObj)
    index.indexObjects("anchors", myObjects)
    editor.flashNotification("Added new posAnchor: " .. anchorName)
  else
    editor.flashNotification("Anchor already exists: " .. anchorName, "info")
  end

  return anchorName
end
```


```space-lua
function addPosAnchor(anchorName)
  local currentPage = editor.getCurrentPage()
  local pos = (widgetContext and widgetContext.pos)
          or (widgetContext and widgetContext.range and widgetContext.range.start)
          or editor.getCursor() -- fallback
  
  -- 构造符合官方规范的 Ref
  local ref = currentPage .. "@" .. tostring(pos)

  local exists = false
  for _, obj in ipairs(myObjects) do
    if obj.name == anchorName then
      exists = true
      break
    end
  end

  if not exists then
    local newObj = {
      tag = "posAnchor",
      name = anchorName,
      ref = ref,
    }
    table.insert(myObjects, newObj)
    index.indexObjects("anchors", myObjects)
    editor.flashNotification("Added posAnchor at " .. ref)
  else
    editor.flashNotification("Anchor already exists: " .. anchorName)
  end

  return ref
end

```


${addPosAnchor("asdf")}


${addPosAnchor("ffd")}

${query[[
    from index.tag "posAnchor"
  ]]}

${myObjects}

# test4

asdfsadfsadfsadfsadf #test

${query[[from index.tag "test"]]}

#test 的 parent 竟然是 page 而不是  paragraph:

${query[[from index.tag "tag" where page == _CTX.currentPage.name select {name=name, parent=parent}]]}

${addPosAnchor("#test")}


# test5

aasdfasdf [[CONFIG@123]] asdfasd


${query[[
    from index.tag "link"
  ]]}

[[label1|Cited by 2 times]]${template.each(query[[
    from index.tag "link"
    where string.startsWith(toPage, "bLabel")
    select {ref=_.ref}
  ]], template.new[==[-[[${_.ref}]]]==])}

dddddddddddddddddddddddddddddddddddddddddddddddddd

[[bLabel1|​]]${template.each(query[[
    from index.tag "link"
    where toPage == "label1"
    select {ref=_.ref}
  ]], template.new[==[[[${_.ref}|label1]]]==])}

[[bLabel2|​]]${template.each(query[[
    from index.tag "link"
    where toPage == "label1"
    select {ref=_.ref}
  ]], template.new[==[[[${_.ref}|custom ref name]]]==])}

${(query[[
  from index.tag "link"
  where string.startsWith(toPage, "bLabel")
]]).length}

# test6 

```
-- local suffixFlabel = "🗨" -- :bubble
-- local suffixBlabel = "🗯"
local prefixLabel = "📍"
local prefixBlabel = "📌"

local function tableBack(prefixBlabel)
  return query[[
    from index.tag "link"
    where string.startsWith(toPage, prefixBlabel)
    select {ref=_.ref}
  ]]
end

function setForwardanchorBackrefs(label)
  local numBlabel = (tableBack(prefixBlabel)).length
  local aspiringPageName = prefixLabel .. label
  local backrefStatistics = "|Cited " .. numBlabel .. " places"
  local forwardAnchor = "[[" .. aspiringPageName .. backrefStatistics .. "]]"
  local backRefs = template.each(tableBack(prefixBlabel), template.new[==[-[[${_.ref}]]​]==])
  return forwardAnchor .. backRefs
end

local function tableForward(label)
  local aspiringPageName = prefixLabel .. label
  return query[[
    from index.tag "link"
    where toPage == aspiringPageName
    select {ref=_.ref}
  ]]
end
local thBlabel = thBlabel or 0

function setBackanchorForwardref(label, alias)
  -- local thBlabel = (tableBack(prefixBlabel)).length + 1
  thBlabel = thBlabel + 1
  local aspiringPageName = prefixBlabel .. thBlabel
  local backAnchor = "[[" .. aspiringPageName .. "]]"
  local mytemplate
  if alias == nil or alias == "" then
    mytemplate = template.new[==[[[${_.ref}]]​]==]
  else
    mytemplate = template.new[==[[[${_.ref}|${alias}]]​]==]
  end
  forwardRef = template.each(tableForward(label), mytemplate)
  return backAnchor .. forwardRef
end
```


${query[[
    from index.tag "link"
    where string.startsWith(toPage, "📌")
    select {ref=_.ref}
  ]]}

${setForwardanchorBackrefs("asdf")}

${setBackanchorForwardref("asdf")}

${setBackanchorForwardref("asdf")}


1. https://chatgpt.com/share/6913891b-f880-8010-814d-697a48b6b203

```
function usrPrompt(hinText)
  local input = editor.prompt(hinText, "")
  if not input then
    editor.flashNotification("Cancelled", "warn")
  end
  return input
end

local suffixFlabel = "➖" -- "🗨"
local suffixBlabel = "➕" -- "🗯"
local F = "🔜" -- »
local B = "🔙" -- «

-- =========== Forth Anchor + Back Refs ==================

local function tableBack(Flabel)
  local aspiringPageBack = Flabel .. suffixBlabel
  return query[[
    from index.tag "link"
    where toPage:find(aspiringPageBack, 1, true) -- no Regex
    order by _.thBlabel
    select {ref=_.ref, thBlabel=_.thBlabel}
  ]]
end

function backrefStat(Flabel)
  return (tableBack(Flabel)).length
end

function backRefs(Flabel)
  return template.each(tableBack(Flabel), template.new[==[​*${_.thBlabel}*​[[${_.ref}]]​]==])
end

command.define {
  name = "insert: Forthanchor + Backrefs",
  key = "Alt-,",
  run = function()
    local Flabel = usrPrompt('Enter: label (to be Referred)')
    if not Flabel then return end
    local aspiringPageForth = Flabel .. suffixFlabel
    local forthAnchor = "[[" .. aspiringPageForth .. "||^|]]"
    local backrefStat = '${backrefStat("' .. Flabel .. '")}'
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
  return template.each(tableForth(Flabel), template.new[==[​[[${_.ref}]]​]==])
end

command.define {
  name = "insert: Backanchor + Forthref",
  key = "Alt-.",
  run = function()
    local Flabel = usrPrompt('Jump to: label')
    if not Flabel then return end
    local aspiringPageBack = Flabel .. suffixBlabel
    local backAnchor = "[[" .. aspiringPageBack .. "||^|]]"
    local thBlabel = "*" .. (tableBack(Flabel)).length + 1 .. "*"
    local backrefStat = '${backrefStat("' .. Flabel .. '")}'
    local forthRef = '${forthRef("' .. Flabel .. '")}'
    local fullText = backAnchor .. thBlabel .. F .. backrefStat .. forthRef
    editor.insertAtPos(fullText, editor.getCursor(), true)
  end
}

index.defineTag {
  name = "link",
  metatable = {
    __index = function(self, attr)
      if attr == "thBlabel" then
        return tonumber(string.match(self.snippet, "%*([^%*]+)%*"))
      end
    end
  }
}
```


[[asdf➖|fAbR]]${backrefStat("asdf")}🔙${backRefs("asdf")}

[[asdf➕|bAfR]]*1*🔜${backrefStat("asdf")}${forthRef("asdf")}

[[asdf➕|bAfR]]*2*🔜${backrefStat("asdf")}${forthRef("asdf")}

[[asdf➕|bAfR]]*3*🔜${backrefStat("asdf")}${forthRef("asdf")}

# test7

1. https://chatgpt.com/share/6913891b-f880-8010-814d-697a48b6b203

```
local prefixLabel = "📍"
local prefixBlabel = "📌"

-- 保存当前页面所有生成的 bLabel 链接
local generatedBLabels = {}

-- 返回当前已有 bLabel 列表
local function tableBack()
  return generatedBLabels
end

-- 根据已有 bLabel 数量动态生成下一个编号
local function nextBLabel()
  return #generatedBLabels + 1
end

-- 生成正向引用（📌）
function setForwardref(label, alias)
  -- 生成编号
  local thBlabel = nextBLabel()
  local aspiringPageName = prefixBlabel .. thBlabel
  local anchor = "[[" .. aspiringPageName .. "]]"

  -- 创建模板对象
  local forwardTpl
  if alias == nil or alias == "" then
    forwardTpl = template.new([==[[[${_.ref}]]​]==])
  else
    forwardTpl = template.new([==[[[${_.ref}|${alias}]]​]==])
  end

  -- 保存到 generatedBLabels 表，供反向引用使用
  table.insert(generatedBLabels, { ref = label, toPage = prefixLabel .. label })

  -- 渲染正向引用
  local forwardRef = template.each(tableForward(label), forwardTpl, { alias = alias })

  return anchor .. forwardRef
end

-- 返回指定 label 的正向引用列表
function tableForward(label)
  local aspiringPageName = prefixLabel .. label
  local forwards = {}
  for _, ref in ipairs(generatedBLabels) do
    if ref.toPage == aspiringPageName then
      table.insert(forwards, ref)
    end
  end
  return forwards
end

-- 生成反向引用锚点（📍）+统计
function setAnchorBackrefs(label)
  local aspiringPageName = prefixLabel .. label
  local backRefsList = tableBack()
  local numBlabel = #backRefsList

  local backrefStatistics = "|Cited " .. numBlabel .. " places"
  local anchor = "[[" .. aspiringPageName .. backrefStatistics .. "]]"

  local backRefs = ""
  if numBlabel > 0 then
    local tpl = template.new([==[-[[${_.ref}]]​]==])
    backRefs = template.each(backRefsList, tpl)
  end

  return anchor .. backRefs
end
```



