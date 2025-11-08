cd /path/to/harsh-designer-studio
npm install
npm run build
# Updating the existing project to add a simple client-side password gate (private preview)
import os, json, shutil
from pathlib import Path

proj = Path("/mnt/data/harsh-designer-studio")
app_path = proj / "src" / "App.jsx"

if not proj.exists() or not app_path.exists():
    raise FileNotFoundError("Project not found at expected path: " + str(proj))

text = app_path.read_text()

# Insert a PasswordGate component near the top and wrap the exported App default to require unlocking.
# We'll add the component just after the import line.
insertion = '''
function PasswordGate({onUnlock}){
  const [pw, setPw] = React.useState('')
  const [err, setErr] = React.useState('')
  const PASSWORD = 'harsh123' // preview password
  function submit(e){ e.preventDefault(); if(pw===PASSWORD){ localStorage.setItem('hds_unlocked','1'); onUnlock(true) } else { setErr('Wrong password / गलत पासवर्ड') } }
  return (
    <div style={{minHeight:'100vh',display:'flex',alignItems:'center',justifyContent:'center',background:'var(--accent)'}}>
      <div className="container" style={{maxWidth:420,textAlign:'center'}}>
        <h2>Private Preview — HARSH DESIGNER STUDIO</h2>
        <p className="small">Enter the preview password to view the site / साइट देखने के लिए पासवर्ड दर्ज करें</p>
        <form onSubmit={submit} style={{display:'grid',gap:8,marginTop:12}}>
          <input className="input" type="password" value={pw} onChange={e=> setPw(e.target.value)} placeholder="Enter preview password / पासवर्ड" />
          <button className="button" type="submit">Unlock / अनलॉक करें</button>
        </form>
        {err && <div style={{color:'red',marginTop:8}}>{err}</div>}
        <div style={{marginTop:12}} className="small">Use the preview password given by Harsh.</div>
      </div>
    </div>
  )
}
'''

# Find the location after the first line "export default function App()"
if "export default function App()" in text:
    new_text = text.replace("export default function App(){", "export default function App(){\n  const [siteUnlocked, setSiteUnlocked] = React.useState(localStorage.getItem('hds_unlocked')==='1')\n  if(!siteUnlocked){ return <PasswordGate onUnlock={setSiteUnlocked} /> }\n")
    # Insert PasswordGate component after imports (after the first occurrence of "import React")
    if "import React" in new_text:
        new_text = new_text.replace("import React, {useState, useEffect, useRef} from \"react\";", "import React, {useState, useEffect, useRef} from \"react\";\n" + insertion)
    app_path.write_text(new_text)
    print("App.jsx updated with PasswordGate")
else:
    raise RuntimeError("Could not find App export to modify")

# Recreate zip
zip_path = "/mnt/data/harsh-designer-studio-private.zip"
if os.path.exists(zip_path):
    os.remove(zip_path)
shutil.make_archive("/mnt/data/harsh-designer-studio-private", 'zip', root_dir=str(proj))
print("Created private zip at:", zip_path)
zip_path

# Retry creating the project zip with correct variables
import os, shutil, json
from pathlib import Path

project_dir = Path("/mnt/data/harsh-designer-studio")
if project_dir.exists():
    shutil.rmtree(project_dir)
project_dir.mkdir(parents=True)

# Copy provided images into project assets
shutil.copy("/mnt/data/1000039370.jpg", project_dir / "logo.png")
shutil.copy("/mnt/data/1000039358.jpg", project_dir / "qr.png")

OWNER_UPI = "6355092424@fam"

# package.json (Vite + React)
package = {
  "name": "harsh-designer-studio",
  "version": "1.0.0",
  "private": True,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0"
  }
}
(project_dir / "package.json").write_text(json.dumps(package, indent=2))

# index.html
index_html = """<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>HARSH DESIGNER STUDIO</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
"""
(project_dir / "index.html").write_text(index_html)

# src directory
src = project_dir / "src"
src.mkdir()

# main.jsx
main_jsx = """import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'
import './styles.css'

createRoot(document.getElementById('root')).render(<App />)
"""
(src / "main.jsx").write_text(main_jsx)

