// ============================================================
//  MultiStepForm.tsx — Framer Code Component
//  Formulaire 3 étapes → Airtable (Personal Access Token)
//  Design : MindCare — violet #7B2D8B, jaune #F5A623, fond crème
// ============================================================

import { useState } from "react"
import { addPropertyControls, ControlType } from "framer"

// ─── À CONFIGURER ───────────────────────────────────────────
const AIRTABLE_PAT =
    "patg4pydWSJAvkHhr.81aa619bbca203793b7db74c3b2aec687c6a6270b48ef336270125618ef57a1e"
const AIRTABLE_BASE_ID = "appE7rsiGhJkNvSe8"
const AIRTABLE_TABLE = "User"
// ────────────────────────────────────────────────────────────

const STEPS = [
    {
        label: "Étape 1 / 3",
        title: "Crée ton espace anonyme",
        subtitle: "Quelques infos pour personnaliser",
        fields: [
            {
                name: "pseudonyme",
                label: "Pseudonyme",
                type: "text",
                placeholder: "Ex: Phoenix2024",
                required: true,
            },
            {
                name: "mail",
                label: "Adresse mail",
                type: "email",
                placeholder: "toi@exemple.com",
                required: true,
            },
        ],
    },
    {
        label: "Étape 2 / 3",
        title: "Crée ton espace anonyme",
        subtitle: "Quelques infos pour personnaliser",
        fields: [
            {
                name: "age",
                label: "Âge",
                type: "number",
                placeholder: "Ex: 19 ans",
                required: true,
                hint: "Pour mieux personnaliser les ressources recommandées",
            },
            {
                name: "genre",
                label: "Genre (optionnel)",
                type: "select",
                options: [
                    "Homme",
                    "Femme",
                    "Non-binaire",
                    "Préfère ne pas répondre",
                ],
                required: false,
            },
        ],
        badge: "MindCare est fait pour les 18-24 ans. Tu es au bon endroit",
        cta: "Commencer gratuitement",
        footer: "En créant ton espace, tu acceptes nos conditions d'utilisation et notre politique de confidentialité RGPD.",
    },
    {
        label: "Étape 3 / 3",
        title: "Crée ton espace anonyme",
        subtitle: "Quelques infos pour personnaliser",
        fields: [
            {
                name: "localisation",
                label: "Localisation",
                type: "text",
                placeholder: "Ex: Béales",
                required: false,
                hint: "Pour te proposer des ressources près de chez toi",
            },
            {
                name: "situation",
                label: "Situation",
                type: "select",
                options: [
                    "Célibataire",
                    "En couple",
                    "Marié(e)",
                    "Divorcé(e)",
                    "Veuf/Veuve",
                    "Autre",
                ],
                required: false,
            },
        ],
        hint: "Ces informations restent 100% anonymes et nous aident à te proposer les ressources les plus adaptées à ta situation.",
        cta: "Créer mon compte",
    },
]

function validateField(field, value) {
    if (field.required && !value) return "Ce champ est obligatoire"
    if (field.type === "email" && value) {
        if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return "Email invalide"
    }
    if (field.type === "number" && value) {
        const n = Number(value)
        if (isNaN(n) || n < 0 || n > 120) return "Âge invalide (0–120)"
    }
    return null
}

