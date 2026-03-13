import streamlit as st
import json, os, time
import plotly.graph_objects as go
from datetime import datetime, timedelta
import requests

# ─── PAGE CONFIG ───
st.set_page_config(page_title="JARVIS-C AutoPilot", page_icon="⚡", layout="centered")

# ─── IMPORTS (local files) ───
import sys
sys.path.insert(0, ".")
from trade_engine import TradeEngine

engine = TradeEngine()

# ─── DARK IRON MAN CSS ───
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Rajdhani:wght@300;500;700&display=swap');
.stApp { background:#000; }
#MainMenu,footer,header{visibility:hidden}
.block-container{padding:1rem;max-width:500px;margin:auto}
.stark-title{font-family:'Orbitron',monospace;font-size:1.5rem;font-weight:900;text-align:center;
  background:linear-gradient(135deg,#e63946,#ff9f1c);-webkit-background-clip:text;
  -webkit-text-fill-color:transparent;letter-spacing:3px;margin:0}
.stark-sub{font-family:'Rajdhani';text-align:center;color:#555;font-size:.75rem;
  letter-spacing:5px;margin-bottom:1.5rem}
.bal-card{background:linear-gradient(135deg,#110000,#0a0a0a);border:1px solid rgba(230,57,70,.35);
  border-radius:16px;padding:20px;text-align:center;margin-bottom:1.5rem;
  box-shadow:0 0 40px rgba(230,57,70,.1)}
.bal-amt{font-family:'Orbitron';font-size:2.2rem;font-weight:900;color:#ff9f1c;
  text-shadow:0 0 25px rgba(255,159,28,.5)}
.bal-lbl{font-family:'Rajdhani';color:#555;font-size:.7rem;letter-spacing:4px;text-transform:uppercase}
.bal-target{font-family:'Rajdhani';color:#e63946;font-size:.85rem;margin-top:5px}
.prog-bar{background:#111;border-radius:6px;height:8px;margin:12px 0;overflow:hidden}
.prog-fill{height:100%;border-radius:6px;background:linear-gradient(90deg,#e63946,#ff9f1c);
  box-shadow:0 0 12px rgba(255,159,28,.6)}
.autopilot-on{background:rgba(74,222,128,.08);border:1px solid rgba(74,222,128,.4);
  border-radius:12px;padding:14px;text-align:center;font-family:'Orbitron';
  font-size:.85rem;color:#4ade80;letter-spacing:2px;margin:1rem 0;
  box-shadow:0 0 20px rgba(74,222,128,.15)}
.autopilot-off{background:rgba(230,57,70,.06);border:1px solid rgba(230,57,70,.25);
  border-radius:12px;padding:14px;text-align:center;font-family:'Orbitron';
  font-size:.85rem;color:#666;letter-spacing:2px;margin:1rem 0}
.log-box{background:#050810;border:1px solid rgba(230,57,70,.15);border-radius:10px;
  padding:14px;font-family:'Courier New';font-size:.72rem;color:#4ade80;
  max-height:200px;overflow-y:auto;line-height:1.9}
.stButton>button{font-family:'Orbitron' !important;font-weight:700 !important;
  font-size:.72rem !important;letter-spacing:1px !important;border-radius:10px !important;
  border:1px solid rgba(230,57,70,.4) !important;background:linear-gradient(135deg,#150303,#0a0000) !important;
  color:#ff9f1c !important;width:100% !important;padding:.85rem !important;
  transition:all .2s !important}
.stButton>button:hover{border-color:#e63946 !important;color:#fff !important;
  box-shadow:0 0 25px rgba(230,57,70,.3) !important;transform:translateY(-2px) !important}
[data-testid="metric-container"]{background:#0d1117 !important;border:1px solid rgba(255,255,255,.06) !important;
  border-radius:10px !important;padding:10px !important}
[data-testid="stMetricValue"]{font-family:'Orbitron' !important;font-size:1rem !important;color:#ff9f1c !important}
[data-testid="stMetricLabel"]{font-family:'Rajdhani' !important;color:#555 !important;
  font-size:.7rem !important;letter-spacing:2px !important}
hr{border-color:rgba(230,57,70,.15) !important}
</style>
""", unsafe_allow_html=True)

# ─── SESSION STATE ───
if "logs" not in st.session_state: st.session_state.logs = []
if "show_chart" not in st.session_state: st.session_state.show_chart = False
if "show_settings" not in st.session_state: st.session_state.show_settings = False

def get_logs():
    if os.path.exists("stark_logs.json"):
        with open("stark_logs.json") as f:
            return json.load(f)
    return ["[SYSTEM] JARVIS-C online. Awaiting orders..."]

def get_trades():
    if os.path.exists("stark_trades.json"):
        with open("stark_trades.json") as f:
            return json.load(f)
    return []

def get_btc():
    try:
        r = requests.get("https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT", timeout=3)
        return float(r.json()["price"])
    except: return 0.0

# ─── HEADER ───
st.markdown('<p class="stark-title">⚡ JARVIS-C</p>', unsafe_allow_html=True)
st.markdown('<p class="stark-sub">AUTO-PILOT COMMAND CENTER</p>', unsafe_allow_html=True)

# ─── BALANCE CARD ───
bal        = engine.config["balance"]
target     = engine.config["target"]
start      = engine.config["start_balance"]
progress   = min(max(((bal - start) / (target - start)) * 100, 0.3), 100)
status     = engine.load_status()
is_auto    = status.get("autopilot", False)

st.markdown(f"""
<div class="bal-card">
  <div class="bal-lbl">CURRENT BALANCE</div>
  <div class="bal-amt">${bal:,.2f}</div>
  <div class="bal-target">TARGET $1,000,000 &nbsp;·&nbsp; NEEDED ${target-bal:,.0f}</div>
  <div class="prog-bar"><div class="prog-fill" style="width:{progress:.2f}%"></div></div>
  <div style="font-family:Rajdhani;color:#444;font-size:.72rem;letter-spacing:3px">{progress:.2f}% COMPLETE</div>
</div>
""", unsafe_allow_html=True)

# ─── STATS ───
trades  = get_trades()
wins    = len([t for t in trades if t.get("pnl",0) > 0])
wr      = f"{wins/max(len(trades),1)*100:.0f}%"
total_pnl = bal - start
c1,c2,c3,c4 = st.columns(4)
c1.metric("TRADES",  len(trades))
c2.metric("WINS",    wins)
c3.metric("WIN RATE",wr)
c4.metric("P&L",     f"${total_pnl:+,.0f}")

st.markdown("<br>", unsafe_allow_html=True)

# ─────────────────────────────────────────
# AUTO-PILOT MASTER BUTTON — THE BIG ONE
# ─────────────────────────────────────────

if is_auto:
    st.markdown('<div class="autopilot-on">● AUTO-PILOT ACTIVE — HUNTING SETUPS</div>', unsafe_allow_html=True)
    if st.button("🔴 AUTO-PILOT OFF", key="ap"):
        engine.set_autopilot(False)
        st.rerun()
else:
    st.markdown('<div class="autopilot-off">○ AUTO-PILOT STANDBY</div>', unsafe_allow_html=True)
    if st.button("🟢 AUTO-PILOT ON", key="ap"):
        engine.set_autopilot(True)
        engine.log("🚀 Auto-Pilot ON! TradingView alerts accept hone lagy.")
        st.rerun()

st.markdown("<br>", unsafe_allow_html=True)

# ─── ACTION BUTTONS ───
col1, col2 = st.columns(2)

with col1:
    if st.button("📈 VIEW PROGRESS"):
        st.session_state.show_chart = not st.session_state.show_chart
        st.rerun()

with col2:
    if st.button("⚙️ SETTINGS"):
        st.session_state.show_settings = not st.session_state.show_settings
        st.rerun()

col3, col4 = st.columns(2)
with col3:
    if st.button("🔄 REFRESH DATA"):
        price = get_btc()
        engine.log(f"🔄 Manual refresh | BTC: ${price:,.2f}")
        st.rerun()

with col4:
    if st.button("📋 TRADE HISTORY"):
        st.session_state.show_trades = not st.session_state.get("show_trades", False)
        st.rerun()

# ─── PROGRESS CHART ───
if st.session_state.show_chart:
    st.markdown("---")
    st.markdown('<p style="font-family:Orbitron;color:#e63946;font-size:.75rem;letter-spacing:2px">📊 EMPIRE GROWTH</p>', unsafe_allow_html=True)

    if trades:
        cumulative = start
        x, y = [trades[0]["time"]], [start]
        for t in trades:
            cumulative += t.get("pnl", 0)
            x.append(t["time"])
            y.append(round(cumulative, 2))
        x.append(datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
        y.append(bal)
    else:
        x = [datetime.now().strftime("%Y-%m-%d %H:%M:%S")]
        y = [bal]

    fig = go.Figure()
    fig.add_trace(go.Scatter(x=x, y=y, fill='tozeroy',
        fillcolor='rgba(230,57,70,0.08)',
        line=dict(color='#e63946', width=2.5),
        hovertemplate='$%{y:,.0f}<extra></extra>'))
    fig.add_hline(y=1000000, line_dash="dot", line_color="#ff9f1c",
        annotation_text="🎯 $1M", annotation_font_color="#ff9f1c", annotation_font_size=11)
    fig.update_layout(
        paper_bgcolor='rgba(0,0,0,0)', plot_bgcolor='rgba(0,0,0,0)',
        margin=dict(l=0,r=0,t=10,b=0), height=220, showlegend=False,
        xaxis=dict(showgrid=False, showticklabels=False),
        yaxis=dict(showgrid=True, gridcolor='rgba(255,255,255,0.04)',
            tickformat='$,.0f', tickfont=dict(color='#444', size=9)))
    st.plotly_chart(fig, use_container_width=True)

# ─── SETTINGS PANEL ───
if st.session_state.show_settings:
    st.markdown("---")
    st.markdown('<p style="font-family:Orbitron;color:#e63946;font-size:.75rem;letter-spacing:2px">⚙️ SETTINGS</p>', unsafe_allow_html=True)

    col_a, col_b = st.columns(2)
    with col_a:
        new_bal = st.number_input("Balance ($)", value=float(engine.config["balance"]), step=100.0)
    with col_b:
        new_risk = st.number_input("Risk %", value=float(engine.config["risk_pct"]), min_value=0.5, max_value=5.0, step=0.5)

    paper = st.toggle("Paper Mode (Safe)", value=engine.config.get("paper_mode", True))

    st.markdown('<p style="font-family:Rajdhani;color:#666;font-size:.8rem;letter-spacing:2px">BINANCE API (Live trading ke liye)</p>', unsafe_allow_html=True)
    api_key    = st.text_input("API Key",    value=engine.config.get("api_key",""),    type="password")
    api_secret = st.text_input("API Secret", value=engine.config.get("api_secret",""), type="password")

    if st.button("💾 SAVE SETTINGS"):
        engine.config["balance"]    = new_bal
        engine.config["risk_pct"]   = new_risk
        engine.config["paper_mode"] = paper
        engine.config["api_key"]    = api_key
        engine.config["api_secret"] = api_secret
        engine.save_config()
        engine.log(f"⚙️ Settings saved | Balance: ${new_bal:,.2f} | Risk: {new_risk}% | Paper: {paper}")
        st.success("✅ Settings saved!")
        st.rerun()

    st.markdown("""---
**📡 TradingView Webhook Setup:**
1. TradingView pe alert banao
2. Webhook URL: `http://YOUR_IP:5000/webhook`
3. Alert Message (JSON format):
```json
{
  "signal": "BUY",
  "symbol": "BTCUSDT",
  "price": {{close}},
  "stop_loss": {{low}},
  "liquidity_swept": true
}
```
""")

# ─── TRADE HISTORY ───
if st.session_state.get("show_trades"):
    st.markdown("---")
    st.markdown('<p style="font-family:Orbitron;color:#e63946;font-size:.75rem;letter-spacing:2px">📋 RECENT TRADES</p>', unsafe_allow_html=True)
    if trades:
        for t in reversed(trades[-5:]):
            pnl_color = "#4ade80" if t.get("pnl",0) >= 0 else "#e63946"
            st.markdown(f"""
            <div style="background:#0d1117;border:1px solid rgba(255,255,255,.06);border-radius:8px;
              padding:10px;margin-bottom:8px;font-family:Rajdhani;font-size:.85rem">
              <span style="color:#888">{t.get('time','—')}</span> &nbsp;
              <span style="color:#ff9f1c;font-weight:700">{t.get('signal','—')}</span> &nbsp;
              <span style="color:#fff">${t.get('price',0):,.0f}</span> &nbsp;
              <span style="color:{pnl_color};font-weight:700">PnL: ${t.get('pnl',0):+,.2f}</span>
            </div>""", unsafe_allow_html=True)
    else:
        st.markdown('<p style="color:#555;font-family:Rajdhani;text-align:center">Abhi koi trade nahi hua.</p>', unsafe_allow_html=True)

# ─── SYSTEM LOG ───
st.markdown("---")
st.markdown('<p style="font-family:Orbitron;color:#e63946;font-size:.7rem;letter-spacing:2px">⚡ SYSTEM LOG</p>', unsafe_allow_html=True)
logs = get_logs()
log_html = "<div class='log-box'>" + "<br>".join(reversed(logs[-15:])) + "</div>"
st.markdown(log_html, unsafe_allow_html=True)

# ─── STATUS BAR ───
st.markdown("<br>", unsafe_allow_html=True)
btc = get_btc()
mode_color = "#4ade80" if is_auto else "#555"
mode_text  = "AUTO-PILOT ● ACTIVE" if is_auto else "AUTO-PILOT ○ STANDBY"
st.markdown(f"""
<div style="text-align:center;font-family:Rajdhani;color:{mode_color};font-size:.8rem;
  letter-spacing:2px;background:rgba(255,255,255,.02);border:1px solid rgba(255,255,255,.05);
  border-radius:8px;padding:10px;">
  {mode_text} &nbsp;|&nbsp; BTC: ${btc:,.0f} &nbsp;|&nbsp; Risk: ${bal*engine.config['risk_pct']/100:,.0f}
</div>
<br>
<p style="text-align:center;font-family:Rajdhani;color:#222;font-size:.65rem;letter-spacing:3px">
  STARK INDUSTRIES © 2025 | JARVIS-C AUTO-PILOT v3.0
</p>
""", unsafe_allow_html=True)