# styles.css with color palette
styles_css = """:root{
  --primary:#3F422E;
  --accent:#F5E6C4;
  --text:#2b2b2b;
}
*{box-sizing:border-box}
body{font-family:Inter, system-ui, Arial, sans-serif;margin:0;background:var(--accent);color:var(--text)}
.header{background:linear-gradient(90deg,var(--primary),#5b5b4a);padding:18px 12px;color:white;display:flex;align-items:center;gap:12px}
.header img{height:64px}
.container{max-width:980px;margin:28px auto;padding:20px;background:white;border-radius:12px;box-shadow:0 6px 22px rgba(0,0,0,0.06)}
.tagline{color:var(--primary);font-weight:600;margin-top:6px}
.grid{display:grid;gap:14px}
.controls{display:flex;gap:10px;flex-wrap:wrap}
.card{background:linear-gradient(180deg,white, #fffaf0);padding:14px;border-radius:10px;border:1px solid #eee}
.input, textarea, select{width:100%;padding:10px;border-radius:8px;border:1px solid #ddd;font-size:14px}
.button{background:var(--primary);color:white;padding:10px 12px;border-radius:8px;border:none;cursor:pointer}
.small{font-size:13px;color:#555}
.order-row{display:flex;justify-content:space-between;gap:10px;align-items:center;padding:12px;border-radius:8px;border:1px solid #eee}
.status-pending{color:#b58d00}
.status-done{color:green;font-weight:700}
.fab-wa{position:fixed;right:18px;bottom:18px;background:#25D366;color:white;padding:12px;border-radius:50%;box-shadow:0 6px 20px rgba(0,0,0,0.2);cursor:pointer}
.timer{font-weight:700;color:var(--primary)}
.portfolio-placeholder{text-align:center;padding:30px;color:#666}
.footer{text-align:center;font-size:13px;color:#666;padding:18px}
.admin-login{max-width:420px;margin:0 auto;padding:20px}
img.qr{max-width:220px;border-radius:8px;border:1px solid #ddd}
.logo-small{height:40px}
"""
(src / "styles.css").write_text(styles_css)

