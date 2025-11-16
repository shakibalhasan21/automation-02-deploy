# Render.com এ Deploy করার ধাপ (বাংলা)

এই নির্দেশিকায় আমি ধাপে ধাপে দেখাবো কীভাবে আপনার `automation-02` প্রজেক্ট Render.com-এ ডিপ্লয় করবেন। আপনার প্রকল্পে দুটি অপশন আছে — সহজ Python runtime (gunicorn) অথবা Docker (Selenium/Chrome দরকার হলে)।

প্রারম্ভিক শর্ত
- আপনার কোড পরিবর্তন গিটে কমিট করা আছে এবং রিপো GitHub-এ আছে (উদাহরণ: `rajib3777/automation-02`).
- Render অ্যাকাউন্ট আছে এবং আপনি লগইন করতে পারবেন।

ফাইলগুলো আমি সেখানে যোগ করেছি:
- `render.yaml` — Python runtime উদাহরণ (gunicorn start command)।
- `render_docker.yaml` — Docker-based deployment (ব্যবহার করলে `Dockerfile_enhanced` ব্যবহার হবে)।

কীভাবে চালাবেন (ধাপে ধাপে)

1) পরিবর্তনগুলি রিপোতে পুশ করুন

```bash
git add render.yaml render_docker.yaml README_DEPLOY_RENDER.md
git commit -m "Add Render deployment configs and README"
git push origin main
```

2) Render-এ নতুন Web Service তৈরি
- Render ড্যাশবোর্ডে যান → `New` → `Web Service`।
- `Connect a repository` → আপনার GitHub অ্যাকাউন্ট কানেক্ট করে `rajib3777/automation-02` রিপো নির্বাচন করুন।

3) কোন কনফিগ ব্যবহার করবেন
- সহজ (Python runtime): যদি আপনার অ্যাপ শুধুমাত্র Flask ও Gunicorn চালায় এবং Selenium/Chrome সার্ভিস না চালাবে, `render.yaml` ব্যবহার করতে পারেন — Render সেটিংস স্বয়ংক্রিয়ভাবে পড়বে।
- Docker (সামর্থ্যশালী/রেকমেন্ডেড যদি Selenium দরকার): `render_docker.yaml` ব্যবহার করুন। এটি `Dockerfile_enhanced` ব্যবহার করে Chrome ও driver ইনস্টল করবে।

4) Manual setup (যদি আপনি render.yaml ব্যবহার করতে না চান)
- Build Command: `pip install --upgrade pip && pip install -r requirements.txt`
- Start Command: `gunicorn ultra_powerful_app_enhanced:app --bind 0.0.0.0:$PORT`

5) Environment Variables (অত্যন্ত জরুরি)
- Render Dashboard → আপনার সার্ভিস → `Environment` → `Environment Variables` এ নিম্নগুলি যোগ করুন:
  - `DATABASE_PATH` = `/data/multi_center_automation.db`
  - `SYSTEM_PASSWORD` = (আপনার গোপন পাসওয়ার্ড)
  - `SECRET_KEY` = (র্যান্ডম সিক্রেট)

নোট: কাউকে আপনার গোপনীর মান দেবেন না এবং এগুলো সোর্স কোডে রেকর্ড করবেন না।

6) Persistent Disk (খুব গুরুত্বপূর্ণ যদি আপনি SQLite ব্যবহার করেন)
- Docker পদ্ধতিতে `render_docker.yaml` ফাইলে `disk.sizeGB` যোগ করা আছে — Render UI-তে এই সার্ভিস তৈরির সময় persistent disk সক্রিয় করুন যাতে `/data` মাউন্ট পেতে পারেন।

7) Deploy এবং লগ মনিটর
- Deploy শুরু হলে `Deploys` → `Live` → `Logs` দেখুন। প্রথম বিল্ডে Docker image তৈরি হলে সময় লাগবে।
- যদি Build ত্রুটি হয়: Build log কপি করে আমাকে দিন, আমি সাহায্য করে দেব।

Local Docker টেস্ট (ঐচ্ছিক — আপনার মেশিনে Docker থাকলে)

```bash
# প্রজেক্ট রুটে:
docker build -t automation-02 -f Dockerfile_enhanced .
mkdir -p ./data
docker run --rm -p 5000:5000 -v $(pwd)/data:/data \ 
  -e DATABASE_PATH=/data/multi_center_automation.db \ 
  -e SYSTEM_PASSWORD=change_me \ 
  automation-02

# পরে ব্রাউজারে দেখুন: http://localhost:5000
```

সম্ভাব্য সমস্যা ও সমাধান
- Chrome/driver সম্পর্কিত ত্রুটি: Docker পদ্ধতি সবচেয়ে নির্ভরযোগ্য। Python runtime সেটআপে headless Chrome সাধারনত কাজ করবে না কিংবা সেটআপ জটিল।
- SQLite concurrency: একাধিক instance চালালে সমস্যা হতে পারে — প্রোডাকশনে Postgres ব্যবহার করার পরামর্শ দেয়া হয়।
- ডিপ্লয় আউটপুটে কোন প্যাকেজ ইনস্টল এরর হলে লোকালি `pip install -r requirements.txt` চালিয়ে দেখে দেখুন কোন প্যাকেজ ব্যর্থ হচ্ছে।

আপনি যদি চান আমি পুরো প্রক্রিয়াটি আপনার জন্য Render UI-তে করে দিই — নীচে কী ধরনের অনুমতি দরকার সেটা উল্লেখ করছি:
- (আমি সার্ভিস তৈরি করতে পারব না) কারণ এই টুলে আমি আপনার Render/GitHub অ্যাক্সেস টোকেন নেই। আপনাকে নিজে GitHub-এ পুশ করে Render UI-তে লগইন করে সার্ভিস তৈরি করতে হবে।

পরবর্তী কী করব
- আমি `render_docker.yaml` যোগ করেছি। আপনি আমাকে বলুন: আমি `render.yaml`-কে Docker-ভিত্তিক করে আপডেট করব, নাকি আপনি নিজে Render-এ Docker-ভিত্তিক সার্ভিস তৈরি করবেন? 

প্রশ্ন বা লোগস পাঠান — আমি ডিবাগ করে দেব।
