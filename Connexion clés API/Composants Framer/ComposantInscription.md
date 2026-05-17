//  MultiStepForm.tsx, Framer Code Component
//  Formulaire 3 étapes vers Airtable
// ============================================================
import { useState } from "react"
import { addPropertyControls, ControlType } from "framer"

// A CONFIGURER
const AIRTABLE_PAT =
    "..."
const AIRTABLE_BASE_ID = "..."
const AIRTABLE_TABLE = "User"

const STEPS = [
    {
        title: "Crée ton espace anonyme",
        subtitle: "Choisis ton pseudonyme unique",
        fields: [
            {
                name: "pseudonyme",
                label: "Pseudonyme",
                type: "text",
                placeholder: "Ex: Phoenix2024",
                required: true,
                description:
                    "C'est ton seul identifiant. Personne ne connaîtra ton vrai nom.",
            },
            {
                name: "email",
                label: "Adresse email",
                type: "email",
                placeholder: "Ex: tonemail@exemple.com",
                required: true,
                description:
                    "Utilisée uniquement pour te reconnecter à ton espace. Elle ne sera jamais partagée.",
            },
        ],
        infoBox: {
            icon: true,
            title: "Anonymat garanti",
            text: "Ton pseudonyme reste ton seul identifiant visible. Ton email est stocké de façon sécurisée et protégé selon le RGPD.",
        },
        cta: "Continuer",
    },
    {
        title: "Crée ton espace anonyme",
        subtitle: "Quelques infos pour personnaliser ton expérience",
        fields: [
            {
                name: "age",
                label: "Âge",
                type: "number",
                placeholder: "Ex: 19",
                required: true,
            },
            {
                name: "genre",
                label: "Genre (optionnel)",
                type: "select",
                options: ["Femme", "Homme", "Autre"],
                required: false,
                description:
                    "Pour mieux personnaliser les ressources qu'on te recommande",
            },
        ],
        simpleBox:
            "MindCare est fait pour les 18-24 ans. Tu es au bon endroit.",
        cta: "Continuer",
    },
    {
        title: "Crée ton espace anonyme",
        subtitle: "Dernière étape !",
        fields: [
            {
                name: "localisation",
                label: "Localisation",
                type: "text",
                placeholder: "Ex: Bègles",
                required: false,
                description: "Pour te proposer des ressources près de chez toi",
            },
            {
                name: "situation",
                label: "Situation",
                type: "select",
                options: ["Lycéen(ne)", "Étudiant(e)", "Salarié(e)", "Autre"],
                required: false,
            },
        ],
        footerHint:
            "Ces informations restent 100% anonymes et nous aident à te proposer les ressources les plus adaptées à ta situation.",
        cta: "Créer mon espace",
    },
]

function validateField(field, value) {
    if (field.required && !value) return "Ce champ est obligatoire"
    if (field.type === "number" && value) {
        const n = Number(value)
        if (isNaN(n) || n < 0 || n > 120) return "Âge invalide (0–120)"
    }
    if (field.type === "email" && value) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!emailRegex.test(value)) return "Adresse email invalide"
    }
    return null
}