# App.jsx - main app implementing features
app_jsx = r'''
import React, {useState, useEffect, useRef} from "react";

const OWNER_NAME = "Harsh";
const OWNER_UPI = "6355092424@fam";
const OWNER_PHONE = "916355092424"; // for wa.me (country code +91)
const ADMIN_USER = "Harsh";
const ADMIN_PASS = "harsh123";

function uid(){ return 'ORD-'+Date.now() }

function nowISO(){ return new Date().toISOString() }

function saveOrders(data){
  localStorage.setItem("hds_orders", JSON.stringify(data))
}
function loadOrders(){
  try {
    return JSON.parse(localStorage.getItem("hds_orders")||"[]")
  } catch(e){ return [] }
}

export default function App(){
  const [view, setView] = useState('home') // home, order, orders, admin, track, portfolio
  const [orders, setOrders] = useState(loadOrders())
  const [form, setForm] = useState({
    customerName:'', email:'', phone:'', company:'', designType:'logo', quantity:1, details:'', txnId:''
  })
  const [timer, setTimer] = useState(0)
  const proofRef = useRef(null)
  const fileRef = useRef(null)
  const [adminAuth, setAdminAuth] = useState(localStorage.getItem("hds_admin") === "1")
  const [adminCred, setAdminCred] = useState({user:'', pass:''})
  const [trackId, setTrackId] = useState('')

  useEffect(()=> saveOrders(orders), [orders])

  // handle payment timer (2 minutes)
  useEffect(()=>{
    let t
    if(timer>0){
      t = setInterval(()=> setTimer(s=> s-1), 1000)
    }
    return ()=> clearInterval(t)
  }, [timer])

  function updateField(k,v){ setForm(f=>({...f,[k]:v})) }

  function startPaymentTimer(){
    setTimer(120)
  }

  function formatTime(s){
    const mm = Math.floor(s/60).toString().padStart(2,'0')
    const ss = (s%60).toString().padStart(2,'0')
    return `${mm}:${ss}`
  }

  async function handlePlaceOrder(e){
    e && e.preventDefault()
    // validation: mandatory fields and proof image must be present
    if(!form.customerName.trim()) return alert("Please enter name / कृपया नाम दर्ज करें")
    if(!form.email.trim()) return alert("Please enter email / कृपया ईमेल डालें")
    if(!form.phone.trim()) return alert("Please enter WhatsApp number / कृपया व्हाट्सऐप नंबर डालें")
    if(!form.details.trim()) return alert("Please describe requirements / कृपया विवरण डालें")
    const proofFile = proofRef.current && proofRef.current.files && proofRef.current.files[0]
    if(!proofFile) return alert("Payment screenshot is required / भुगतान स्क्रीनशॉट अनिवार्य है")
    // read proof as base64 (for demo storage only)
    const reader = new FileReader()
    reader.onload = function(ev){
      const proofData = ev.target.result
      const order = {
        id: uid(),
        ...form,
        fileName: fileRef.current?.files?.[0]?.name || null,
        proofName: proofFile.name,
        proofData,
        createdAt: nowISO(),
        status: "pending-payment", // awaiting admin verification
        paidUpfront: true
      }
      setOrders(o=> [order, ...o])
      setForm({customerName:'', email:'', phone:'', company:'', designType:'logo', quantity:1, details:'', txnId:''})
      if(proofRef.current) proofRef.current.value = null
      if(fileRef.current) fileRef.current.value = null
      setView('orders')
      alert("Order placed. Please click 'Message on WhatsApp' to notify Harsh and attach the screenshot in WhatsApp too. / ऑर्डर दर्ज हो गया। कृपया Harsh को सूचित करने के लिए WhatsApp संदेश भेजें और स्क्रीनशॉट अटैच करें।")
    }
    reader.readAsDataURL(proofFile)
  }

  function openWhatsAppPrefilled(order){
    const name = encodeURIComponent(order.customerName)
    const type = encodeURIComponent(order.designType)
    const msg = encodeURIComponent(`Hello Harsh, maine 50% advance payment kar diya hai. Payment ka screenshot bhej raha hoon.%0AName: ${order.customerName}%0AOrder ID: ${order.id}%0AOrder Type: ${order.designType}`)
    window.open(`https://wa.me/${OWNER_PHONE}?text=${msg}`, '_blank')
  }

  function adminLogin(e){
    e && e.preventDefault()
    if(adminCred.user === ADMIN_USER && adminCred.pass === ADMIN_PASS){
      setAdminAuth(true)
      localStorage.setItem("hds_admin","1")
      alert("Admin logged in")
    } else {
      alert("Invalid credentials")
    }
  }

  function adminLogout(){
    setAdminAuth(false)
    localStorage.removeItem("hds_admin")
  }

  function adminConfirmOrder(id){
    setOrders(o=> o.map(x=> x.id===id? {...x, status:'confirmed', confirmedAt:nowISO()} : x))
    alert("Order confirmed. You can send WhatsApp confirmation if you want.")
  }

  function adminVerifyPayment(id){
    setOrders(o=> o.map(x=> x.id===id? {...x, status:'in-progress'} : x))
    alert("Payment verified & marked in-progress.")
  }

  function customerTrack(){
    const ord = orders.find(o=> o.id === trackId.trim())
    if(!ord) return alert("Order not found / ऑर्डर नहीं मिला")
    setView('track')
  }

  function sendWhatsAppFromAdmin(order){
    const msg = encodeURIComponent(`Hello ${order.customerName}, Your order ${order.id} has been confirmed by Harsh Designer Studio. We will start work and update you soon.`)
    window.open(`https://wa.me/${order.phone.startsWith('91')? order.phone : '91'+order.phone}?text=${msg}`,'_blank')
  }

  return (
    <div>
      <header className="header">
        <img src="/logo.png" alt="logo"/>
        <div style={{flex:1}}>
          <div style={{fontSize:20,fontWeight:700}}>HARSH DESIGNER STUDIO</div>
          <div className="tagline">Creative Designs, Delivered Professionally</div>
        </div>
        <div>
          <button className="button" onClick={()=> setView('order')}>Order Now / ऑर्डर करें</button>
        </div>
      </header>

      <main className="container">
        {view==='home' && <>
          <section className="grid" style={{gridTemplateColumns:'1fr 320px',gap:18}}>
            <div>
              <h2>Welcome to HARSH DESIGNER STUDIO</h2>
              <p className="small">Professional logos, business cards, posters, banners & social media posts — order online / पेशेवर डिज़ाइन सर्विसेस - ऑनलाइन ऑर्डर करें</p>
              <div style={{marginTop:12}} className="card">
                <h3>Services / सेवाएँ</h3>
                <ul>
                  <li>Logo Design / लोगो डिज़ाइन</li>
                  <li>Business Cards / बिजनेस कार्ड</li>
                  <li>Posters & Banners / पोस्टर और बैनर</li>
                  <li>Social Media Posts / सोशल मीडिया पोस्ट्स</li>
                </ul>
              </div>

              <div style={{marginTop:12}} className="card">
                <h3>How it works / प्रक्रिया</h3>
                <ol>
                  <li>Place order & upload payment screenshot (50% advance)</li>
                  <li>We verify & confirm → You see ✔️ Order Confirmed</li>
                  <li>We deliver design, pay remaining 50%</li>
                </ol>
              </div>
            </div>

            <aside>
              <div className="card" style={{textAlign:'center'}}>
                <h4>Pay 50% Advance / 50% अग्रिम भुगतान</h4>
                <img className="qr" src="/qr.png" alt="QR"/>
                <div className="small" style={{marginTop:8}}>UPI ID: <strong>{OWNER_UPI}</strong></div>
                <div style={{marginTop:8}}>
                  <button className="button" onClick={()=> startPaymentTimer()}>Start 2-min Timer / टाइमर शुरू करें</button>
                </div>
                <div style={{marginTop:8}} className="small">Time left: <span className="timer">{timer>0? formatTime(timer): '00:00'}</span></div>
              </div>

              <div style={{marginTop:12}} className="card">
                <h4>Portfolio</h4>
                <div className="portfolio-placeholder">Portfolio coming soon — you can add your work later.</div>
                <div style={{textAlign:'center', marginTop:8}}>
                  <button className="button" onClick={()=> setView('portfolio')}>Open Portfolio</button>
                </div>
              </div>
            </aside>
          </section>
        </>}

        {view==='order' && <>
          <h2>Place Order / ऑर्डर फॉर्म</h2>
          <form onSubmit={handlePlaceOrder} className="grid" style={{gridTemplateColumns:'1fr 1fr',gap:12}}>
            <div>
              <label className="small">Name / नाम</label>
              <input className="input" value={form.customerName} onChange={e=> updateField('customerName', e.target.value)} />
            </div>
            <div>
              <label className="small">Email / ईमेल</label>
              <input className="input" type="email" value={form.email} onChange={e=> updateField('email', e.target.value)} />
            </div>
            <div>
              <label className="small">WhatsApp Number / व्हाट्सऐप</label>
              <input className="input" value={form.phone} onChange={e=> updateField('phone', e.target.value)} placeholder="10 digits" />
            </div>
            <div>
              <label className="small">Company / ब्रांड (optional)</label>
              <input className="input" value={form.company} onChange={e=> updateField('company', e.target.value)} />
            </div>

            <div style={{gridColumn:'1 / -1'}}>
              <label className="small">Service Type / सेवा चुनें</label>
              <select className="input" value={form.designType} onChange={e=> updateField('designType', e.target.value)}>
                <option value="logo">Logo / लोगो</option>
                <option value="business_card">Business Card / बिजनेस कार्ड</option>
                <option value="poster">Poster / पोस्टर</option>
                <option value="banner">Banner / बैनर</option>
                <option value="social_post">Social Media Post / सोशल पोस्ट</option>
                <option value="other">Other / अन्य</option>
              </select>
            </div>

            <div style={{gridColumn:'1 / -1'}}>
              <label className="small">Details / विवरण</label>
              <textarea className="input" rows={5} value={form.details} onChange={e=> updateField('details', e.target.value)} />
            </div>

            <div>
              <label className="small">Reference File (optional) / संदर्भ फाइल (ऐच्छिक)</label>
              <input ref={fileRef} type="file" className="input" />
            </div>

            <div>
              <label className="small">Payment Screenshot (Required) / भुगतान स्क्रीनशॉट (जरूरी)</label>
              <input ref={proofRef} type="file" accept="image/*" className="input" />
            </div>

            <div style={{gridColumn:'1 / -1', display:'flex', gap:10, alignItems:'center'}}>
              <label style={{display:'flex',alignItems:'center',gap:8}}><input type="checkbox" required /> <span className="small">I confirm I paid 50% to {OWNER_UPI} / मैं पुष्टि करता हूँ कि मैंने 50% अग्रिम अदा कर दिया है</span></label>
              <button className="button" type="submit">Submit Order / ऑर्डर जमा करें</button>
            </div>

            <div style={{gridColumn:'1 / -1'}} className="small">After submitting, click the WhatsApp button to send a confirmation message to Harsh and attach the screenshot in WhatsApp too. / सबमिट करने के बाद Harsh को सूचित करने के लिए WhatsApp बटन पर क्लिक करें और स्क्रीनशॉट अटैच करें।</div>
          </form>
        </>}

        {view==='orders' && <>
          <h2>Your Orders / आपके ऑर्डर</h2>
          {orders.length===0 && <div className="card">No orders yet / अभी कोई ऑर्डर नहीं</div>}
          <div style={{display:'grid',gap:10, marginTop:10}}>
            {orders.map(o=>(
              <div key={o.id} className="order-row">
                <div>
                  <div style={{fontWeight:700}}>{o.customerName} — {o.designType}</div>
                  <div className="small">Order ID: {o.id}</div>
                  <div className="small">Status: {o.status==='pending-payment'? 'Pending Confirmation ⏳ / पुष्टि लंबित' : o.status==='in-progress'? 'Payment Verified / भुगतान सत्यापित' : 'Order Confirmed ✔️'}</div>
                </div>
                <div style={{display:'flex',flexDirection:'column',gap:8,alignItems:'flex-end'}}>
                  <button className="button" onClick={()=> openWhatsAppPrefilled(o)}>Message on WhatsApp</button>
                  {o.proofName && <a href={o.proofData} download={o.proofName} className="small">Download Screenshot</a>}
                </div>
              </div>
            ))}
          </div>
        </>}

        {view==='portfolio' && <>
          <h2>Portfolio / पोर्टफोलियो</h2>
          <div className="card">
            <div className="portfolio-placeholder">Portfolio Coming Soon — You can add items later from Admin. / पोर्टफोलियो जल्द ही जोड़ा जाएगा।</div>
            <div style={{marginTop:10}}>
              <button className="button" onClick={()=> alert('Portfolio editor available in admin panel once you add items')}>Edit Portfolio (Admin)</button>
            </div>
          </div>
        </>}

        {view==='track' && <>
          <h2>Track Order / ऑर्डर ट्रैक करें</h2>
          <div style={{display:'flex',gap:8,alignItems:'center'}}>
            <input className="input" placeholder="Enter Order ID" value={trackId} onChange={e=> setTrackId(e.target.value)} />
            <button className="button" onClick={customerTrack}>Track / ट्रैक करें</button>
          </div>
          <div style={{marginTop:10}}>
            {orders.filter(x=> x.id===trackId.trim()).map(o=>(
              <div key={o.id} className="card">
                <div style={{fontWeight:700}}>{o.customerName} — {o.designType}</div>
                <div className="small">Status timeline:</div>
                <ul>
                  <li>{o.paidUpfront? 'Payment Received ✅ / भुगतान प्राप्त' : 'Payment Pending ⏳'}</li>
                  <li>{o.status==='in-progress' || o.status==='confirmed' ? 'Work In Progress 🛠️' : 'Waiting for confirmation'}</li>
                  <li>{o.status==='confirmed' ? 'Design Sent / डिज़ाइन भेजा गया' : ''}</li>
                </ul>
              </div>
            ))}
          </div>
        </>}

        {view==='admin' && <>
          <h2>Admin Panel</h2>
          {!adminAuth && <>
            <div className="admin-login card">
              <h3>Login</h3>
              <form onSubmit={adminLogin}>
                <div className="small">Username</div>
                <input className="input" value={adminCred.user} onChange={e=> setAdminCred(c=>({...c,user:e.target.value}))} />
                <div className="small">Password</div>
                <input className="input" type="password" value={adminCred.pass} onChange={e=> setAdminCred(c=>({...c,pass:e.target.value}))} />
                <div style={{marginTop:8}}>
                  <button className="button" type="submit">Login</button>
                </div>
              </form>
            </div>
          </>}

          {adminAuth && <>
            <div style={{display:'flex',justifyContent:'space-between',alignItems:'center',marginBottom:12}}>
              <div>Welcome, Harsh</div>
              <div>
                <button className="button" onClick={adminLogout}>Logout</button>
              </div>
            </div>

            <div style={{display:'grid',gap:8}}>
              {orders.length===0 && <div className="card">No orders</div>}
              {orders.map(o=>(
                <div key={o.id} className="card" style={{display:'flex',justifyContent:'space-between',alignItems:'center'}}>
                  <div>
                    <div style={{fontWeight:700}}>{o.customerName} — {o.designType}</div>
                    <div className="small">Email: {o.email} | WhatsApp: {o.phone}</div>
                    <div className="small">Order ID: {o.id}</div>
                    <div className="small">Uploaded: {o.proofName}</div>
                  </div>
                  <div style={{display:'flex',flexDirection:'column',gap:8,alignItems:'flex-end'}}>
                    <a href={o.proofData} download={o.proofName} className="small">Download Screenshot</a>
                    <div style={{display:'flex',gap:8}}>
                      <button className="button" onClick={()=> adminVerifyPayment(o.id)}>Verify Payment</button>
                      <button className="button" onClick={()=> adminConfirmOrder(o.id)}>Confirm Order</button>
                    </div>
                    <button className="small" onClick={()=> sendWhatsAppFromAdmin(o)}>Send WhatsApp</button>
                  </div>
                </div>
              ))}
            </div>
          </>}
        </>}

      </main>

      <div className="fab-wa" title="Chat on WhatsApp" onClick={()=> window.open(`https://wa.me/${OWNER_PHONE}`,'_blank')}>💬</div>

      <footer className="footer">
        © {new Date().getFullYear()} HARSH DESIGNER STUDIO • UPI: {OWNER_UPI}
      </footer>
    </div>
  )
}
'''
(src / "App.jsx").write_text(app_jsx)

