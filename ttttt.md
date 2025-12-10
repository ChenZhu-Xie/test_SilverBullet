

```space-lua
-- priority: 11
command.define {
  name = "Navigate: Tags Picker",
  key = "Ctrl-Alt-a",
  run = function()
    local selectedNames = {}

    local description = "Choose a search mode"
    local placeholder = "After each selection, which list to display?"
    local availableOptions = {}
    table.insert(availableOptions, {name="1- The complete list, always", value=1 })
    table.insert(availableOptions, {name="2- Only tags present in objects conforming to the criteria", value=2 })
    local mode = editor.filterBox("🔀Search mode", availableOptions, description, placeholder)
    
    if not mode then
      return
    end

    local allTags = query[[from index.tag "tag" select {name = _.name}]]
	
    while true do
	  if mode.value == 2 then
		  local potentialTags = {}
		  
		  if #selectedNames == 0 then
			potentialTags = allTags
		  else
			
			local q = query[[from index.tag(selectedNames[1])]]
			
			for i = 2, #selectedNames do
			  q = query[[
				from q where table.includes(_.tags, selectedNames[i])
			  ]]
			end
			
			local tagSet = {}
			for _, obj in ipairs(q) do
			  if obj.tags and type(obj.tags) == "table" then
				for _, t in ipairs(obj.tags) do
				  tagSet[t] = true
				end
			  end
			end
			
			-- convert Set to list for Picker
			for tagName, _ in pairs(tagSet) do
			  table.insert(potentialTags, {name = tagName})
			end
			
			table.sort(potentialTags, function(a, b) return a.name < b.name end)
		  end
		  allTags = potentialTags
	  end
  
      local availableOptions = {}
      for _, tagObj in ipairs(allTags) do
        if not table.includes(selectedNames, tagObj.name) then
          table.insert(availableOptions, tagObj)
        end
      end
      
      if #availableOptions == 0 then
        break
      end

      local description = ""
      if mode.value == 1 then
        description = "Complete list, always"
        else
        description = "Complete list then filtered list"
      end
      local placeholder = "🔖 a Tag"
      if #selectedNames > 0 then
        description = "Selected Tags ⏺️：" .. table.concat(selectedNames, ", ") .. " ➕ (ESC to Go)"
        placeholder = string.rep("🔖", #selectedNames) .. " a Tag"
      end
      
      local selection = editor.filterBox("🤏 Pick", availableOptions, description, placeholder)
      
      if selection then
        table.insert(selectedNames, selection.name)
      else
        if #selectedNames == 0 then
          return
        else
          break
        end
      end
    end
    
    local targetPage = "tags:" .. table.concat(selectedNames, ",")
    editor.navigate(targetPage)
  end
}
```

