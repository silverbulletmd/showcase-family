#meta

The Bakers’ home page: who’s cooking, whose chores are left and what’s coming up. The shopping list is written on the home page itself.

```space-lua
family = {}

local function avatar(handle)
  return dom.span { class = "fam-avatar fam-" .. handle, handle:sub(1, 1):upper() }
end

function family.header()
  local people = query[[from p = index.contentPages "person" order by p.name select p.handle]]
  local faces = {}
  for _, h in ipairs(people) do table.insert(faces, avatar(h)) end
  return widget.htmlBlock(dom.div {
    class = "fam fam-header",
    dom.div { dom.h1 { "The Bakers" }, dom.p { "Our shared space: meals, shopping, chores and plans." } },
    dom.div { class = "fam-faces", table.unpack(faces) },
  })
end

function family.meals()
  local meals = query[[from m = index.items "meal" select m]]
  local order = { Mon = 1, Tue = 2, Wed = 3, Thu = 4, Fri = 5, Sat = 6, Sun = 7 }
  table.sort(meals, function(a, b) return order[a.day] < order[b.day] end)
  local today = os.date "%a"
  local cards = {}
  for _, m in ipairs(meals) do
    table.insert(cards, dom.div {
      class = m.day == today and "fam-meal fam-today" or "fam-meal",
      dom.div { class = "fam-day", m.day }, dom.div { class = "fam-dish", m.name }, dom.div { class = "fam-cook", m.cook },
    })
  end
  return widget.htmlBlock(dom.div { class = "fam fam-meals", table.unpack(cards) })
end

function family.chores()
  local people = query[[from p = index.contentPages "person" order by p.name select p]]
  local chores = query[[from t = index.tasks() where t.page == "Chores" and not t.done select t]]
  local rows = {}
  for _, p in ipairs(people) do
    local mine = {}
    for _, t in ipairs(chores) do
      if t.text:find("@" .. p.handle, 1, true) then
        if #mine > 0 then table.insert(mine, ", ") end
        table.insert(mine, dom.a { href = "/" .. t.ref, (t.text:gsub("@%w+ ", "")) })
      end
    end
    if #mine == 0 then mine = { "all done ✓" } end
    table.insert(rows, dom.div { class = "fam-chore", avatar(p.handle), dom.span { table.unpack(mine) } })
  end
  return widget.htmlBlock(dom.div { class = "fam fam-chores", table.unpack(rows) })
end

function family.events()
  local events = query[[from e = index.items "event" order by e.date select e]]
  local rows = {}
  for _, e in ipairs(events) do
    local y, m, d = e.date:match "(%d+)-(%d+)-(%d+)"
    local t = os.time { year = tonumber(y), month = tonumber(m), day = tonumber(d), hour = 12 }
    table.insert(rows, dom.div { class = "fam-event", dom.div { class = "fam-date", dom.span { os.date("%b", t) }, dom.strong { tostring(tonumber(d)) } }, dom.span { e.name } })
  end
  return widget.htmlBlock(dom.div { class = "fam fam-events", table.unpack(rows) })
end
```

```space-style
/* The family’s accent colour */
html:root {
  --ui-accent-color: #ea580c;
}

/* Let the home widgets sit on the page without the usual widget frame */
.sb-lua-directive-block:has(.fam) {
  border: none !important;
  background: none !important;
}

/* Banner with everyone’s faces */
.fam-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 14px 18px;
  border-radius: 14px;
  background: linear-gradient(135deg, #f97316, #fb923c 60%, #fdba74);
  color: #fff;
}
.fam-header h1 {
  margin: 0;
  font-size: 1.9em;
  color: #fff;
}
.fam-header p {
  margin: 4px 0 0;
  font-size: 0.85em;
  color: #ffedd5;
}
.fam-faces {
  display: flex;
}
.fam-faces .fam-avatar {
  width: 34px;
  height: 34px;
  margin-left: -8px;
  box-shadow: 0 0 0 3px #fb923c;
}

/* Avatars: one colour per person */
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

/* A card per dinner; today’s is filled in */
.fam-meals {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 6px;
}
.fam-meal {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 6px 7px;
  border-radius: 10px;
  background: #fff7ed;
  border: 1px solid #fed7aa;
  font-size: 0.75em;
  line-height: 1.25;
}
.fam-day {
  color: #c2410c;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.fam-dish {
  color: #431407;
  font-weight: 600;
}
.fam-cook,
.fam-cook a {
  color: #9a3412 !important;
  text-decoration: none !important;
}
.fam-today {
  background: #ea580c;
  border-color: #ea580c;
}
.fam-today .fam-day,
.fam-today .fam-dish,
.fam-today .fam-cook,
.fam-today .fam-cook a {
  color: #fff !important;
  opacity: 1;
}

/* Chores, two people per row */
.fam-chores {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 6px 16px;
  font-size: 0.85em;
}
.fam-chore a {
  color: inherit !important;
  text-decoration: none !important;
}
.fam-chore {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Upcoming events with a date badge */
.fam-events {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}
.fam-event {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.85em;
}
.fam-date {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 44px;
  padding: 4px 0;
  border-radius: 8px;
  background: #fff7ed;
  border: 1px solid #fed7aa;
  line-height: 1.1;
}
.fam-date span {
  font-size: 0.7em;
  text-transform: uppercase;
  color: #c2410c;
}
.fam-date strong {
  font-size: 1.2em;
  color: #431407;
}
```
