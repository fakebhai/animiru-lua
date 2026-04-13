## About
- If chapters are available then short tap makes it seek to next chapter and long seeks to previous chapter
- In case of no chapters being available it acts as your regular +85s button

## Code
Lua code
```local chapter_count = mp.get_property_number("chapters", 0)
if chapter_count > 0 then
    local chapter = mp.get_property_number("chapter")
    local title = mp.get_property("chapter-list/" .. chapter .. "/title")
    local chapter_title = "current chapter"
    if title ~= nil then
        chapter_title = title
    end

    mp.command("no-osd add chapter 1")
    aniyomi.show_text("Skipped " .. chapter_title)
else
    local intro_length = mp.get_property_number("user-data/current-anime/intro-length")
    aniyomi.right_seek_by(intro_length)
end
```

Lua code (on long press)
```local chapter_count = mp.get_property_number("chapters", 0)
if chapter_count > 0 then
    local chapter = mp.get_property_number("chapter")
    local title = mp.get_property("chapter-list/" .. chapter .. "/title")
    local chapter_title = "current chapter"
    if title ~= nil then
        chapter_title = title
    end

    mp.command("no-osd add chapter -2")
    aniyomi.show_text("Unskipped " .. chapter_title)
else
    aniyomi.int_picker("Change intro length", "%ds", 0, 255, 1, "user-data/current-anime/intro-length")
end
```

On startup
```local is_observing_intro = true

local function update_intro(_, length)
    if length == nil then return end

    if length == 0 then
        aniyomi.hide_button()
        return
    else
        aniyomi.show_button()
    end
    aniyomi.set_button_title("+" .. length .. " s")
end

local function update_chapter(_, value)
    if value == nil then return end
    local title = mp.get_property("chapter-list/" .. value .. "/title")
    local chapter_title = "N/A"
    if title ~= nil then
        chapter_title = title
    end

    aniyomi.show_button()
    aniyomi.set_button_title(chapter_title)
end

if $isPrimary then
    mp.observe_property("user-data/current-anime/intro-length", "number", update_intro)
end

mp.register_event("file-loaded", function()
    if not $isPrimary then return end
    local chapter_count = mp.get_property_number("chapters", 0)
    if chapter_count > 0 then
        mp.observe_property("chapter", "number", update_chapter)
        mp.unobserve_property("user-data/current-anime/intro-length", update_intro)
        is_observing_intro = false
    else
        mp.unobserve_property("chapter", update_chapter)
        if not is_observing_intro then
            mp.observe_property("user-data/current-anime/intro-length", "number", update_intro)
        end
        is_observing_intro = true
    end
end)
```

### To-Do: 
- Maybe use something better than -2 for seeking to previous chapter
- add current_chap/total_chap indicator on button