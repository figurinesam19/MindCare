//  MindCare Form — Framer Code Component
//  Formulaire 2 étapes → Envoi vers Make → Affichage du résultat dynamique
// ============================================================
import { useState } from "react"
import { addPropertyControls, ControlType } from "framer"

// ─── À CONFIGURER ───────────────────────────────────────────
// Remplace ceci par l'URL du Webhook Custom généré dans Make
const MAKE_WEBHOOK_URL =
    "..."
// const STEPS = [
    {
        label: "Étape 1 / 2",
        title: "Ton état du jour",
        subtitle: "Prends un instant pour toi",
        fields: [
            {
                name: "humeur",
                label: "Comment te sens-tu aujourd'hui ?",
                type: "cards",
                options: [
                    { value: "super_bien", label: "Super bien", emoji: "🤩" },
                    { value: "plutot_bien", label: "Plutôt bien", emoji: "🙂" },
                    { value: "bof", label: "Bof", emoji: "😐" },
                    { value: "tres_mal", label: "Très mal", emoji: "😢" },
                ],
                required: true,
            },
        ],
    },
    {
        label: "Étape 2 / 2",
        title: "Ton énergie",
        subtitle: "Presque terminé",
        fields: [
            {
                name: "energie",
                label: "Comment est ton énergie sociale ?",
                type: "cards",
                options: [
                    {
                        value: "pleine",
                        label: "Plein.e d'énergie",
                        emoji: "⚡",
                    },
                    { value: "moyenne", label: "Énergie moyenne", emoji: "🔋" },
                    { value: "fatigue", label: "Fatigué.e", emoji: "🪫" },
                ],
                required: true,
            },
        ],
        cta: "Voir mes suggestions",
    },
]

