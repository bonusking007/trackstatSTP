local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local req = http_request or request or (syn and syn.request)
assert(req, "executor ไม่รองรับ http_request")

local API_URL = "https://stpwebdashboard-production.up.railway.app/api/update"
local API_KEY = "gemkey"
local player = Players.LocalPlayer

local function getGems()
    local ok, val = pcall(function()
        return player.PlayerGui.Main.GemDisplay.Count.Text
    end)
    if not ok then return 0 end
    -- ลบ comma และ space ออกก่อน tonumber
    local clean = tostring(val):gsub(",", ""):gsub("%s", "")
    return tonumber(clean) or 0
end

local function send()
    local gems = getGems()
    pcall(function()
        req({
            Url = API_URL,
            Method = "POST",
            Headers = {
                ["Content-Type"] = "application/json",
                ["x-api-key"] = API_KEY
            },
            Body = HttpService:JSONEncode({
                username = player.Name,
                gems = gems,
                ts = os.time()
            }),
            Timeout = 10
        })
    end)
    print("[Gem Tracker] Sent:", gems)
end

send()
task.spawn(function()
    while task.wait(23) do
        if not player.Parent then break end
        send()
    end
end)
