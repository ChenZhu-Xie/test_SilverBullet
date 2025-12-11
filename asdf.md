

asdf

asdfsadfsadfasdfssadfasdfadfsdaf [[la2🟣|]]==1== ➡️ ${forthRef("la2")}${backrefStat("la2")}*~Σ~*  asdfsadfsadf







```space-lua
-- priority: 10
yg = yg or {}

-- 模板改为使用 ${badge}，具体符号在数据阶段注入
local function bc_last()
  return template.new([==[${badge}[[${name}]]​]==])
end

-- 面包屑：根据是否有子页面，使用 🧑‍🤝‍🧑 或 👩🏼‍🤝‍👩🏻 拼接
function yg.bc(path)
  local mypage = path or editor.getCurrentPage()
  -- 仅决定视觉符号，不再直接拼接字符串
  local arrow_symbol_1 = has_children(mypage) and "⇩" or "⬇"
  local arrow_symbol_2 = has_children(mypage) and "🧑‍🤝‍🧑" or "👬🏼"
  
  local parts = string.split(mypage, "/")
  local current = ""
  local dom_list = {"[[.]]"}

  -- 抽出来一个辅助函数：给定 parent_path/current，算出可用的 sibling options
  local function collect_siblings(parent_path, current_page)
    -- 1. 确定查询前缀：如果是根目录则为空，否则加 /
    local prefix = parent_path == "" and "" or (parent_path .. "/")

    local siblings = query[[
      from index.tag 'page'
      where _.name:startsWith(prefix) and _.name != current_page
      select {name = _.name}
    ]]

    -- 3. 过滤：只保留直接子级（模拟文件系统的同级目录），排除孙级页面
    local options = {}
    for _, item in ipairs(siblings) do
      local p_name = item.name
      -- 获取相对路径
      local rel_path = p_name:sub(#prefix + 1)

      -- 如果相对路径中没有 "/"，说明是直接同级
      if not rel_path:find("/") then
        table.insert(options, { name = p_name })
      end
    end
    return options
  end

  for _, part in ipairs(parts) do
    -- 记录当前层级的父路径（用于查询同级页面）
    local parent_path = current

    if current ~= "" then current = current .. "/" end
    current = current .. part

    -- 先预查一次 siblings
    local options = collect_siblings(parent_path, current)

    if #options == 0 then
      -- 没有 siblings：只渲染一个箭头符号字符串，避免“点了也没用”的按钮
      table.insert(dom_list, arrow_symbol_1)
    else
      -- 有 siblings：生成按钮，点击时直接用预先算好的 options
      local function pick_sibling()
        local opt = editor.filterBox("🤏 Pick", options, "Select a Sibling", "🧑‍🤝‍🧑 a Sibling")
        if not opt then return end
        editor.navigate(opt.name)
      end

      local buto = widgets.button(arrow_symbol_2 .. #options, pick_sibling)
      table.insert(dom_list, buto)
    end
    table.insert(dom_list, "[[" .. current .. "]]")
  end

  -- 最近修改/访问徽章
  local lastMs = template.each(yg.lastM(mypage), bc_last()) or ""
  local lastVs = template.each(yg.lastV(mypage), bc_last()) or ""

  -- 访问次数
  local data = datastore.get({"Visitimes", mypage}) or {}
  local visits = data.value or 0
  -- local visitsSuffix = "[[CONFIG/Add_Fields_for_Obj/Last_Opened-Page/Visit_Times|" .. "👀" .. tostring(visits) .. "]]"
  local visiTimes = "[[CONFIG/Add_Fields_for_Obj/Last_Opened-Page/Visit_Times|" .. tostring(visits) .. "]]"

  -- pick children
  local options = query[[from index.tag "page"
         -- where _.name:startsWith(mypage .. "/")
         where _.name:find("^" .. mypage .. "/")
         select {name = _.name}]]
  -- table.insert(dom_list, " " .. visitsSuffix)
  if #options == 0 then
    table.insert(dom_list, "👀")
  else
    local function pick_child()
      local opt = editor.filterBox("🤏 Pick", options, "Select a Child", "👶🏻 a Child")
      if not opt then return end
      editor.navigate(opt.name)
    end
    local buto = widgets.button("👶🏻" .. #options, pick_child)
    table.insert(dom_list, buto)
  end
  table.insert(dom_list, visiTimes)
  table.insert(dom_list, "\n" .. lastMs)
  table.insert(dom_list, "\n" .. lastVs)

  return dom_list
end

-- 支持最多 9 个（对应 1~9）
local max_num = 5

-- 辅助：判断是否有子页面
local function has_children(mypage)
  local children = query[[from index.tag "page"
         where _.name:find("^" .. mypage .. "/")
         limit 1]]
  return #children > 0
end

function yg.lastM(mypage)
  local hasChild = has_children(mypage)

  -- 选择数据源：有子页面时选子页面最近修改，否则全局最近修改（排除当前页）
  local list = hasChild and query[[from index.tag "page" 
         where _.name:find("^" .. mypage .. "/")
         order by _.lastModified desc
         limit max_num]]
       or query[[from index.tag "page"
         where _.name != mypage
         order by _.lastModified desc
         limit max_num]]

  -- 序号徽章（bc_lastM）
  local M_hasCHILD  = {"1⃣","2⃣","3⃣","4⃣","5⃣","6⃣","7⃣","8⃣","9⃣"}
  local M_noCHILD   = {"1️⃣","2️⃣","3️⃣","4️⃣","5️⃣","6️⃣","7️⃣","8️⃣","9️⃣"}
  local badges = hasChild and M_hasCHILD or M_noCHILD

  for i, item in ipairs(list) do
    item.badge = badges[i] or ""
  end
  return list
end

function yg.lastV(mypage)
  local hasChild = has_children(mypage)

  -- 选择数据源：有子页面时选子页面最近访问，否则全局最近访问（排除当前页）
  local list = hasChild and 
  query[[from editor.getRecentlyOpenedPages "page" 
         where _.lastOpened and _.name:find("^" .. mypage .. "/")
         order by _.lastOpened desc
         limit max_num]]
       or query[[from editor.getRecentlyOpenedPages "page" 
         where _.lastOpened and _.name != mypage
         order by _.lastOpened desc
         limit max_num]]
  
  -- 序号徽章（bc_lastV）
  local V_hasCHILD  = {"①","②","③","④","⑤","⑥","⑦","⑧","⑨"}
  local V_noCHILD   = {"➊","➋","➌","➍","➎","➏","➐","➑","➒"}
  local badges = hasChild and V_hasCHILD or V_noCHILD

  for i, item in ipairs(list) do
    item.badge = badges[i] or ""
  end
  return list
end

function widgets.breadcrumbs()
  return widget.new {
    -- markdown = yg.bc()
    html = dom.div(yg.bc()),
    display = "block",
  }
end
```



