-- สร้างตัวแปร workspace และ RunService
local wsp = game:GetService("Workspace")
local RunService = game:GetService("RunService")

-- สร้างตัวแปร Players และ Local
local Players = game:GetService("Players")
local Local = Players.LocalPlayer

-- สร้างตัวแปร Camera และ Balls
local Camera = wsp.CurrentCamera
local Balls = wsp:WaitForChild("Balls")

-- สร้างตัวแปร Signal เพื่อใช้เก็บค่าตัวแปร Signal
getgenv().Signal = Signal or {}

-- ฟังก์ชัน PlayerPoints เพื่อหาตำแหน่งของผู้เล่นในจอ
function PlayerPoints()
    local tbl = {}
    for i, v in pairs(Players:GetPlayers()) do
        local UserId, HumanoidRootPart = tostring(v.UserId), v.Character and v.Character:FindFirstChild("HumanoidRootPart")
        if HumanoidRootPart and v == Local then
            warn(v)
            tbl[UserId] = Camera:WorldToScreenPoint(HumanoidRootPart.Position)
        end
    end
    
    print(unpack(tbl))
    table.foreach(tbl, print)
    return tbl
end

-- ฟังก์ชัน Parry เพื่อแยกยิงลูกบอล
function Parry()
    if Local.Character then
        local Remote = game:GetService("ReplicatedStorage"):WaitForChild("Remotes"):WaitForChild("ParryAttempt")
        local WorldToScreenPoint = Camera:WorldToScreenPoint(Local.Character.HumanoidRootPart.Position)
        local args = {
            [1] = 0.5,
            [2] = wsp.CurrentCamera.CFrame,
            [3] = PlayerPoints(),
            [4] = {
                [1] = WorldToScreenPoint.X,
                [2] = WorldToScreenPoint.Y
            }
        }
        
        warn("Players:", unpack(args[3]))
        Remote:FireServer(unpack(args))
    end
end

-- ฟังก์ชัน Anticipate เพื่อตรวจสอบการยิงลูกบอล
local Debounce, LastTime = false
function Anticipate(Time)
    if Debounce then return end
    
    if LastTime then
        local Sum = (Time - LastTime)
        if (Sum >= -25 and Sum <= 25) then
            print("Anticipated Time:", Sum, "Time:", Time, "LastTime:", LastTime)
            if Sum >= 25 or Sum <= -25 then
                return true
            end
        end
    end
    
    LastTime = Time
end

-- ฟังก์ชันคำนวณเวลาที่ลูกบอลใช้ในการเดินทางถึงเป้าหมาย
function calculateProjectileTime(initialPosition, targetPosition, initialVelocity)
    local distance = (targetPosition - initialPosition).Magnitude
    local time = distance / initialVelocity.Magnitude
    return time
end

-- ฟังก์ชันคำนวณระยะห่างระหว่างลูกบอลและวัตถุ
function calculateDistance(projectilePosition, objectPosition)
    return math.abs((projectilePosition - objectPosition).Magnitude)
end

-- ฟังก์ชันตรวจสอบว่าวัตถุสามารถป้องกัน (Parry) ลูกบอลได้หรือไม่
function canObjectParry(projectilePosition, objectPosition, projectileVelocity, objectVelocity)
    local timeToIntercept = calculateProjectileTime(projectilePosition, objectPosition, projectileVelocity)
    local distanceToIntercept = calculateDistance(projectilePosition + projectileVelocity * timeToIntercept, objectPosition + objectVelocity * timeToIntercept)
    local Anticipate = Anticipate(timeToIntercept)
    
    print("CanParry:", distanceToIntercept, timeToIntercept, Anticipate)
    
    local conditions = {
        (Anticipate and distanceToIntercept <= 75);
        (distanceToIntercept >= 35 and distanceToIntercept <= 50 and timeToIntercept <= 0.6);
        (distanceToIntercept >= 50 and distanceToIntercept <= 75 and timeToIntercept >= 0.6 and timeToIntercept <= 0.75);
        (distanceToIntercept <= 35 and timeToIntercept <= 0.5);
        (distanceToIntercept <= 12.5 and timeToIntercept >= 0.5 and timeToIntercept <= 0.75);
        (distanceToIntercept <= 0.025 and timeToIntercept <= 0.75);
        (distanceToIntercept >= 75 and distanceToIntercept <= 100 and timeToIntercept <= 0.5);
    }
    
    local r
    for i, v in pairs(conditions) do
        if v == true then
            warn(i,
            
