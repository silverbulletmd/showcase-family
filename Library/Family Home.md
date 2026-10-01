#meta

The Bakers’ home page: who’s cooking, whose chores are left and what’s coming up. The shopping list is written on the home page itself.

```space-lua
family = {}

local function avatar(handle)
  return dom.span { class = "fam-avatar fam-" .. handle, handle:sub(1, 1):upper() }
end

-- The fridge door: a note with everyone’s faces, and one saying what’s for dinner tonight
function family.header()
  local people = query[[from p = index.contentPages "person" order by p.name select p.handle]]
  local faces = {}
  for _, h in ipairs(people) do table.insert(faces, avatar(h)) end
  local today = os.date "%a"
  local meals = query[[from m = index.items "meal" where m.day == today select m]]
  local tonight = meals[1]
  return widget.htmlBlock(dom.div {
    class = "fam fam-header",
    dom.div {
      class = "fam-note fam-title",
      dom.h1 { "The Bakers" }, dom.p { "Meals, shopping, chores and plans." }, dom.div { class = "fam-faces", table.unpack(faces) },
    },
    tonight and dom.div {
      class = "fam-note fam-tonight",
      dom.span { "Tonight" }, dom.strong { tonight.name }, dom.span { "cooking: ", tonight.cook },
    } or nil,
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
/* The family’s accent colour, on a warm paper background */
html[data-theme="light"]:root {
  --ui-accent-color: #ea580c;
  --root-background-color: #f4ead8;
  --top-background-color: #eadcc2;
}

/* Let the home widgets sit on the page without the usual widget frame */
.sb-lua-directive-block:has(.fam),
.sb-lua-directive-block:has(.fam) .content {
  border: none !important;
  background: none !important;
  overflow: visible !important;
}

/* The widget buttons float above the notes without a grey box */
.sb-lua-directive-block:has(.fam) .button-bar {
  z-index: 2;
  background: none !important;
}

/* Linked headings and cooks without the usual link chip */
#sb-main .cm-editor .sb-line-h1 .sb-wiki-link,
.fam-cook a {
  background: none !important;
  color: #c2410c !important;
}

/* Notes stuck on the fridge: slightly crooked, with a strip of tape */
.fam-note,
.fam-meal,
.fam-chores,
.fam-events {
  position: relative;
  border-radius: 2px;
  box-shadow: 0 4px 10px rgba(67, 20, 7, 0.18);
  color: #431407;
}
.fam-note::before,
.fam-chores::before,
.fam-events::before {
  content: "";
  position: absolute;
  top: -9px;
  left: calc(50% - 40px);
  width: 80px;
  height: 18px;
  background: rgba(255, 255, 255, 0.55);
  transform: rotate(-3deg);
}

/* The header: a title note and tonight’s dinner */
.fam-header {
  display: flex;
  align-items: flex-start;
  gap: 28px;
  padding: 30px 6px 6px;
}
.fam-title {
  flex: 1;
  padding: 16px 18px;
  background: #fef3c7;
  transform: rotate(-1.5deg);
}
.fam-title h1 {
  margin: 0;
  font-size: 1.9em;
  color: #c2410c;
}
.fam-title p {
  margin: 4px 0 10px;
  font-size: 0.85em;
}
.fam-tonight {
  display: flex;
  flex-direction: column;
  gap: 4px;
  margin-top: 10px;
  padding: 14px 16px;
  background: #fb923c;
  color: #fff;
  transform: rotate(3deg);
}
.fam-tonight span {
  font-size: 0.75em;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}
.fam-tonight strong {
  font-size: 1.3em;
}
.fam-tonight a {
  color: #fff !important;
}
.fam-faces {
  display: flex;
  padding-left: 8px;
}
.fam-faces .fam-avatar {
  width: 32px;
  height: 32px;
  margin-left: -8px;
  box-shadow: 0 0 0 3px #fef3c7;
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

/* A little note per dinner, each one a bit crooked; today’s is orange */
.fam-meals {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
  padding: 6px 2px;
}
.fam-meal {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 7px 7px 9px;
  background: #fffbeb;
  font-size: 0.75em;
  line-height: 1.25;
}
.fam-meal:nth-child(odd) {
  background: #ffedd5;
  transform: rotate(-2deg);
}
.fam-meal:nth-child(even) {
  transform: rotate(1.5deg);
}
.fam-day {
  color: #c2410c;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.fam-dish {
  font-weight: 600;
}
.fam-cook,
.fam-cook a {
  color: #9a3412 !important;
  text-decoration: none !important;
}
.fam-meal.fam-today {
  background: #fb923c;
}
.fam-today .fam-day,
.fam-today .fam-dish,
.fam-today .fam-cook,
.fam-today .fam-cook a {
  color: #fff !important;
}

/* Chores and plans, each on a sheet of paper */
.fam-chores,
.fam-events {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px 16px;
  margin: 4px 2px;
  padding: 16px 16px 12px;
  background: #fffdf8;
  font-size: 0.85em;
}
.fam-chores {
  transform: rotate(0.6deg);
}
.fam-events {
  transform: rotate(-0.6deg);
}
.fam-chore a {
  color: inherit !important;
  text-decoration: none !important;
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
  width: 44px;
  padding: 4px 0;
  border: 1px solid #fed7aa;
  background: #fff7ed;
  line-height: 1.1;
}
.fam-date span {
  font-size: 0.7em;
  text-transform: uppercase;
  color: #c2410c;
}
.fam-date strong {
  font-size: 1.2em;
}
```
