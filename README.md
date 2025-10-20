--[[ واجهة عربية مطابقة لأسلوب Rayfield ومربوطة بسكربت أساسي (MainModule)

ملاحظات هامة:

هذا ملف واجهة يعتمد على مكتبة Rayfield الموجودة في: https://raw.githubusercontent.com/UI-Interface/CustomFIeld/main/RayField.lua

الواجهة مكتوبة بالعربية (عناوين، وصف الأزرار، إشعارات).

الواجهة تُخاطب "MainModule" كواجهة تنفيذية لوظائف اللعبة. السكربت يحتوي على نموذج (MainModule) مبسّط — استبدل/اربِطه بموديولك الحقيقي في حال كان في مكان آخر (مثلاً ReplicatedStorage أو ModuleScript داخل المشروع).

لا أغير منطق الهجوم الفعلي لأن أسماء RemoteEvents والمنطق تختلف بين الألعاب. لكن كل أزرار الواجهة تستدعي دوال MainModule (StartAutoFarm, StopAutoFarm, ToggleFly, SetSpeed, TeleportToSpawn, AttackNearest)


كيفية الاستخدام السريع:

1. ضع هذا الملف في مستغل يدعم HttpGet و writefile/readfile.


2. إذا كان لديك ModuleScript فعلي باسم "MainModule" في ReplicatedStorage أو مكان آخر، عدّل السطر الذي يقوم بتحميل MainModule ليرجع إليه.


3. شغّل الواجهة واضغط على الأزرار.



--]]

-- إعداد الخدمات local HttpService = game:GetService("HttpService") local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- تحميل مكتبة Rayfield local RAYFIELD_URL = "https://raw.githubusercontent.com/UI-Interface/CustomFIeld/main/RayField.lua" local function safeHttpGet(url) local ok, res = pcall(function() if syn and syn.request then local r = syn.request({Url = url, Method = "GET"}) return r.Body elseif http and http.get then return http.get(url) else return game:HttpGet(url) end end) if ok then return res end error("فشل تحميل Rayfield: " .. tostring(res)) end

local Rayfield = loadstring(safeHttpGet(RAYFIELD_URL))()


---

-- MainModule: نموذج مبدئي يربط الوظائف بالواجهة. استبدل بتنفيذك الخاص.


---

local MainModule = {} MainModule._running = {}

-- حالة المزرعة التلقائية MainModule.AutoFarmEnabled = false