export default function MentalHealthForm({
    accentColor,
    cardBgColor,
    textColor,
    resourcesLink,
}) {
    const [step, setStep] = useState(0)
    const [formData, setData] = useState({})
    const [status, setStatus] = useState("idle")
    const [suggestionResult, setSuggestionResult] = useState(null)

    const PURPLE = accentColor || "#7B2D8B"
    const TEXT = textColor || "#1A1A1A"
    const MUTED = "rgba(128, 128, 128, 0.8)"
    const CARD_BG = cardBgColor || "transparent"

    const handleSelect = (name, value) => {
        setData((prev) => ({ ...prev, [name]: value }))
    }

    const handleNext = () => {
        if (!formData[STEPS[step].fields[0].name]) return
        setStep((s) => s + 1)
    }

    const handlePrev = () => setStep((s) => s - 1)

    const handleSubmit = async () => {
        if (!formData.energie) return
        setStatus("sending")

        try {
            const res = await fetch(MAKE_WEBHOOK_URL, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(formData),
            })

            if (!res.ok) throw new Error("Erreur serveur")

            const responseData = await res.json()
            setSuggestionResult(
                responseData.suggestion ||
                    "Prends du temps pour toi aujourd'hui."
            )
            setStatus("success")
        } catch (err) {
            console.error(err)
            setTimeout(() => {
                setSuggestionResult(
                    "Il semblerait que notre serveur prenne une pause. Repose-toi bien !"
                )
                setStatus("success")
            }, 1000)
        }
    }

    const s = {
        outer: {
            display: "flex",
            justifyContent: "center",
            padding: 20,
            fontFamily: "'DM Sans', sans-serif",
        },
        card: {
            background: CARD_BG,
            borderRadius: 24,
            padding: "32px 28px",
            width: "100%",
            maxWidth: 400,
        },
        title: {
            fontSize: 22,
            fontWeight: 800,
            color: TEXT,
            textAlign: "center",
            marginBottom: 6,
        },
        subtitle: {
            fontSize: 13,
            color: MUTED,
            textAlign: "center",
            marginBottom: 24,
        },
        label: {
            display: "block",
            fontSize: 16,
            fontWeight: 700,
            color: TEXT,
            marginBottom: 16,
            textAlign: "center",
        },
        cardsWrap: { display: "flex", flexDirection: "column", gap: 12 },
        optionCard: (selected) => ({
            display: "flex",
            alignItems: "center",
            gap: 12,
            padding: "16px",
            borderRadius: 16,
            cursor: "pointer",
            transition: "all 0.2s",
            border: selected
                ? `2px solid ${PURPLE}`
                : "2px solid rgba(128,128,128,0.2)",
            background: selected ? `${PURPLE}11` : "transparent",
        }),
        emoji: { fontSize: 24 },
        optionText: { fontSize: 16, fontWeight: 600, color: TEXT },
        btnPrimary: {
            width: "100%",
            padding: "16px",
            border: "none",
            borderRadius: 50,
            background: PURPLE,
            color: "#fff",
            fontSize: 16,
            fontWeight: 700,
            cursor: "pointer",
            marginTop: 24,
            opacity: status === "sending" ? 0.6 : 1,
            display: "block",
            textAlign: "center",
            textDecoration: "none",
            boxSizing: "border-box",
        },
        resultBox: {
            background: `${PURPLE}11`,
            padding: 20,
            borderRadius: 16,
            marginTop: 16,
            border: `1px solid ${PURPLE}44`,
        },
        resultText: { fontSize: 15, color: TEXT, lineHeight: 1.5 },
    }

    if (status === "success") {
        return (
            <div style={s.outer}>
                <div style={s.card}>
                    <p style={s.title}>Voici ta suggestion</p>
                    <p style={s.subtitle}>Basée sur tes réponses</p>
                    <div style={s.resultBox}>
                        <p style={s.resultText}>{suggestionResult}</p>
                    </div>

                    {/* --- BOUTON DE REMPLACEMENT ICI --- */}
                    <a
                        href={resourcesLink}
                        target="_top"
                        onClick={(e) => {
                            if (!resourcesLink) e.preventDefault()
                        }}
                        style={s.btnPrimary}
                    >
                        Voir mes suggestions
                    </a>
                </div>
            </div>
        )
    }

    const currentStep = STEPS[step]
    const isLast = step === STEPS.length - 1
    const currentFieldName = currentStep.fields[0].name
    const canGoNext = !!formData[currentFieldName]

    return (
        <div style={s.outer}>
            <div style={s.card}>
                <p style={s.title}>{currentStep.title}</p>
                <p style={s.subtitle}>{currentStep.label}</p>
                <label style={s.label}>{currentStep.fields[0].label}</label>
                <div style={s.cardsWrap}>
                    {currentStep.fields[0].options.map((opt) => (
                        <div
                            key={opt.value}
                            style={s.optionCard(
                                formData[currentFieldName] === opt.value
                            )}
                            onClick={() =>
                                handleSelect(currentFieldName, opt.value)
                            }
                        >
                            <span style={s.emoji}>{opt.emoji}</span>
                            <span style={s.optionText}>{opt.label}</span>
                        </div>
                    ))}
                </div>

                {isLast ? (
                    <div style={{ display: "flex", gap: 10, marginTop: 24 }}>
                        <button
                            style={{
                                ...s.btnPrimary,
                                marginTop: 0,
                                background: "transparent",
                                color: TEXT,
                                border: "2px solid rgba(128,128,128,0.2)",
                            }}
                            onClick={handlePrev}
                        >
                            Retour
                        </button>
                        <button
                            style={{ ...s.btnPrimary, marginTop: 0 }}
                            onClick={handleSubmit}
                            disabled={!canGoNext}
                        >
                            {status === "sending"
                                ? "Analyse..."
                                : currentStep.cta}
                        </button>
                    </div>
                ) : (
                    <button
                        style={{
                            ...s.btnPrimary,
                            opacity: canGoNext ? 1 : 0.5,
                        }}
                        onClick={handleNext}
                        disabled={!canGoNext}
                    >
                        Suivant →
                    </button>
                )}
            </div>
        </div>
    )
}

addPropertyControls(MentalHealthForm, {
    accentColor: {
        type: ControlType.Color,
        title: "Accent",
        defaultValue: "#7B2D8B",
    },
    cardBgColor: {
        type: ControlType.Color,
        title: "Fond",
        defaultValue: "transparent",
    },
    textColor: {
        type: ControlType.Color,
        title: "Texte",
        defaultValue: "#1A1A1A",
    },
    resourcesLink: { type: ControlType.Link, title: "Lien Ressources (CTA)" },
})
