### WSL (Ubuntu) এর ভিতরে ঢোকার (open করার) কয়েকটা সহজ উপায় আছে 👍
```
✅ Method 1: Start Menu থেকে (সবচেয়ে সহজ)
Windows key চাপো
লিখো: Ubuntu
Open করো

👉 এটা খুললেই তুমি WSL (Ubuntu) terminal এর ভিতরে চলে যাবে

✅ Method 2: Windows Terminal ব্যবহার করে
Windows Terminal open করো
dropdown (▼) ক্লিক করো
Ubuntu select করো
✅ Method 3: Command ব্যবহার করে (PowerShell / CMD)

PowerShell বা CMD এ লিখো:

wsl

👉 এটা দিলে default Ubuntu WSL খুলে যাবে
```
### Option 1: Run Redis with WSL (Recommended)
```
Step 1: Redis restart (clean way)

WSL terminal এ চালাও:

sudo service redis-server stop

তারপর আবার start করো:

sudo service redis-server start
✅ Step 2: Check Redis running কিনা
redis-cli ping

Output আসবে:

PONG

👉 মানে Redis ready ✔️

🔥 Step 3: Fully fresh restart.

Step 3: Fully fresh restart (best practice)

যদি তুমি clean reset চাও:

sudo service redis-server restart
🧪 Step 4: Test new session
redis-cli

তারপর:

SET test "hello redis"
GET test
⚠️ যদি Redis start না হয়

Redis officially supports Linux, so using the Windows Subsystem for Linux (WSL) is the easiest and most reliable approach.

Open PowerShell as Administrator:

wsl --install
if install before then 
wsl

Restart your computer if prompted.
Open Ubuntu from the Start menu.

Install Redis:

sudo apt update
if not install before then 
sudo apt install redis-server -y

Start Redis:

sudo service redis-server start

Verify it's running:

redis-cli ping

Expected output:

PONG

You can learn more from the official Redis docs: Redis Installation Guide
```