function MainModule.StartAutoFarm() if MainModule.AutoFarmEnabled then return end MainModule.AutoFarmEnabled = true MainModule._running.autofarm = task.spawn(function() while MainModule.AutoFarmEnabled do -- ضع هنا منطق إيجاد الهدف والضرب (مثلاً FireServer لمواردك بشكل آمن) -- المثال التالي مجرد تحريك تجريبي نحو أقرب NPC pcall(function() local pl = game.Players.LocalPlayer if pl and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then local root = pl.Character.HumanoidRootPart -- محاولة ايجاد أقرب نموذج local nearest, nd = nil, math.huge for _,v in ipairs(workspace:GetDescendants()) do if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") then local hrp = v:FindFirstChild("HumanoidRootPart") local dist = (hrp.Position - root.Position).Magnitude if dist < nd then nd = dist nearest = v end end end if nearest and nearest:FindFirstChild("HumanoidRootPart") then root.CFrame = nearest.HumanoidRootPart.CFrame * CFrame.new(0,3,3) end end end) task.wait(0.8) end end) end

function MainModule.StopAutoFarm() MainModule.AutoFarmEnabled = false if MainModule._running and MainModule._running.autofarm then -- الخيط سيتوقف تلقائياً MainModule._running.autofarm = nil end end

-- تبديل المزرعة function MainModule.ToggleAutoFarm() if MainModule.AutoFarmEnabled then MainModule.StopAutoFarm() else MainModule.StartAutoFarm() end end

-- هجوم يدوي على أقرب هدف (يُستعمل من زر) function MainModule.AttackNearest() pcall(function() local pl = game.Players.LocalPlayer if pl and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then local root = pl.Character.HumanoidRootPart local nearest, nd = nil, math.huge for _,v in ipairs(workspace:GetDescendants()) do if v:IsA("Model") and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") then local hrp = v:FindFirstChild("HumanoidRootPart") local dist = (hrp.Position - root.Position).Magnitude if dist < nd then nd = dist nearest = v end end end if nearest and nearest:FindFirstChild("HumanoidRootPart") then root.CFrame = nearest.HumanoidRootPart.CFrame * CFrame.new(0,3,3) Rayfield:Notify({Title = "هجوم", Content = "اقتربت من الهدف لأداء الهجوم (ضع المنطق الخاص بك)", Duration = 2.5}) -- هنا يمكنك استدعاء RemoteEvent إن أردت: مثال -- local ev = ReplicatedStorage:FindFirstChild("AttackRemote") if ev then ev:FireServer(nearest) end else Rayfield:Notify({Title = "هجوم", Content = "لم أعثر على هدف قريب.", Duration = 2}) end end end) end

-- خصائص الطيران (نمطي بسيط) MainModule.FlyEnabled = false MainModule.FlyBody = nil MainModule.Speed = 16

function MainModule.EnableFly() if MainModule.FlyEnabled then return end MainModule.FlyEnabled = true local pl = game.Players.LocalPlayer if not pl or not pl.Character or not pl.Character:FindFirstChild("HumanoidRootPart") then return end local hrp = pl.Character.HumanoidRootPart local bv = Instance.new("BodyVelocity") bv.MaxForce = Vector3.new(1e5,1e5,1e5) bv.Parent = hrp MainModule.FlyBody = bv MainModule._running.fly = task.spawn(function() local UserInputService = game:GetService("UserInputService") local RunService = game:GetService("RunService") while MainModule.FlyEnabled and bv.Parent do local dir = Vector3.new(0,0,0) if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + hrp.CFrame.lookVector end if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - hrp.CFrame.lookVector end if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - hrp.CFrame.rightVector end if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + hrp.CFrame.rightVector end if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0,1,0) end if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir = dir - Vector3.new(0,1,0) end if dir.Magnitude > 0 then bv.Velocity = dir.Unit * MainModule.Speed else bv.Velocity = Vector3.new(0,0,0) end RunService.Heartbeat:Wait() end if bv and bv.Parent then bv:Destroy() end end) end

function MainModule.DisableFly() MainModule.FlyEnabled = false if MainModule._running.fly then MainModule._running.fly = nil end if MainModule.FlyBody and MainModule.FlyBody.Parent then MainModule.FlyBody:Destroy() end MainModule.FlyBody = nil end

function MainModule.ToggleFly() if MainModule.FlyEnabled then MainModule.DisableFly() else MainModule.EnableFly() end end

function MainModule.SetSpeed(val) MainModule.Speed = tonumber(val) or 16 -- أيضاً نحدد سرعة المشي الافتراضية local pl = game.Players.LocalPlayer if pl and pl.Character then local hum = pl.Character:FindFirstChildOfClass("Humanoid") if hum then pcall(function() hum.WalkSpeed = MainModule.Speed end) end end end

function MainModule.TeleportToSpawn() pcall(function() local pl = game.Players.LocalPlayer if pl and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then local spawn = workspace:FindFirstChildOfClass("SpawnLocation") or workspace:FindFirstChild("Spawn") if spawn then pl.Character.HumanoidRootPart.CFrame = CFrame.new(spawn.Position + Vector3.new(0,3,0)) Rayfield:Notify({Title = "Teleport", Content = "تم النقل إلى نقطة البداية.", Duration = 2}) else Rayfield:Notify({Title = "Teleport", Content = "لم أجد نقطة بداية في الخريطة.", Duration = 2.5}) end end end) end


---

-- بناء واجهة Rayfield بالعربية وربطها بدوال MainModule


---

local Window = Rayfield:CreateWindow({ Name = "لوحة التحكم - واجهتي العربية", LoadingTitle = "جارِ بناء الواجهة", LoadingSubtitle = "التحقق من الربط بالسكربت الأساسي", ConfigurationSaving = { Enabled = true, FolderName = "واجهتي_العربية", FileName = "الإعدادات" } })

-- تبويبات واقسام local Tab_Main = Window:CreateTab("الأساسيات", 4483362458) local Section_Farm = Tab_Main:CreateSection("المزرعة التلقائية") local Section_Player = Tab_Main:CreateSection("اللاعب") local Section_Utilities = Tab_Main:CreateSection("أدوات مساعدة")

-- زر تبديل المزرعة Section_Farm:CreateToggle({ Name = "تشغيل/إيقاف المزرعة", CurrentValue = false, Flag = "AutoFarmToggle", Callback = function(val) if val then MainModule.StartAutoFarm() Rayfield:Notify({Title = "المزرعة", Content = "المزرعة التلقائية مفعلة", Duration = 2}) else MainModule.StopAutoFarm() Rayfield:Notify({Title = "المزرعة", Content = "المزرعة التلقائية متوقفة", Duration = 2}) end end })

-- Slider سرعة الاقتراب / سرعة المشي Section_Farm:CreateSlider({ Name = "سرعة الحركة", Range = {8, 150}, Increment = 1, Suffix = "", CurrentValue = MainModule.Speed, Flag = "SpeedSlider", Callback = function(value) MainModule.SetSpeed(value) Rayfield:Notify({Title = "السرعة", Content = "تم ضبط السرعة إلى: "..tostring(value), Duration = 1.7}) end })

-- زر هجوم يدوي Section_Player:CreateButton({ Name = "هاجم أقرب هدف الآن", Callback = function() MainModule.AttackNearest() end })

-- تبديل الطيران Section_Player:CreateToggle({ Name = "تشغيل الطيران", CurrentValue = false, Flag = "FlyToggle", Callback = function(val) if val then MainModule.EnableFly() Rayfield:Notify({Title = "طيران", Content = "تم تفعيل الطيران. استخدم WASD و المسافة للتحكم.", Duration = 3}) else MainModule.DisableFly() Rayfield:Notify({Title = "طيران", Content = "تم إيقاف الطيران.", Duration = 2}) end end })

-- اختصارات مفيدة في قسم الأدوات Section_Utilities:CreateButton({ Name = "الذهاب لنقطة البداية", Callback = function() MainModule.TeleportToSpawn() end })

Section_Utilities:CreateKeybind({ Name = "مفتاح تبديل المزرعة (F)", CurrentKeybind = "F", Hold = false, Flag = "AutoFarmKeybind", Callback = function() MainModule.ToggleAutoFarm() Rayfield:Notify({Title = "Keybind", Content = "تم تبديل حالة المزرعة تلقائياً.", Duration = 1.5}) end })

-- شريط الحفظ والتحميل من Config Window:CreateLabel({ Name = "حالة", Text = "الربط بالسكربت الأساسي: مفعل (MainModule مدمج كمثال).", Tooltip = "إذا أردت ربطه بموديول خارجي، عدل الكود في الأعلى.", })

-- إشعار جاهزية الواجهة Rayfield:Notify({Title = "جاهز", Content = "الواجهة العربية جاهزة ومربوطة بالسكربت الأساسي (MainModule). عدّل منطق الهجوم أو الربط حسب حاجتك.", Duration = 4})

-- حفظ مرجع MainModule داخل Rayfield Flags لسهولة الوصول من سكربتات خارجية Rayfield.MainModule = MainModule

-- نهاية الملف

