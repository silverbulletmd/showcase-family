The Bakers’ home page: who’s cooking, whose chores are left and what’s coming up. The shopping list is written on the home page itself.

```space-lua
family = {}

-- Widget links open pages through editor.navigate, so they work wherever the space is served
local function go(ref)
  return function()
    editor.navigate(ref)
  end
end

local function avatar(handle)
  return dom.span {
    class = "fam-avatar fam-" .. handle,
    handle:sub(1, 1):upper(),
  }
end

-- The fridge door: a note with everyone’s faces, and one saying what’s for dinner tonight
function family.header()
  local handles = query[[
    from p = index.contentPages "person"
    order by p.name
    select p.handle
  ]]
  local faces = {}
  for _, handle in ipairs(handles) do
    table.insert(faces, avatar(handle))
  end
  local today = os.date "%a"
  local meals = query[[
    from m = index.items "meal"
    where m.day == today
    select m
  ]]
  local tonight = meals[1]
  return widget.htmlBlock(dom.div {
    class = "fam fam-header",
    dom.div {
      class = "fam-note fam-title",
      dom.h1 { "The Bakers" },
      dom.p { "Meals, shopping, chores and plans." },
      dom.div {
        class = "fam-faces",
        table.unpack(faces),
      },
    },
    tonight and dom.div {
      class = "fam-note fam-tonight",
      dom.span { "Tonight" },
      dom.strong { tonight.name },
      dom.span { "cooking: ", tonight.cook },
    } or nil,
  })
end

local DAYS = {
  Mon = 1,
  Tue = 2,
  Wed = 3,
  Thu = 4,
  Fri = 5,
  Sat = 6,
  Sun = 7,
}

function family.meals()
  local meals = query[[
    from m = index.items "meal"
    select m
  ]]
  table.sort(meals, function(a, b)
    return DAYS[a.day] < DAYS[b.day]
  end)
  local today = os.date "%a"
  local cards = {}
  for _, m in ipairs(meals) do
    table.insert(cards, dom.div {
      class = m.day == today and "fam-meal fam-today" or "fam-meal",
      dom.div { class = "fam-day", m.day },
      dom.div { class = "fam-dish", m.name },
      dom.div { class = "fam-cook", m.cook },
    })
  end
  return widget.htmlBlock(dom.div {
    class = "fam fam-meals",
    table.unpack(cards),
  })
end

function family.chores()
  local people = query[[
    from p = index.contentPages "person"
    order by p.name
    select p
  ]]
  local chores = query[[
    from t = index.tasks()
    where t.page == "Chores" and not t.done
    select t
  ]]
  local rows = {}
  for _, p in ipairs(people) do
    local mine = {}
    for _, t in ipairs(chores) do
      if t.text:find("@" .. p.handle, 1, true) then
        if #mine > 0 then
          table.insert(mine, ", ")
        end
        table.insert(mine, dom.a {
          onclick = go(t.ref),
          (t.text:gsub("@%w+ ", "")),
        })
      end
    end
    if #mine == 0 then
      mine = { "all done ✓" }
    end
    table.insert(rows, dom.div {
      class = "fam-chore",
      avatar(p.handle),
      dom.span { table.unpack(mine) },
    })
  end
  return widget.htmlBlock(dom.div {
    class = "fam fam-chores",
    table.unpack(rows),
  })
end

function family.events()
  local events = query[[
    from e = index.items "event"
    order by e.date
    select e
  ]]
  local rows = {}
  for _, e in ipairs(events) do
    local y, m, d = e.date:match "(%d+)-(%d+)-(%d+)"
    local time = os.time {
      year = tonumber(y),
      month = tonumber(m),
      day = tonumber(d),
      hour = 12,
    }
    table.insert(rows, dom.div {
      class = "fam-event",
      dom.div {
        class = "fam-date",
        dom.span { os.date("%b", time) },
        dom.strong { tostring(tonumber(d)) },
      },
      dom.span { e.name },
    })
  end
  return widget.htmlBlock(dom.div {
    class = "fam fam-events",
    table.unpack(rows),
  })
end
```