export default function MultiStepForm({ accentColor, fontFamily }) {
    const [step, setStep] = useState(0)
    const [formData, setData] = useState({})
    const [errors, setErrors] = useState({})
    const [status, setStatus] = useState("idle")

    const PURPLE = "#7B2D8B"
    const YELLOW = "#F5A623"
    const BG = "#F5F4F0"
    const CARD_BG = "#FFFFFF"
    const INPUT_BG = "#F0EFE9"
    const TEXT = "#1A1A1A"
    const MUTED = "#888888"

    // Couleurs pastel des chips par index
    const CHIP_COLORS = [
        { bg: "#D6E8F5", text: "#2C5F7A" }, // bleu clair
        { bg: "#C8EDE4", text: "#1E5E4A" }, // vert menthe
        { bg: "#B8E8D8", text: "#1A5040" }, // vert
        { bg: "#D8EDD5", text: "#2A5228" }, // vert sauge
        { bg: "#F5EAC8", text: "#7A5A10" }, // jaune pâle
        { bg: "#F5D8CC", text: "#7A3020" }, // rose saumon
    ]

    const handleChipSelect = (name, value) => {
        setData((prev) => ({ ...prev, [name]: value }))
        setErrors((prev) => ({ ...prev, [name]: null }))
    }

    const handleChange = (e) => {
        const { name, value } = e.target
        setData((prev) => ({ ...prev, [name]: value }))
        setErrors((prev) => ({ ...prev, [name]: null }))
    }

    const handleNext = () => {
        const currentFields = STEPS[step].fields
        const newErrors = {}
        let hasError = false
        currentFields.forEach((f) => {
            const err = validateField(f, formData[f.name])
            if (err) {
                newErrors[f.name] = err
                hasError = true
            }
        })
        if (hasError) {
            setErrors(newErrors)
            return
        }
        setStep((s) => s + 1)
    }

    const handlePrev = () => setStep((s) => s - 1)

    const handleSubmit = async () => {
        const currentFields = STEPS[step].fields
        const newErrors = {}
        let hasError = false
        currentFields.forEach((f) => {
            const err = validateField(f, formData[f.name])
            if (err) {
                newErrors[f.name] = err
                hasError = true
            }
        })
        if (hasError) {
            setErrors(newErrors)
            return
        }
        setStatus("sending")
        try {
            const res = await fetch(
                `https://api.airtable.com/v0/${AIRTABLE_BASE_ID}/${encodeURIComponent(AIRTABLE_TABLE)}`,
                {
                    method: "POST",
                    headers: {
                        Authorization: `Bearer ${AIRTABLE_PAT}`,
                        "Content-Type": "application/json",
                    },
                    body: JSON.stringify({
                        fields: {
                            pseudonyme: formData.pseudonyme || "",
                            mail: formData.mail || "",
                            age: formData.age ? Number(formData.age) : null,
                            genre: formData.genre || "",
                            localisation: formData.localisation || "",
                            situation: formData.situation || "",
                        },
                    }),
                }
            )
            if (!res.ok) {
                const errorBody = await res.json()
                throw new Error(errorBody?.error?.message || "Erreur Airtable")
            }
            setStatus("success")
        } catch (err) {
            setStatus("error")
        }
    }

    const s = {
        outer: {
            background: BG,
            minHeight: "100vh",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            padding: "20px",
            fontFamily: fontFamily || "'DM Sans', system-ui, sans-serif",
        },
        card: {
            background: CARD_BG,
            borderRadius: 24,
            padding: "32px 28px 28px",
            width: "100%",
            maxWidth: 400,
            boxShadow: "0 4px 24px rgba(0,0,0,0.08)",
        },
        // Progress dots/bars
        progWrap: {
            display: "flex",
            gap: 6,
            justifyContent: "center",
            marginBottom: 28,
        },
        seg: (i) => ({
            width: 52,
            height: 5,
            borderRadius: 3,
            background: i <= step ? PURPLE : "#E0DDD6",
            transition: "background .35s",
        }),
        // Header
        title: {
            fontSize: 22,
            fontWeight: 800,
            color: TEXT,
            textAlign: "center",
            marginBottom: 6,
            lineHeight: 1.25,
        },
        subtitle: {
            fontSize: 13,
            color: MUTED,
            textAlign: "center",
            marginBottom: 24,
        },
        // Fields
        fieldWrap: { marginBottom: 16 },
        label: {
            display: "block",
            fontSize: 14,
            fontWeight: 700,
            color: TEXT,
            marginBottom: 8,
        },
        input: (hasErr) => ({
            width: "100%",
            padding: "14px 16px",
            border: `1.5px solid ${hasErr ? "#e24b4a" : "transparent"}`,
            borderRadius: 12,
            fontSize: 15,
            color: hasErr ? "#e24b4a" : MUTED,
            background: INPUT_BG,
            outline: "none",
            boxSizing: "border-box",
            fontFamily: "inherit",
        }),
        hint: {
            fontSize: 12,
            color: MUTED,
            marginTop: 6,
        },
        errorMsg: {
            fontSize: 12,
            color: "#e24b4a",
            marginTop: 4,
        },
        // Badge jaune
        badge: {
            background: YELLOW,
            borderRadius: 12,
            padding: "12px 16px",
            fontSize: 13,
            fontWeight: 700,
            color: "#fff",
            textAlign: "center",
            marginBottom: 16,
            lineHeight: 1.4,
        },
        // Bouton principal violet pill
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
            fontFamily: "inherit",
            marginBottom: 8,
            transition: "opacity .2s",
        },
        // Chips select
        chipsWrap: {
            display: "flex",
            flexWrap: "wrap",
            gap: 8,
            marginTop: 4,
        },
        chip: (color, selected) => ({
            padding: "8px 16px",
            borderRadius: 50,
            fontSize: 14,
            fontWeight: selected ? 700 : 500,
            cursor: "pointer",
            border: selected
                ? `2px solid ${color.text}`
                : "2px solid transparent",
            background: color.bg,
            color: color.text,
            transition: "all .15s",
            userSelect: "none",
        }),
        footer: {
            fontSize: 11,
            color: MUTED,
            textAlign: "center",
            marginTop: 16,
            lineHeight: 1.5,
        },
        // Succès
        successWrap: {
            textAlign: "center",
            padding: "32px 0",
        },
        successIcon: {
            width: 64,
            height: 64,
            borderRadius: "50%",
            background: PURPLE + "22",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            margin: "0 auto 16px",
            fontSize: 28,
        },
        successTitle: {
            fontSize: 22,
            fontWeight: 800,
            color: TEXT,
            marginBottom: 8,
        },
        successText: {
            fontSize: 14,
            color: MUTED,
            lineHeight: 1.6,
        },
    }

    if (status === "success") {
        return (
            <div style={s.outer}>
                <div style={s.card}>
                    <div style={s.successWrap}>
                        <div style={s.successIcon}>✓</div>
                        <p style={s.successTitle}>Merci !</p>
                        <p style={s.successText}>
                            Ton espace anonyme a bien été créé.
                        </p>
                    </div>
                </div>
            </div>
        )
    }

    const currentStep = STEPS[step]
    const isLast = step === STEPS.length - 1

    return (
        <div style={s.outer}>
            <div style={s.card}>
                {/* Titre */}
                <p style={s.title}>{currentStep.title}</p>
                <p style={s.subtitle}>{currentStep.subtitle}</p>

                {/* Barre de progression */}
                <div style={s.progWrap}>
                    {STEPS.map((_, i) => (
                        <div key={i} style={s.seg(i)} />
                    ))}
                </div>

                {/* Champs */}
                {currentStep.fields.map((f) => (
                    <div key={f.name} style={s.fieldWrap}>
                        <label style={s.label}>{f.label}</label>

                        {f.type === "select" ? (
                            <div style={s.chipsWrap}>
                                {f.options.map((o, i) => {
                                    const color =
                                        CHIP_COLORS[i % CHIP_COLORS.length]
                                    const selected = formData[f.name] === o
                                    return (
                                        <div
                                            key={o}
                                            style={s.chip(color, selected)}
                                            onClick={() =>
                                                handleChipSelect(f.name, o)
                                            }
                                        >
                                            {o}
                                        </div>
                                    )
                                })}
                            </div>
                        ) : (
                            <input
                                type={f.type}
                                name={f.name}
                                value={formData[f.name] || ""}
                                onChange={handleChange}
                                placeholder={f.placeholder || ""}
                                style={s.input(!!errors[f.name])}
                            />
                        )}

                        {f.hint && !errors[f.name] && (
                            <p style={s.hint}>{f.hint}</p>
                        )}
                        {errors[f.name] && (
                            <p style={s.errorMsg}>{errors[f.name]}</p>
                        )}
                    </div>
                ))}

                {/* Hint global étape 3 */}
                {currentStep.hint && (
                    <p
                        style={{
                            ...s.hint,
                            textAlign: "center",
                            marginBottom: 20,
                        }}
                    >
                        {currentStep.hint}
                    </p>
                )}

                {/* Badge jaune étape 2 */}
                {currentStep.badge && (
                    <div style={s.badge}>{currentStep.badge}</div>
                )}

                {/* Erreur réseau */}
                {status === "error" && (
                    <p
                        style={{
                            ...s.errorMsg,
                            marginBottom: 12,
                            fontSize: 13,
                        }}
                    >
                        Erreur lors de l'envoi. Vérifie ta connexion et
                        réessaie.
                    </p>
                )}

                {/* CTA principal */}
                {isLast ? (
                    <button
                        style={{
                            ...s.btnPrimary,
                            opacity: status === "sending" ? 0.6 : 1,
                            cursor:
                                status === "sending"
                                    ? "not-allowed"
                                    : "pointer",
                        }}
                        onClick={handleSubmit}
                        disabled={status === "sending"}
                    >
                        {status === "sending"
                            ? "Envoi…"
                            : currentStep.cta || "Créer mon compte"}
                    </button>
                ) : (
                    <button style={s.btnPrimary} onClick={handleNext}>
                        {currentStep.cta || "Suivant →"}
                    </button>
                )}

                {/* Footer RGPD */}
                {currentStep.footer && (
                    <p style={s.footer}>{currentStep.footer}</p>
                )}
            </div>
        </div>
    )
}

addPropertyControls(MultiStepForm, {
    accentColor: {
        type: ControlType.Color,
        title: "Couleur accent",
        defaultValue: "#7B2D8B",
    },
    fontFamily: {
        type: ControlType.String,
        title: "Police",
        defaultValue: "DM Sans, system-ui, sans-serif",
    },
})