```space-lua
-- priority: 10
Yg = Yg or {}

-- 仅用于 pattern() 的场景选择（保留原逻辑）
local function choose(a, b, path)
  if path and #path > 0 then
    return a
  else
    return b
  end
end

-- 模板使用 ${badge}，序号徽章在数据阶段注入
local function Bc_last()
  return template.new([==[${badge}[[${name}]]​]==])
end

-- 与原逻辑一致：决定“同父级子页”或“顶层单段”的匹配
local function pattern(path)
  -- return choose("^" .. path .. "/[^/]+$", "^[^/]+$", path)
  local a = path and ("^" .. path .. "/[^/]+$") or nil
  return choose(a, "^[^/]+$", path)
end

local max_num = 5  -- 如需覆盖 1~9，可改为 9

function Yg.lastM(thisPage, mypath)
  local list = query[[from index.tag "page" 
         where _.name ~= thisPage and _.name:find(pattern(mypath))
         order by _.lastModified desc
         limit max_num]]

  -- 方块风格（沿用 Top 的约定）
  local M_hasFATHER = {"1⃣","2⃣","3⃣","4⃣","5⃣","6⃣","7⃣","8⃣","9⃣"}
  local M_noFATHER  = {"1️⃣","2️⃣","3️⃣","4️⃣","5️⃣","6️⃣","7️⃣","8️⃣","9️⃣"}
  local badges = choose(M_hasFATHER, M_noFATHER, mypath)

  for i, item in ipairs(list) do
    item.badge = badges[i] or ""
  end
  return list
end

function Yg.lastV(thisPage, mypath)
  local list = query[[from editor.getRecentlyOpenedPages "page"
         where _.lastOpened and _.name ~= thisPage and _.name:find(pattern(mypath))
         order by _.lastOpened desc
         limit max_num]]
  
  -- 圆形风格（沿用 Top 的约定）
  local V_hasFATHER = {"①","②","③","④","⑤","⑥","⑦","⑧","⑨"}
  local V_noFATHER  = {"➊","➋","➌","➍","➎","➏","➐","➑","➒"}
  local badges = choose(V_hasFATHER, V_noFATHER, mypath)

  for i, item in ipairs(list) do
    item.badge = badges[i] or ""
  end
  return list
end

-- 主面包屑：按是否有子页面切换 ⇦⇨ / ⬅⮕ 分隔符，并追加 👀访问次数
function Yg.bc(path)
  local thisPage = path or editor.getCurrentPage()
  local mypath = thisPage:match("^(.*)/[^/]*$")
  local arrow_symbol_1 = choose("⇦⇨", "⬅⮕", mypath)
  local arrow_symbol_2 = choose("👶🏻", "👼🏻", mypath)

  -- 构建 .⇦⇨CONFIG⇦⇨Widget... 或 .⬅⮕CONFIG⬅⮕Widget...
  local dom_list = {"[[.]]"}
  local parts = string.split(thisPage, "/")
  local current = ""
  
  -- 抽出来一个辅助函数：给定 current，算出可用的 Children options
  local function collect_children(current_page)
    return query[[
      from index.tag 'page'
      -- where _.name:find("^" .. current_page .. "/")
      where _.name:startsWith(current_page .. "/")
      select {name = _.name}
    ]]
  end

  for _, part in ipairs(parts) do
    -- 记录当前层级的父路径（用于查询同级页面）
    local parent_path = current

    if current ~= "" then current = current .. "/" end
    current = current .. part

    -- 先预查一次 children
    local options = collect_children(current)

    if #options == 0 then
      -- 没有 children：只渲染一个箭头符号字符串，避免“点了也没用”的按钮
      table.insert(dom_list, arrow_symbol_1)
    else
      -- 有 children：生成按钮，点击时直接用预先算好的 options
      local function pick_child()
        local opt = editor.filterBox("🤏 Pick", options, "Select a Child", "👶🏻 a Child")
        if not opt then return end
        editor.navigate(opt.name)
      end

      local buto = widgets.button(arrow_symbol_2 .. #options, pick_child)
      table.insert(dom_list, buto)
    end
    table.insert(dom_list, "[[" .. current .. "]]")
  end

  -- 最近修改 / 最近访问（带序号徽章）
  local lastMs = template.each(Yg.lastM(thisPage, mypath), Bc_last()) or ""
  local lastVs = template.each(Yg.lastV(thisPage, mypath), Bc_last()) or ""

  -- 访问次数
  local data = datastore.get({"Visitimes", thisPage}) or {}
  local visits = data.value or 0
  -- local visitsSuffix = "[[CONFIG/Add Fields for Obj/Last Opened/Visit Times|" .. "👀" .. tostring(visits) .. "]]"
  local visiTimes = "[[CONFIG/Add_Fields_for_Obj/Last_Opened-Page/Visit_Times|" .. tostring(visits) .. "]]"

  -- pick siblings
  local options = query[[from index.tag "page" 
         where _.name ~= thisPage and _.name:find(pattern(mypath))
         select {name = _.name}]]
  -- table.insert(dom_list, " " .. visitsSuffix)
  if #options == 0 then
    table.insert(dom_list, "👀")
  else
    local function pick_sibling()
      local opt = editor.filterBox("🤏 Pick", options, "Select a Sibling", "🧑‍🤝‍🧑 a Sibling")
      if not opt then return end
      editor.navigate(opt.name)
    end
    local buto = widgets.button("🧑‍🤝‍🧑" .. #options, pick_sibling)
    table.insert(dom_list, buto)
  end
  table.insert(dom_list, visiTimes)
  table.insert(dom_list, "\n" .. lastMs)
  table.insert(dom_list, "\n" .. lastVs)
  
  return dom_list
end

function widgets.breadcrumbs_B()
  return widget.new {
    -- markdown = Yg.bc()
    html = dom.div(Yg.bc()),
    display = "block",
  }
end
```