# README with deployment instructions
readme = f"""
HARSH DESIGNER STUDIO - Ready React (Vite) project
=================================================

This project is a lightweight React + Vite frontend implementing:
- Manual UPI payment flow (UPI: {OWNER_UPI})
- QR and logo included (logo.png, qr.png)
- 2-minute timer for payment
- Mandatory payment screenshot upload (stored locally in browser for demo)
- WhatsApp prefilled messaging (wa.me)
- Admin panel with username 'Harsh' and password 'harsh123' (demo only)
- Portfolio placeholder (editable later)
- Bilingual labels (English + Hindi)

How to run locally:
1. Make sure Node.js 18+ is installed.
2. In the project folder, run:
   npm install
   npm run dev
3. Open the shown local URL (usually http://localhost:5173)

How to deploy to Vercel:
1. Make sure you have a git repo. Commit this project.
2. Go to https://vercel.com and import the repository.
3. Vercel should detect a Vite project. Build command: npm run build, Output directory: dist.
4. After deploy, add a custom domain 'www.harshdesignerstudio.com' in Vercel settings and follow DNS instructions.

Notes:
- This demo stores orders in browser localStorage. For production, add a backend (Node/MongoDB or Firebase) and file storage (S3).
- WhatsApp automatic sending needs WhatsApp Business API / Twilio integration; currently the site opens wa.me links for manual confirmation.

Files included:
- package.json
- index.html
- src/ (App.jsx, main.jsx, styles.css, logo.png, qr.png)

"""
(project_dir / "README.md").write_text(readme)

# Zip the project
shutil.make_archive(str(project_dir), 'zip', root_dir=project_dir)

zip_path = str(project_dir) + ".zip"
print("Created project zip at:", zip_path)

zip_link = "sandbox:" + zip_path
zip_link

# 1. unzip (agar zip download kiya hai)
unzip /path/to/harsh-designer-studio.zip -d harsh-designer-studio
cd harsh-designer-studio

# 2. init git, commit
git init
git add .
git commit -m "Initial HARSH DESIGNER STUDIO frontend"
# create a repo (locally) and push to your GitHub account
# replace <your-username> and repo name
git branch -M main
git remote add origin https://github.com/<your-username>/harsh-designer-studio.git
git push -u origin main
cd harsh-designer-studio
npm install
npm run dev
# open http://localhost:5173
