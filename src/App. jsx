import { useState, useEffect } from "react";

// ─── PLANS CONFIG ────────────────────────────────────────────────────────────
const PLANS = {
  free: {
    id: "free",
    name: "Free",
    price: 0,
    label: "₦0 / month",
    limit: 5,
    badge: null,
    color: "#444",
    perks: ["5 generations/month", "Captions only", "All platforms"],
  },
  creator: {
    id: "creator",
    name: "Creator",
    price: 250000, // in kobo (₦2,500)
    label: "₦2,500 / month",
    limit: 100,
    badge: "Most Popular",
    color: "#c8922a",
    perks: ["100 generations/month", "All content types", "All platforms", "Priority speed"],
  },
  pro: {
    id: "pro",
    name: "Pro",
    price: 600000, // in kobo (₦6,000)
    label: "₦6,000 / month",
    limit: Infinity,
    badge: "Best Value",
    color: "#e8b84b",
    perks: ["Unlimited generations", "All content types", "All platforms", "Priority speed", "Early access to new features"],
  },
};

const CONTENT_TYPES = [
  { id: "caption", label: "📝 Caption", free: true },
  { id: "script", label: "🎬 Reel Script", free: false },
  { id: "hashtags", label: "#️⃣ Hashtags", free: false },
  { id: "week_ideas", label: "📅 Week Ideas", free: false },
];

const TONES = ["Funny 😂", "Motivational 💪", "Relatable 😅", "Informative 📚", "Trendy 🔥"];

const SYSTEM_PROMPT = `You are NaijaContentAI, an expert social media content creator specializing in Nigerian and African content for Facebook, TikTok, and Instagram.
Generate engaging, culturally relevant content that resonates with Nigerian audiences. Use Nigerian expressions, Pidgin English where appropriate, and references to Nigerian culture, food, daily life, and humor.
Always be energetic, relatable, and use emojis naturally. Respond ONLY with the requested content — no explanations or preamble.`;

// ─── PAYSTACK HELPER ─────────────────────────────────────────────────────────
function initPaystack({ email, amount, plan, onSuccess }) {
  // NOTE: Replace YOUR_PAYSTACK_PUBLIC_KEY below with your actual Paystack public key
  // Get it free at: https://dashboard.paystack.com/#/settings/developer
  const PAYSTACK_PUBLIC_KEY = "YOUR_PAYSTACK_PUBLIC_KEY";

  if (!window.PaystackPop) {
    alert("Payment system loading... please try again in a moment.");
    return;
  }

  const handler = window.PaystackPop.setup({
    key: PAYSTACK_PUBLIC_KEY,
    email,
    amount, // in kobo
    currency: "NGN",
    ref: `naija_${plan}_${Date.now()}`,
    metadata: { plan },
    callback: (response) => onSuccess(response),
    onClose: () => {},
  });

  handler.openIframe();
}

