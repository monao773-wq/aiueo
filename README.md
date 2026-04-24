print[[

██╗███╗░░██╗██╗████████╗
██║████╗░██║██║╚══██╔══╝
██║██╔██╗██║██║░░░██║░░░
██║██║╚████║██║░░░██║░░░
██║██║░╚███║██║░░░██║░░░
╚═╝╚═╝░░╚══╝╚═╝░░░╚═╝░░░

]]
task.spawn(pcall, function()
    if SPY_LOADED == true then return end
    pcall(function() getgenv().SPY_LOADED = true end)
    -- // Initialise
    --if (getgenv().ChatSpy) then return getgenv().ChatSpy; end;
    repeat wait() until game:GetService("ContentProvider").RequestQueueSize == 0
    repeat wait() until game:IsLoaded()

    -- // Vars
    local Players = game:GetService("Players")
    local StarterGui = game:GetService("StarterGui")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local LocalPlayer = Players.LocalPlayer
    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
    local TextChatService, General = game:GetService("TextChatService")
    for _,v in pairs(TextChatService:GetChildren()) do
        if v.Name == "TextChannels" and v:FindFirstChild("RBXGeneral") then
            General = v.RBXGeneral
        end
    end
    getgenv().ChatSpy = {
        Enabled = true,
        EnabledTF = "",
        SpyOnSelf = false,
        Public = false,
        Count = 0,
        OnMsg = false,
        Chat = {
            Color  = Color3.fromRGB(0, 255, 255),
            Font = Enum.Font.SourceSansBold,
            TextSize = 18,
            Text = "",
        },
        IgnoreList = {
            {Message = ":part/1/1/1", ExactMatch = true},
            {Message = ":part/10/10/10", ExactMatch = true},
            {Message = "A?????????", ExactMatch = false},
            {Message = ":colorshifttop 10000 0 0", ExactMatch = true},
            {Message = ":colorshiftbottom 10000 0 0", ExactMatch = true},
            {Message = ":colorshifttop 0 10000 0", ExactMatch = true},
            {Message = ":colorshiftbottom 0 10000 0", ExactMatch = true},
            {Message = ":colorshifttop 0 0 10000", ExactMatch = true},
            {Message = ":colorshiftbottom 0 0 10000", ExactMatch = true},
        },
    };

    -- // Function
    function ChatSpy.checkIgnored(message)
        for i = 1, #ChatSpy.IgnoreList do
            local v = ChatSpy.IgnoreList[i]
            if (v.ExactMatch and message == v.Message) or (not v.ExactMatch and string.match(v.Message, message)) then 
                return true
            end
        end
        return false
    end

    function ChatSpy.onChatted(targetPlayer, message)
        if (targetPlayer == LocalPlayer and string.lower(message) == "/spy") then
        ChatSpy.Count = ChatSpy.Count + 1
        if ChatSpy.Count < 2 then
            ChatSpy.Enabled = not(ChatSpy.Enabled)
            ChatSpy.EnabledTF = ChatSpy.Enabled and "Enabled." or "Disabled."
            ChatSpy.Chat.Text = `<font color='#10e3df'>{"{SPY} - "..ChatSpy.EnabledTF}</font>`
            task.wait(0.55)
            General:DisplaySystemMessage(ChatSpy.Chat.Text)
        else
            ChatSpy.Count = 0
        end
        elseif (ChatSpy.Enabled and (ChatSpy.SpyOnSelf or targetPlayer ~= LocalPlayer)) then
            local message = message:gsub("[\n\r]",''):gsub("\t",' '):gsub("[ ]+",' ')

            local Hidden = true
            local Connect = TextChatService.MessageReceived:Connect(function()
                Hidden = false
            end)

            task.wait(0.75)
            Connect:Disconnect()
            Connect = nil

            if (Hidden and ChatSpy.Enabled and not ChatSpy.checkIgnored(message)) then
                if (#message > 1200) then
                    message = message:sub(1200) .. "..."
                end
                if message:sub(1,2) == "/w" then
                    for _,plr in pairs(Players:GetPlayers()) do
                        local msg, count = message:gsub(plr.Name,plr.DisplayName)
                        if count ~= 0 then
                            ChatSpy.Chat.Text = `<font color='#10e3df'>{"{SPY} ["..targetPlayer.DisplayName.."]: "..msg}</font>`
                            break
                        end
                    end
                else
                    ChatSpy.Chat.Text = `<font color='#10e3df'>{"{SPY} ["..targetPlayer.DisplayName.."]: "..message}</font>`
                end
                General:DisplaySystemMessage(ChatSpy.Chat.Text)
            end
        end
    end

    -- // Handling Chats
    local AllPlayers = Players:GetPlayers()
    for i = 1, #AllPlayers do
        local player = AllPlayers[i]
        player.Chatted:Connect(function(message)
            ChatSpy.onChatted(player, message)
        end)
    end

    Players.PlayerAdded:Connect(function(player)
        player.Chatted:Connect(function(message)
            ChatSpy.onChatted(player, message)
        end)
    end)

    -- // Initialise Text
    ChatSpy.EnabledTF = ChatSpy.Enabled and "Enabled." or "Disabled."
    ChatSpy.Chat.Text = `<font color='#10e3df'>{"{SPY} - "..ChatSpy.EnabledTF}</font>`
    General:DisplaySystemMessage(ChatSpy.Chat.Text)
end)
local Rayfield = (function()
    if debugX then
        warn('Initialising Rayfield')
    end

    local function getService(name)
        local service = game:GetService(name)
        return if cloneref then cloneref(service) else service
    end

    local function loadWithTimeout(url: string, timeout: number?): ...any
        assert(type(url) == "string", "Expected string, got " .. type(url))
        timeout = timeout or 5
        local requestCompleted = false
        local success, result = false, nil

        local requestThread = task.spawn(function()
            local fetchSuccess, fetchResult = pcall(game.HttpGet, game, url)

            if not fetchSuccess or #fetchResult == 0 then
                if #fetchResult == 0 then
                    fetchResult = "Empty response" 
                end
                success, result = false, fetchResult
                requestCompleted = true
                return
            end
            local content = fetchResult -- Fetched content
            local execSuccess, execResult = pcall(function()
                return loadstring(content)()
            end)
            success, result = execSuccess, execResult
            requestCompleted = true
        end)

        local timeoutThread = task.delay(timeout, function()
            if not requestCompleted then
                warn(`Request for {url} timed out after {timeout} seconds`)
                task.cancel(requestThread)
                result = "Request timed out"
                requestCompleted = true
            end
        end)

        -- Wait for completion or timeout
        while not requestCompleted do
            task.wait()
        end
        -- Cancel timeout thread if still running when request completes
        if coroutine.status(timeoutThread) ~= "dead" then
            task.cancel(timeoutThread)
        end
        if not success then
            warn(`Failed to process {url}: {result}`)
        end
        return if success then result else nil
    end

    local requestsDisabled = true --getgenv and getgenv().DISABLE_RAYFIELD_REQUESTS
    local InterfaceBuild = '3K3W'
    local Release = "Build 1.68"
    local RayfieldFolder = "Rayfield"
    local ConfigurationFolder = RayfieldFolder.."/Configurations"
    local ConfigurationExtension = ".rfld"
    local settingsTable = {
        General = {
            rayfieldOpen = {Type = 'bind', Value = 'N', Name = 'Rayfield Keybind'},

        },
        System = {
            usageAnalytics = {Type = 'toggle', Value = true, Name = 'Anonymised Analytics'},
        }
    }

    local overriddenSettings: { [string]: any } = {} -- For example, overriddenSettings["System.rayfieldOpen"] = "J"
    local function overrideSetting(category: string, name: string, value: any)
        overriddenSettings[`{category}.{name}`] = value
    end

    local function getSetting(category: string, name: string): any
        if overriddenSettings[`{category}.{name}`] ~= nil then
            return overriddenSettings[`{category}.{name}`]
        elseif settingsTable[category][name] ~= nil then
            return settingsTable[category][name].Value
        end
    end

    if requestsDisabled then
        overrideSetting("System", "usageAnalytics", false)
    end

    local HttpService = getService('HttpService')
    local RunService = getService('RunService')

    -- Environment Check
    local useStudio = RunService:IsStudio() or false

    local settingsCreated = false
    local settingsInitialized = false -- Whether the UI elements in the settings page have been set to the proper values
    local cachedSettings
    local prompt = useStudio and require(script.Parent.prompt) or loadWithTimeout('https://raw.githubusercontent.com/SiriusSoftwareLtd/Sirius/refs/heads/request/prompt.lua')
    local requestFunc = (syn and syn.request) or (fluxus and fluxus.request) or (http and http.request) or http_request or request

    if not prompt and not useStudio then
        warn("Failed to load prompt library, using fallback")
        prompt = {
            create = function() end -- No-op fallback
        }
    end



    local function loadSettings()
        local file = nil

        local success, result =	pcall(function()
            task.spawn(function()
                if isfolder and isfolder(RayfieldFolder) then
                    if isfile and isfile(RayfieldFolder..'/settings'..ConfigurationExtension) then
                        file = readfile(RayfieldFolder..'/settings'..ConfigurationExtension)
                    end
                end

                if useStudio then
                    file = [[
            {"General":{"rayfieldOpen":{"Value":"K","Type":"bind","Name":"Rayfield Keybind","Element":{"HoldToInteract":false,"Ext":true,"Name":"Rayfield Keybind","Set":null,"CallOnChange":true,"Callback":null,"CurrentKeybind":"K"}}},"System":{"usageAnalytics":{"Value":false,"Type":"toggle","Name":"Anonymised Analytics","Element":{"Ext":true,"Name":"Anonymised Analytics","Set":nu
