---
author: Chenzhu-Xie
name: Library/xczphysics/STYLE/Theme/LinkFloater
tags: meta/library
files:
- LinkFloater.js
pageDecoration.prefix: "🖇 "
share.uri: "https://github.com/ChenZhu-Xie/xczphysics_SilverBullet/blob/main/Library/xczphysics/STYLE/Theme/LinkFloater.md"
share.hash: a6a85c7c
share.mode: pull
---

# LinkFloater - Realtime Forward/Backward Link Navigator

这个插件，说实话有点像 [[PKM/Apps/Tana|]] 中的 Related Content
- 写了这么多 JS 代码，不如 Tana 的 Related Content 中一个 Query ?

## 1. JS Logic

本插件可能会用到：JS 与 Lua 通信
1. [task explorer](https://community.silverbullet.md/t/task-explorer/3747/2?u=chenzhu-xie) #community #silverbullet

## 2. Lua Logic (Bridge)
This part queries the index and pushes data to the JS view.i

```space-lua
-- 启用插件
js.import("/.fs/Library/xczphysics/STYLE/Theme/LinkFloater.js").enable()
js.import("/.fs/Library/xczphysics/STYLE/Theme/HHH.js").enableHighlight()

local function pushBacklinks()
    local currentPage = editor.getCurrentPage()
    
    local results = query[[
      from index.tag "link"
      where _.toPage == currentPage and _.page != currentPage
      select { ref=_.ref, page=_.page, pos=_.pos }
    ]]
    js.import("/.fs/Library/xczphysics/STYLE/Theme/LinkFloater.js").updateBacklinks(results)
end

local function pushForwardlinks()
    local currentPage = editor.getCurrentPage()
    
    local results = query[[
      from index.tag "link"
      where _.page == currentPage and not _.toFile
      select { ref=_.ref, toPage=_.toPage, pos=_.pos }
      order by _.pos
    ]]
    js.import("/.fs/Library/xczphysics/STYLE/Theme/LinkFloater.js").updateForwardlinks(results)
end

-- 页面加载时
event.listen {
  -- name = "hooks:renderTopWidgets", -- https://community.silverbullet.md/t/permanent-dark-mode/370/9?u=chenzhu-xie
  -- name = "system:ready", -- https://deepwiki.com/search/js-syscall-silverbullet-lua_ba58d949-b824-46ca-8063-6e7b42989389?mode=fast
  name = "editor:pageLoaded",
  run = function()
    js.import("/.fs/Library/xczphysics/STYLE/Theme/LinkFloater.js").refresh()
    pushBacklinks()
    pushForwardlinks()
  end
}

-- -- 页面修改时（更实时）
-- event.listen {
--   name = "editor:pageModified",
--   run = function()
--     pushForwardlinks()
--   end
-- }

-- 页面保存时
event.listen {
  name = "editor:pageSaving",
  run = function()
    pushForwardlinks()
    pushBacklinks()
  end
}
```

## 3. CSS Style

- need `CSS` from [[Library/xczphysics/STYLE/Theme/HHH|]]