// ─── MAIN APP ────────────────────────────────────────────────────────────────
export default function NaijaContentAIPro() {
  const [screen, setScreen] = useState("landing"); // landing | app | upgrade
  const [plan, setPlan] = useState("free");
  const [usageCount, setUsageCount] = useState(0);
  const [email, setEmail] = useState("");
  const [emailInput, setEmailInput] = useState("");
  const [topic, setTopic] = useState("");
  const [contentType, setContentType] = useState("caption");
  const [tone, setTone] = useState("Funny 😂");
  const [platform, setPlatform] = useState("Facebook");
  const [result, setResult] = useState("");
  const [loading, setLoading] = useState(false);
  const [copied, setCopied] = useState(false);
  const [error, setError] = useState("");
  const [paystackLoaded, setPaystackLoaded] = useState(false);

  // Load Paystack script
  useEffect(() => {
    const script = document.createElement("script");
    script.src = "https://js.paystack.co/v1/inline.js";
    script.onload = () => setPaystackLoaded(true);
    document.body.appendChild(script);

    // Load saved state
    const saved = localStorage.getItem("naija_ai_user");
    if (saved) {
      const { plan: p, usage, em } = JSON.parse(saved);
      setPlan(p || "free");
      setUsageCount(usage || 0);
      setEmail(em || "");
      if (em) setScreen("app");
    }
  }, []);

  const saveState = (newPlan, newUsage, em) => {
    localStorage.setItem("naija_ai_user", JSON.stringify({ plan: newPlan, usage: newUsage, em }));
  };

  const currentPlan = PLANS[plan];
  const remaining = currentPlan.limit === Infinity ? "∞" : Math.max(0, currentPlan.limit - usageCount);
  const canGenerate = currentPlan.limit === Infinity || usageCount < currentPlan.limit;
  const canUseType = plan !== "free" || contentType === "caption";

  const handleStart = () => {
    if (!emailInput.includes("@")) return;
    setEmail(emailInput);
    saveState("free", 0, emailInput);
    setScreen("app");
  };

  const generate = async () => {
    if (!topic.trim() || !canGenerate || !canUseType) return;
    setLoading(true);
    setResult("");
    setError("");
    setCopied(false);

    const typePrompts = {
      caption: `Write a ${tone} social media caption for ${platform} about: "${topic}". Use Nigerian expressions naturally, 3–5 sentences, end with a call to action.`,
      script: `Write a short ${tone} Reel/TikTok script for ${platform} about: "${topic}".\n\nHOOK (0–3 sec):\nMAIN CONTENT (3–25 sec):\nCALL TO ACTION (25–30 sec):\n\nUse Nigerian expressions, make it punchy.`,
      hashtags: `Generate a hashtag pack for a ${platform} post about: "${topic}" targeting Nigerian audiences. Give 20 hashtags grouped as:\n🔥 Trending | 🇳🇬 Nigerian | 📌 Niche`,
      week_ideas: `Generate 7 content ideas for the week about: "${topic}" for a Nigerian ${platform} creator with a ${tone} tone.\nFormat: Day + Title + One-line description.`,
    };

    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: SYSTEM_PROMPT,
          messages: [{ role: "user", content: typePrompts[contentType] }],
        }),
      });
      const data = await res.json();
      const text = data?.content?.[0]?.text || "";
      if (!text) throw new Error();
      setResult(text);
      const newCount = usageCount + 1;
      setUsageCount(newCount);
      saveState(plan, newCount, email);
    } catch {
      setError("Something went wrong. Please try again.");
    } finally {
      setLoading(false);
    }
  };

  const handleUpgrade = (targetPlan) => {
    initPaystack({
      email,
      amount: PLANS[targetPlan].price,
      plan: targetPlan,
      onSuccess: (response) => {
        setPlan(targetPlan);
        setUsageCount(0);
        saveState(targetPlan, 0, email);
        setScreen("app");
        alert(`🎉 Payment successful! You're now on the ${PLANS[targetPlan].name} plan.`);
      },
    });
  };

  // ── LANDING SCREEN ──
  if (screen === "landing") return (
    <div style={styles.root}>
      <div style={styles.grain} />
      <div style={{ maxWidth: 500, margin: "0 auto", padding: "40px 24px", textAlign: "center" }}>
        <div style={styles.tagline}>🇳🇬 For Nigerian Creators</div>
        <h1 style={styles.hero}>Naija<span style={{ color: "#e8b84b" }}>Content</span>AI</h1>
        <p style={styles.sub}>Generate fire captions, scripts & hashtags for Facebook, TikTok & Instagram — in seconds.</p>

        <div style={styles.card}>
          <p style={{ color: "#c8922a", fontSize: 13, letterSpacing: 2, textTransform: "uppercase", marginBottom: 16 }}>Get Started Free</p>
          <input
            type="email"
            placeholder="Enter your email address"
            value={emailInput}
            onChange={e => setEmailInput(e.target.value)}
            onKeyDown={e => e.key === "Enter" && handleStart()}
            style={styles.input}
          />
          <button onClick={handleStart} style={styles.btnPrimary}>
            Start Creating Free ⚡
          </button>
          <p style={{ color: "#444", fontSize: 12, marginTop: 12 }}>5 free generations • No credit card needed</p>
        </div>

        {/* Mini plan preview */}
        <div style={{ display: "flex", gap: 10, marginTop: 32, justifyContent: "center", flexWrap: "wrap" }}>
          {Object.values(PLANS).map(p => (
            <div key={p.id} style={{ background: "#0f0f0f", border: `1px solid ${p.color}33`, borderRadius: 10, padding: "12px 18px", minWidth: 130 }}>
              <div style={{ color: p.color, fontWeight: 700, fontSize: 14 }}>{p.name}</div>
              <div style={{ color: "#888", fontSize: 12 }}>{p.label}</div>
              <div style={{ color: "#555", fontSize: 11, marginTop: 4 }}>{p.limit === Infinity ? "Unlimited" : `${p.limit} gen/mo`}</div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  // ── UPGRADE SCREEN ──
  if (screen === "upgrade") return (
    <div style={styles.root}>
      <div style={styles.grain} />
      <div style={{ maxWidth: 600, margin: "0 auto", padding: "32px 20px" }}>
        <button onClick={() => setScreen("app")} style={styles.backBtn}>← Back</button>
        <h2 style={{ ...styles.hero, fontSize: "2rem", marginBottom: 8 }}>Choose Your Plan</h2>
        <p style={{ color: "#666", textAlign: "center", marginBottom: 32, fontSize: 14 }}>Unlock more generations and content types</p>

        <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
          {Object.values(PLANS).map(p => (
            <div key={p.id} style={{
              background: plan === p.id ? "#1a0f00" : "#0f0f0f",
              border: `1px solid ${plan === p.id ? p.color : "#1e1e1e"}`,
              borderRadius: 14, padding: "20px 24px",
              position: "relative", overflow: "hidden",
            }}>
              {p.badge && (
                <div style={{ position: "absolute", top: 12, right: 16, background: p.color, color: "#000", fontSize: 10, fontWeight: 800, padding: "3px 10px", borderRadius: 20, letterSpacing: 1 }}>
                  {p.badge.toUpperCase()}
                </div>
              )}
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", flexWrap: "wrap", gap: 12 }}>
                <div>
                  <div style={{ color: p.color, fontSize: 20, fontWeight: 800 }}>{p.name}</div>
                  <div style={{ color: "#aaa", fontSize: 15, marginTop: 2 }}>{p.label}</div>
                </div>
                {plan === p.id ? (
                  <div style={{ color: "#4a8a4a", fontSize: 13, border: "1px solid #4a8a4a", borderRadius: 20, padding: "6px 16px" }}>✓ Current Plan</div>
                ) : p.id === "free" ? (
                  <div style={{ color: "#444", fontSize: 13 }}>Default</div>
                ) : (
                  <button onClick={() => handleUpgrade(p.id)} style={{ ...styles.btnPrimary, width: "auto", padding: "10px 24px", fontSize: 14 }}>
                    Upgrade →
                  </button>
                )}
              </div>
              <div style={{ display: "flex", flexWrap: "wrap", gap: "6px 16px", marginTop: 14 }}>
                {p.perks.map(perk => (
                  <span key={perk} style={{ color: "#666", fontSize: 12 }}>✓ {perk}</span>
                ))}
              </div>
            </div>
          ))}
        </div>

        <p style={{ textAlign: "center", color: "#333", fontSize: 12, marginTop: 24 }}>
          Payments secured by Paystack 🔒 • Cancel anytime
        </p>
      </div>
    </div>
  );

  // ── MAIN APP SCREEN ──
  return (
    <div style={styles.root}>
      <div style={styles.grain} />

      {/* Top bar */}
      <div style={styles.topbar}>
        <div style={{ fontWeight: 800, fontSize: 16, color: "#e8b84b" }}>NaijaContentAI</div>
        <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
          <div style={{ fontSize: 12, color: "#666" }}>
            {remaining === "∞" ? "∞ left" : `${remaining}/${currentPlan.limit} left`}
          </div>
          <div style={{
            background: currentPlan.color + "22",
            border: `1px solid ${currentPlan.color}44`,
            color: currentPlan.color,
            fontSize: 11,
            fontWeight: 700,
            padding: "4px 10px",
            borderRadius: 20,
            letterSpacing: 1,
            textTransform: "uppercase",
          }}>
            {currentPlan.name}
          </div>
          {plan !== "pro" && (
            <button onClick={() => setScreen("upgrade")} style={styles.upgradeBtn}>
              Upgrade ↑
            </button>
          )}
        </div>
      </div>

      <div style={{ maxWidth: 660, margin: "0 auto", padding: "24px 20px" }}>

        {/* Usage bar */}
        {currentPlan.limit !== Infinity && (
          <div style={{ marginBottom: 24 }}>
            <div style={{ display: "flex", justifyContent: "space-between", fontSize: 11, color: "#555", marginBottom: 6 }}>
              <span>Generations used this month</span>
              <span>{usageCount} / {currentPlan.limit}</span>
            </div>
            <div style={{ background: "#111", borderRadius: 4, height: 4, overflow: "hidden" }}>
              <div style={{
                height: "100%",
                width: `${Math.min(100, (usageCount / currentPlan.limit) * 100)}%`,
                background: usageCount >= currentPlan.limit ? "#cc3333" : `linear-gradient(90deg, ${currentPlan.color}, #e8b84b)`,
                borderRadius: 4,
                transition: "width 0.4s ease",
              }} />
            </div>
          </div>
        )}

        {/* Content Type */}
        <div style={styles.section}>
          <label style={styles.sectionLabel}>What do you need?</label>
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10 }}>
            {CONTENT_TYPES.map(ct => {
              const locked = plan === "free" && !ct.free;
              return (
                <button key={ct.id} onClick={() => !locked && setContentType(ct.id)} style={{
                  padding: "13px",
                  background: contentType === ct.id ? `linear-gradient(135deg, #c8922a, #e8b84b)` : "#0f0f0f",
                  border: contentType === ct.id ? "none" : "1px solid #1e1e1e",
                  borderRadius: 10,
                  color: locked ? "#333" : contentType === ct.id ? "#000" : "#777",
                  fontWeight: contentType === ct.id ? 700 : 400,
                  cursor: locked ? "not-allowed" : "pointer",
                  fontSize: 13,
                  transition: "all 0.2s",
                  display: "flex",
                  alignItems: "center",
                  justifyContent: "center",
                  gap: 6,
                }}>
                  {ct.label} {locked && <span style={{ fontSize: 11 }}>🔒</span>}
                </button>
              );
            })}
          </div>
          {plan === "free" && (
            <p style={{ color: "#444", fontSize: 11, marginTop: 8, textAlign: "center" }}>
              🔒 Scripts, Hashtags & Week Ideas require Creator plan —{" "}
              <span onClick={() => setScreen("upgrade")} style={{ color: "#c8922a", cursor: "pointer" }}>Upgrade</span>
            </p>
          )}
        </div>

        {/* Platform + Tone */}
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20, marginBottom: 24 }}>
          <div>
            <label style={styles.sectionLabel}>Platform</label>
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {["Facebook", "TikTok", "Instagram"].map(p => (
                <button key={p} onClick={() => setPlatform(p)} style={{
                  padding: "9px 14px",
                  background: platform === p ? "#c8922a" : "#0f0f0f",
                  border: platform === p ? "none" : "1px solid #1e1e1e",
                  borderRadius: 8, color: platform === p ? "#000" : "#666",
                  fontWeight: platform === p ? 700 : 400,
                  cursor: "pointer", fontSize: 13, textAlign: "left",
                  transition: "all 0.2s",
                }}>{p}</button>
              ))}
            </div>
          </div>
          <div>
            <label style={styles.sectionLabel}>Tone</label>
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {TONES.map(t => (
                <button key={t} onClick={() => setTone(t)} style={{
                  padding: "9px 14px",
                  background: tone === t ? "#1a0f00" : "#0f0f0f",
                  border: tone === t ? "1px solid #c8922a" : "1px solid #1e1e1e",
                  borderRadius: 8, color: tone === t ? "#e8b84b" : "#555",
                  cursor: "pointer", fontSize: 12, textAlign: "left",
                  transition: "all 0.2s",
                }}>{t}</button>
              ))}
            </div>
          </div>
        </div>

        {/* Topic Input */}
        <div style={styles.section}>
          <label style={styles.sectionLabel}>Your Topic</label>
          <textarea
            value={topic}
            onChange={e => setTopic(e.target.value)}
            placeholder="e.g. 'Adulting in Nigeria is not easy', 'My morning routine', 'Lagos traffic wahala'..."
            rows={3}
            style={styles.textarea}
            onFocus={e => e.target.style.borderColor = "#c8922a"}
            onBlur={e => e.target.style.borderColor = "#1e1e1e"}
          />
        </div>

        {/* Generate Button */}
        {!canGenerate ? (
          <div style={{ textAlign: "center" }}>
            <div style={{ color: "#cc5533", marginBottom: 12, fontSize: 14 }}>
              You've used all your {currentPlan.limit} free generations this month.
            </div>
            <button onClick={() => setScreen("upgrade")} style={styles.btnPrimary}>
              Upgrade to Keep Creating 🚀
            </button>
          </div>
        ) : (
          <button onClick={generate} disabled={loading || !topic.trim() || !canUseType} style={{
            ...styles.btnPrimary,
            background: loading || !topic.trim() || !canUseType ? "#1a1a1a" : "linear-gradient(135deg, #c8922a, #e8b84b)",
            color: loading || !topic.trim() || !canUseType ? "#333" : "#000",
            cursor: loading || !topic.trim() || !canUseType ? "not-allowed" : "pointer",
          }}>
            {loading ? "✨ Generating..." : "⚡ Generate Content"}
          </button>
        )}

        {error && <div style={styles.errorBox}>{error}</div>}

        {/* Result */}
        {result && (
          <div style={{ marginTop: 28 }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
              <span style={styles.sectionLabel}>Your Content</span>
              <div style={{ display: "flex", gap: 8 }}>
                <button onClick={() => { setResult(""); setTopic(""); }} style={styles.ghostBtn}>Clear</button>
                <button onClick={() => { navigator.clipboard.writeText(result); setCopied(true); setTimeout(() => setCopied(false), 2000); }} style={{
                  ...styles.ghostBtn,
                  borderColor: copied ? "#4a8a4a" : "#1e1e1e",
                  color: copied ? "#6aaa6a" : "#666",
                }}>
                  {copied ? "✓ Copied!" : "Copy"}
                </button>
              </div>
            </div>
            <div style={styles.resultBox}>{result}</div>
            <button onClick={generate} style={{ ...styles.ghostBtn, width: "100%", marginTop: 10, padding: "12px" }}>
              🔄 Regenerate
            </button>
          </div>
        )}

        <div style={{ marginTop: 40, textAlign: "center", fontSize: 11, color: "#222", borderTop: "1px solid #111", paddingTop: 16 }}>
          NaijaContentAI • Built for Nigerian Creators 🇳🇬 • Payments by Paystack
        </div>
      </div>
    </div>
  );
}