export default function MultiStepForm({ accentColor, fontFamily, homeLink }) {
    const [step, setStep] = useState(0)
    const [formData, setData] = useState({})
    const [errors, setErrors] = useState({})
    const [status, setStatus] = useState("idle")

    const ACCENT = accentColor || "#E6AD1A"
    const BG_PAGE = "#F7F8F9"
    const BG_CARD = "#FFFFFF"
    const TEXT = "#1A1A1A"
    const MUTED = "#666666"
    const INPUT_BG = "#F4EFE6"
    const BORDER = "#EAEAEA"

    const handleSelect = (name, value) => {
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
                            email: formData.email || "",
                            age: formData.age ? Number(formData.age) : null,
                            genre: formData.genre || "",
                            localisation: formData.localisation || "",
                            situation: formData.situation || "",
                        },
                    }),
                }
            )
            if (!res.ok) {
                throw new Error("Erreur Airtable")
            }
            setStatus("success")
        } catch (err) {
            setStatus("error")
        }
    }

    const s = {
        outer: {
            background: BG_PAGE,
            minHeight: "100vh",
            display: "flex",
            flexDirection: "column",
            alignItems: "center",
            justifyContent: "center",
            padding: "16px",
            fontFamily: fontFamily || "'Inter', system-ui, sans-serif",
            boxSizing: "border-box",
        },
        card: {
            background: BG_CARD,
            borderRadius: 24,
            padding: "32px 24px 28px",
            width: "100%",
            maxWidth: 420,
            boxShadow: "0px 4px 20px rgba(0, 0, 0, 0.03)",
            border: `1px solid ${BORDER}`,
            boxSizing: "border-box",
        },
        logoWrap: {
            display: "flex",
            justifyContent: "center",
            marginBottom: 16,
        },
        title: {
            fontSize: 20,
            fontWeight: 800,
            color: TEXT,
            textAlign: "center",
            marginBottom: 6,
            lineHeight: 1.25,
        },
        subtitle: {
            fontSize: 14,
            color: MUTED,
            textAlign: "center",
            marginBottom: 24,
        },
        progWrap: {
            display: "flex",
            gap: 8,
            justifyContent: "center",
            marginBottom: 28,
        },
        seg: (i) => ({
            flex: 1,
            maxWidth: 60,
            height: 6,
            borderRadius: 4,
            background: i <= step ? ACCENT : "#EBEBEB",
            transition: "background .35s",
        }),
        fieldWrap: { marginBottom: 20 },
        label: {
            display: "block",
            fontSize: 14,
            fontWeight: 500,
            color: TEXT,
            marginBottom: 8,
        },
        input: (hasErr) => ({
            width: "100%",
            padding: "14px 16px",
            border: hasErr ? "1.5px solid #e24b4a" : "1.5px solid transparent",
            borderRadius: 16,
            fontSize: 15,
            color: TEXT,
            background: INPUT_BG,
            outline: "none",
            boxSizing: "border-box",
            fontFamily: "inherit",
            WebkitAppearance: "none",
            appearance: "none",
        }),
        chipsWrap: {
            display: "flex",
            flexWrap: "wrap",
            gap: 10,
            width: "100%",
        },
        chip: (selected) => ({
            flex: "1 1 calc(50% - 5px)",
            minWidth: 0,
            padding: "12px 8px",
            textAlign: "center",
            borderRadius: 16,
            fontSize: 14,
            fontWeight: 500,
            cursor: "pointer",
            border: selected ? `2px solid ${ACCENT}` : "2px solid transparent",
            background: INPUT_BG,
            color: TEXT,
            transition: "all .2s",
            userSelect: "none",
            boxSizing: "border-box",
        }),
        description: {
            fontSize: 12,
            color: MUTED,
            marginTop: 6,
            lineHeight: 1.4,
        },
        errorMsg: {
            fontSize: 12,
            color: "#e24b4a",
            marginTop: 6,
        },
        infoBoxOuter: {
            border: `1px solid ${BORDER}`,
            borderRadius: 16,
            padding: "16px",
            marginBottom: 24,
        },
        infoBoxHeader: {
            display: "flex",
            alignItems: "center",
            gap: 8,
            marginBottom: 6,
        },
        infoBoxTitle: {
            fontSize: 14,
            fontWeight: 700,
            color: TEXT,
            margin: 0,
        },
        infoBoxText: {
            fontSize: 12,
            color: MUTED,
            lineHeight: 1.5,
            margin: 0,
        },
        simpleBox: {
            border: `1px solid ${BORDER}`,
            borderRadius: 16,
            padding: "14px 16px",
            fontSize: 13,
            color: TEXT,
            lineHeight: 1.5,
            marginBottom: 24,
        },
        footerHintText: {
            fontSize: 12,
            color: TEXT,
            textAlign: "center",
            lineHeight: 1.5,
            marginBottom: 20,
            padding: "0 8px",
        },
        btnPrimary: {
            width: "100%",
            padding: "16px",
            border: "none",
            borderRadius: 50,
            background: ACCENT + "33",
            color: ACCENT,
            fontSize: 15,
            fontWeight: 800,
            cursor: "pointer",
            fontFamily: "inherit",
            transition: "opacity .2s",
            textDecoration: "none",
            display: "block",
            textAlign: "center",
            boxSizing: "border-box",
        },
        pageFooter: {
            fontSize: 12,
            color: MUTED,
            textAlign: "center",
            marginTop: 24,
            lineHeight: 1.6,
            maxWidth: 380,
            padding: "0 16px",
        },
        // Écran de confirmation
        successOuter: {
            textAlign: "center",
            padding: "24px 0 8px",
        },
        successIcon: {
            display: "flex",
            justifyContent: "center",
            marginBottom: 20,
        },
        successTitle: {
            fontSize: 20,
            fontWeight: 800,
            color: TEXT,
            marginBottom: 8,
        },
        successSubtitle: {
            fontSize: 14,
            color: MUTED,
            lineHeight: 1.5,
            marginBottom: 12,
        },
        successPseudo: {
            display: "inline-block",
            background: INPUT_BG,
            borderRadius: 12,
            padding: "10px 20px",
            fontSize: 15,
            fontWeight: 700,
            color: ACCENT,
            marginBottom: 28,
        },
        successList: {
            textAlign: "left",
            background: INPUT_BG,
            borderRadius: 16,
            padding: "16px 20px",
            marginBottom: 28,
            fontSize: 13,
            color: TEXT,
            lineHeight: 1.7,
        },
    }

    const LogoSVG = () => (
        <svg
            width="48"
            height="48"
            viewBox="0 0 24 24"
            fill="none"
            stroke="#1A1A1A"
            strokeWidth="1.5"
            strokeLinecap="round"
            strokeLinejoin="round"
        >
            <path d="M12 12C12 12 9 4 6 5C3 6 3 10 6 12C9 14 12 12 12 12Z" />
            <path d="M12 12C12 12 15 4 18 5C21 6 21 10 18 12C15 14 12 12 12 12Z" />
            <path d="M12 12C12 12 9 20 6 19C3 18 3 14 6 12C9 10 12 12 12 12Z" />
            <path d="M12 12C12 12 15 20 18 19C21 18 21 14 18 12C15 10 12 12 12 12Z" />
        </svg>
    )

    const LockSVG = () => (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="#1A1A1A"
            strokeWidth="2"
            strokeLinecap="round"
            strokeLinejoin="round"
        >
            <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
            <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
        </svg>
    )

    const CheckSVG = () => (
        <svg width="56" height="56" viewBox="0 0 56 56" fill="none">
            <circle cx="28" cy="28" r="28" fill={ACCENT + "22"} />
            <circle cx="28" cy="28" r="20" fill={ACCENT + "44"} />
            <path
                d="M19 28.5L25 34.5L37 22"
                stroke={ACCENT}
                strokeWidth="2.5"
                strokeLinecap="round"
                strokeLinejoin="round"
            />
        </svg>
    )

    // Écran succès enrichi
    if (status === "success") {
        return (
            <div style={s.outer}>
                <div style={s.card}>
                    <div style={s.successOuter}>
                        <div style={s.successIcon}>
                            <CheckSVG />
                        </div>
                        <p style={s.successTitle}>Ton espace est prêt !</p>
                        <p style={s.successSubtitle}>
                            Bienvenue sur MindCare,{" "}
                        </p>
                        <span style={s.successPseudo}>
                            {formData.pseudonyme || "toi"}
                        </span>
                        <div style={s.successList}>
                            <p style={{ margin: "0 0 6px", fontWeight: 600 }}>
                                Ce qui t'attend :
                            </p>
                            <p style={{ margin: "0 0 4px" }}>
                                Des ressources adaptées à ta situation
                            </p>
                            <p style={{ margin: "0 0 4px" }}>
                                Un espace 100% anonyme et sécurisé
                            </p>
                            <p style={{ margin: 0 }}>
                                Un lien de reconnexion envoyé à ton email
                            </p>
                        </div>
                        <a href={homeLink} target="_top" style={s.btnPrimary}>
                            Accéder à MindCare
                        </a>
                    </div>
                </div>
                <p style={s.pageFooter}>
                    Un doute ? Consulte notre politique de confidentialité RGPD.
                </p>
            </div>
        )
    }

    // Écran erreur
    if (status === "error") {
        return (
            <div style={s.outer}>
                <div style={s.card}>
                    <div style={{ textAlign: "center", padding: "24px 0" }}>
                        <p style={s.successTitle}>Une erreur est survenue</p>
                        <p style={s.successSubtitle}>
                            On n'a pas pu créer ton espace. Réessaie dans
                            quelques instants.
                        </p>
                        <button
                            style={s.btnPrimary}
                            onClick={() => setStatus("idle")}
                        >
                            Réessayer
                        </button>
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
                <div style={s.logoWrap}>
                    <LogoSVG />
                </div>

                <p style={s.title}>{currentStep.title}</p>
                <p style={s.subtitle}>{currentStep.subtitle}</p>

                <div style={s.progWrap}>
                    {STEPS.map((_, i) => (
                        <div key={i} style={s.seg(i)} />
                    ))}
                </div>

                {currentStep.fields.map((f) => (
                    <div key={f.name} style={s.fieldWrap}>
                        <label style={s.label}>{f.label}</label>

                        {f.type === "select" ? (
                            <div style={s.chipsWrap}>
                                {f.options.map((o) => {
                                    const selected = formData[f.name] === o
                                    return (
                                        <div
                                            key={o}
                                            style={s.chip(selected)}
                                            onClick={() =>
                                                handleSelect(f.name, o)
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
                                inputMode={
                                    f.type === "number"
                                        ? "numeric"
                                        : f.type === "email"
                                          ? "email"
                                          : "text"
                                }
                            />
                        )}

                        {f.description && (
                            <p style={s.description}>{f.description}</p>
                        )}
                        {errors[f.name] && (
                            <p style={s.errorMsg}>{errors[f.name]}</p>
                        )}
                    </div>
                ))}

                {currentStep.infoBox && (
                    <div style={s.infoBoxOuter}>
                        <div style={s.infoBoxHeader}>
                            {currentStep.infoBox.icon && <LockSVG />}
                            <p style={s.infoBoxTitle}>
                                {currentStep.infoBox.title}
                            </p>
                        </div>
                        <p style={s.infoBoxText}>{currentStep.infoBox.text}</p>
                    </div>
                )}

                {currentStep.simpleBox && (
                    <div style={s.simpleBox}>{currentStep.simpleBox}</div>
                )}

                {currentStep.footerHint && (
                    <p style={s.footerHintText}>{currentStep.footerHint}</p>
                )}

                <button
                    style={{
                        ...s.btnPrimary,
                        opacity: status === "sending" ? 0.6 : 1,
                        cursor:
                            status === "sending" ? "not-allowed" : "pointer",
                    }}
                    onClick={isLast ? handleSubmit : handleNext}
                    disabled={status === "sending"}
                >
                    {status === "sending" ? "Envoi en cours…" : currentStep.cta}
                </button>
            </div>

            <p style={s.pageFooter}>
                En créant ton espace, tu acceptes nos conditions d'utilisation
                et notre politique de confidentialité RGPD.
            </p>
        </div>
    )
}

addPropertyControls(MultiStepForm, {
    accentColor: {
        type: ControlType.Color,
        title: "Couleur accent",
        defaultValue: "#E6AD1A",
    },
    fontFamily: {
        type: ControlType.String,
        title: "Police",
        defaultValue: "Inter, system-ui, sans-serif",
    },
    homeLink: { type: ControlType.Link, title: "Lien final (CTA)" },
})