```space-style
/* Let the home widgets sit on the page without the usual widget frame */
.sb-lua-directive-block:has(.fam) {
  border: none !important;
  background: none !important;
}

/* Notes on the fridge: cards on a faint wash of the accent */
.fam {
  --fam-card: var(--ui-accent-surface-color);
}
.fam-note,
.fam-meal,
.fam-chores,
.fam-events {
  border: 1px solid var(--ui-surface-border-color);
  border-radius: 12px;
  background: var(--fam-card);
}

/* The header: a title note and tonight’s dinner */
.fam-header {
  display: flex;
  align-items: stretch;
  gap: 10px;
}
.fam-title {
  flex: 1;
  padding: 16px 18px;
}
.fam-title h1 {
  margin: 0;
  font-size: 1.8em;
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--ui-accent-text-color);
}
.fam-title p {
  margin: 2px 0 12px;
  font-size: 0.9em;
  color: var(--subtle-color);
}
.fam-tonight {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 2px;
  min-width: 180px;
  padding: 14px 18px;
  border-color: transparent;
  background: var(--ui-accent-color);
  color: var(--ui-accent-contrast-color);
}
.fam-tonight > span {
  font-size: 13px;
  font-weight: 500;
  opacity: 0.85;
}
.fam-tonight > strong {
  font-size: 1.3em;
  font-weight: 700;
  letter-spacing: -0.01em;
}
.fam-tonight a {
  color: inherit !important;
}
.fam-faces {
  display: flex;
  padding-left: 6px;
}
.fam-faces .fam-avatar {
  width: 32px;
  height: 32px;
  margin-left: -6px;
  box-shadow: 0 0 0 2px var(--fam-card);
}

/* Avatars: one color per person */
.fam-avatar {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex: none;
  width: 26px;
  height: 26px;
  border-radius: 50%;
  color: #fff;
  font-size: 0.8em;
  font-weight: 700;
}
.fam-mia {
  background: #be123c;
}
.fam-leo {
  background: #0369a1;
}
.fam-noor {
  background: #7e22ce;
}
.fam-finn {
  background: #15803d;
}

/* One card per dinner; today’s is filled with the accent */
.fam-meals {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
}
.fam-meal {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 8px 10px 10px;
  border-radius: 10px;
  font-size: 0.8em;
  line-height: 1.3;
}
.fam-day {
  font-size: 12px;
  font-weight: 650;
  color: var(--ui-accent-text-color);
}
.fam-dish {
  font-weight: 600;
}
.fam-cook {
  color: var(--subtle-color);
}
.fam-meal.fam-today {
  border-color: transparent;
  background: var(--ui-accent-color);
}
.fam-today .fam-day,
.fam-today .fam-dish,
.fam-today .fam-cook,
.fam-today .fam-cook a {
  color: var(--ui-accent-contrast-color) !important;
  text-decoration-color: color-mix(in srgb, var(--ui-accent-contrast-color) 50%, transparent);
}

/* Chores and plans, each on one card */
.fam-chores,
.fam-events {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px 16px;
  padding: 14px 16px;
  font-size: 0.9em;
}
.fam-chore a {
  color: inherit !important;
  text-decoration: none !important;
  cursor: pointer;
}
.fam-chore a:hover {
  color: var(--ui-accent-text-color) !important;
  text-decoration: underline !important;
}
.fam-chore,
.fam-event {
  display: flex;
  align-items: center;
  gap: 10px;
}
.fam-date {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: none;
  width: 44px;
  padding: 4px 0;
  border-radius: 8px;
  background: var(--tone-accent-soft);
  line-height: 1.1;
}
.fam-date > span {
  font-size: 11px;
  font-weight: 600;
  color: var(--ui-accent-text-color);
}
.fam-date > strong {
  font-size: 1.15em;
  font-weight: 700;
}
```