// ─── STYLES ──────────────────────────────────────────────────────────────────
const styles = {
  root: {
    minHeight: "100vh",
    background: "linear-gradient(160deg, #080808 0%, #120800 60%, #080808 100%)",
    fontFamily: "'Georgia', serif",
    color: "#f0ece0",
    position: "relative",
    overflowX: "hidden",
  },
  grain: {
    position: "fixed", inset: 0, pointerEvents: "none", zIndex: 0,
    backgroundImage: "url(\"data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.04'/%3E%3C/svg%3E\")",
    opacity: 0.4,
  },
  tagline: { fontSize: 12, letterSpacing: 3, color: "#c8922a", textTransform: "uppercase", marginBottom: 12 },
  hero: {
    fontSize: "clamp(2.2rem, 7vw, 3.8rem)", fontWeight: 900, margin: "0 0 12px",
    background: "linear-gradient(135deg, #e8b84b, #f5d98a, #c8922a)",
    WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent",
    letterSpacing: "-1px", textAlign: "center",
  },
  sub: { color: "#777", fontSize: 15, lineHeight: 1.6, marginBottom: 32 },
  card: {
    background: "#0f0f0f", border: "1px solid #1e1e1e", borderRadius: 16,
    padding: "28px 24px", textAlign: "left",
  },
  input: {
    width: "100%", background: "#080808", border: "1px solid #1e1e1e",
    borderRadius: 10, padding: "14px 16px", color: "#f0ece0",
    fontSize: 15, fontFamily: "inherit", outline: "none",
    boxSizing: "border-box", marginBottom: 12,
  },
  btnPrimary: {
    width: "100%", padding: "16px", background: "linear-gradient(135deg, #c8922a, #e8b84b)",
    border: "none", borderRadius: 12, color: "#000", fontSize: 15,
    fontWeight: 800, letterSpacing: 1, cursor: "pointer",
    textTransform: "uppercase", transition: "all 0.3s",
  },
  topbar: {
    display: "flex", justifyContent: "space-between", alignItems: "center",
    padding: "14px 24px", borderBottom: "1px solid #111",
    background: "#050505", position: "sticky", top: 0, zIndex: 100,
  },
  upgradeBtn: {
    background: "linear-gradient(135deg, #c8922a, #e8b84b)",
    border: "none", borderRadius: 20, color: "#000",
    fontSize: 11, fontWeight: 800, padding: "5px 14px",
    cursor: "pointer", letterSpacing: 1,
  },
  section: { marginBottom: 24 },
  sectionLabel: {
    display: "block", fontSize: 10, letterSpacing: 3,
    color: "#c8922a", textTransform: "uppercase", marginBottom: 10,
  },
  textarea: {
    width: "100%", background: "#0a0a0a", border: "1px solid #1e1e1e",
    borderRadius: 12, padding: "14px 16px", color: "#f0ece0",
    fontSize: 14, fontFamily: "inherit", resize: "vertical",
    outline: "none", boxSizing: "border-box", lineHeight: 1.6,
    transition: "border-color 0.2s",
  },
  resultBox: {
    background: "#0a0a0a", border: "1px solid #1e1e1e", borderLeft: "3px solid #c8922a",
    borderRadius: 12, padding: 20, fontSize: 14, lineHeight: 1.8,
    whiteSpace: "pre-wrap", color: "#e0d8c8",
  },
  ghostBtn: {
    padding: "7px 16px", background: "transparent", border: "1px solid #1e1e1e",
    borderRadius: 20, color: "#555", cursor: "pointer", fontSize: 12,
    transition: "all 0.2s",
  },
  errorBox: {
    marginTop: 14, padding: 14, background: "#1a0000",
    border: "1px solid #440000", borderRadius: 10, color: "#ff6666", fontSize: 13,
  },
  backBtn: {
    background: "transparent", border: "1px solid #1e1e1e", borderRadius: 20,
    color: "#666", cursor: "pointer", fontSize: 13, padding: "7px 16px",
    marginBottom: 24, display: "block",
  },
};
