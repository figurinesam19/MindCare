//  MindCare Journal — Framer Code Component
//  Sauvegarde LOCALE (Local Storage) - 100% Privé & RGPD
// ============================================================
import { useState } from "react"
import { addPropertyControls, ControlType } from "framer"

export default function JournalForm({ accentColor, bgColor }) {
    const [mood, setMood] = useState(null)
    const [entry, setEntry] = useState("")
    const [status, setStatus] = useState("idle") // idle, success, error

    const PURPLE = accentColor || "#3A2B5E"
    const BG = bgColor || "#F6F5F0"
    const TEXT = "#1A1A1A"
    const MUTED = "#7A7A7A"

    const handleSubmit = () => {
        if (!mood || !entry.trim()) return

        try {
            // 1. Création de la nouvelle entrée
            const newEntry = {
                id: Date.now(),
                date: new Date().toISOString(),
                mood: mood,
                content: entry,
            }

            // 2. Récupération de l'historique existant dans le navigateur
            const existingData = localStorage.getItem("mindcare_journal")
            const history = existingData ? JSON.parse(existingData) : []

            // 3. Ajout et sauvegarde sur l'appareil de l'utilisateur
            history.push(newEntry)
            localStorage.setItem("mindcare_journal", JSON.stringify(history))

            // 4. Affichage du succès
            setStatus("success")
            setTimeout(() => {
                setStatus("idle")
                setMood(null)
                setEntry("")
            }, 3000)
        } catch (err) {
            console.error("Erreur de sauvegarde locale", err)
            setStatus("error")
        }
    }

    const s = {
        outer: {
            padding: 20,
            fontFamily: "'DM Sans', sans-serif",
            background: BG,
            minHeight: "100vh",
        },
        header: {
            display: "flex",
            alignItems: "center",
            gap: 16,
            marginBottom: 24,
        },
        backBtn: {
            width: 40,
            height: 40,
            borderRadius: "50%",
            border: "1px solid #E0E0E0",
            background: "#fff",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            cursor: "pointer",
            fontSize: 18,
        },
        pageTitle: { fontSize: 24, fontWeight: 800, color: TEXT, margin: 0 },
        pageSubtitle: { fontSize: 14, color: MUTED, margin: 0 },
        card: {
            background: "#fff",
            borderRadius: 24,
            padding: 24,
            boxShadow: "0px 4px 20px rgba(0,0,0,0.03)",
        },
        sectionTitle: {
            fontSize: 16,
            fontWeight: 700,
            color: TEXT,
            marginBottom: 12,
        },
        moodWrap: { display: "flex", gap: 12, marginBottom: 24 },
        moodBtn: (active) => ({
            flex: 1,
            padding: "16px 8px",
            borderRadius: 16,
            cursor: "pointer",
            border: active ? `2px solid ${PURPLE}` : "2px solid transparent",
            background: active ? `${PURPLE}11` : "#F5F5F5",
            display: "flex",
            flexDirection: "column",
            alignItems: "center",
            gap: 8,
            transition: "all 0.2s",
        }),
        moodIcon: { fontSize: 28 },
        moodLabel: { fontSize: 13, fontWeight: 600, color: TEXT },
        textarea: {
            width: "100%",
            height: 160,
            padding: 16,
            borderRadius: 16,
            border: "none",
            background: "#F5F5F5",
            fontSize: 15,
            color: TEXT,
            resize: "none",
            outline: "none",
            fontFamily: "inherit",
            marginBottom: 12,
            boxSizing: "border-box",
        },
        disclaimerWrap: {
            display: "flex",
            gap: 8,
            alignItems: "flex-start",
            marginBottom: 24,
        },
        disclaimerText: {
            fontSize: 12,
            color: MUTED,
            lineHeight: 1.4,
            margin: 0,
        },
        footerBtns: { display: "flex", gap: 12 },
        btnCancel: {
            flex: 1,
            padding: "14px",
            borderRadius: 50,
            border: "none",
            background: "#F5F5F5",
            color: TEXT,
            fontWeight: 600,
            cursor: "pointer",
        },
        btnSave: (disabled) => ({
            flex: 2,
            padding: "14px",
            borderRadius: 50,
            border: "none",
            background: PURPLE,
            color: "#fff",
            fontWeight: 600,
            cursor: disabled ? "not-allowed" : "pointer",
            opacity: disabled ? 0.5 : 1,
        }),
    }

    if (status === "success") {
        return (
            <div style={s.outer}>
                <div style={s.card}>
                    <p
                        style={{
                            textAlign: "center",
                            fontWeight: "bold",
                            fontSize: 18,
                            color: PURPLE,
                        }}
                    >
                        Journal sauvegardé ! 💙
                    </p>
                    <p
                        style={{
                            textAlign: "center",
                            fontSize: 14,
                            color: MUTED,
                        }}
                    >
                        Tes notes sont sécurisées sur ton appareil.
                    </p>
                </div>
            </div>
        )
    }

    return (
        <div style={s.outer}>
            <div style={s.header}>
                <button style={s.backBtn}>←</button>
                <div>
                    <h1 style={s.pageTitle}>Mon Journal</h1>
                    <p style={s.pageSubtitle}>
                        Ton espace privé pour écrire librement
                    </p>
                </div>
            </div>

            <div style={s.card}>
                <h2 style={s.sectionTitle}>Comment te sens-tu ?</h2>
                <div style={s.moodWrap}>
                    {[
                        { id: "Bien", icon: "🙂", label: "Bien" },
                        { id: "Neutre", icon: "😐", label: "Neutre" },
                        { id: "Difficile", icon: "☹️", label: "Difficile" },
                    ].map((m) => (
                        <div
                            key={m.id}
                            style={s.moodBtn(mood === m.id)}
                            onClick={() => setMood(m.id)}
                        >
                            <span style={s.moodIcon}>{m.icon}</span>
                            <span style={s.moodLabel}>{m.label}</span>
                        </div>
                    ))}
                </div>

                <h2 style={s.sectionTitle}>
                    Qu'est-ce qui se passe dans ta tête ?
                </h2>
                <textarea
                    style={s.textarea}
                    placeholder="Écris librement ce que tu ressens, ce qui t'est arrivé aujourd'hui, ce qui te préoccupe... Aucun jugement, juste toi et tes mots."
                    value={entry}
                    onChange={(e) => setEntry(e.target.value)}
                />

                <div style={s.disclaimerWrap}>
                    <span>🔒</span>
                    <p style={s.disclaimerText}>
                        Tes écrits sont stockés uniquement sur ton appareil.
                        Personne d'autre ne peut y accéder.
                    </p>
                </div>

                {status === "error" && (
                    <p style={{ color: "red", fontSize: 12, marginBottom: 12 }}>
                        Erreur lors de la sauvegarde locale.
                    </p>
                )}

                <div style={s.footerBtns}>
                    <button
                        style={s.btnCancel}
                        onClick={() => {
                            setMood(null)
                            setEntry("")
                        }}
                    >
                        Effacer
                    </button>
                    <button
                        style={s.btnSave(!mood || !entry.trim())}
                        onClick={handleSubmit}
                        disabled={!mood || !entry.trim()}
                    >
                        Enregistrer dans mon journal
                    </button>
                </div>
            </div>
        </div>
    )
}

addPropertyControls(JournalForm, {
    accentColor: {
        type: ControlType.Color,
        title: "Accent",
        defaultValue: "#3A2B5E",
    },
    bgColor: {
        type: ControlType.Color,
        title: "Fond global",
        defaultValue: "#F6F5F0",
    },
})
