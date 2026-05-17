// MindCare Ressources — Navigation CMS Framer
import { useState, useEffect } from "react"
import { addPropertyControls, ControlType } from "framer"

const AIRTABLE_PAT =
    "..."
const AIRTABLE_BASE_ID = "..."
const AIRTABLE_TABLE = "contenu positif"

const FILTERS = [
    { label: "Toutes" },
    { label: "Exercices" },
    { label: "Articles" },
    { label: "Témoignage" },
]

const toSlug = (titre) =>
    titre
        .toLowerCase()
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "")
        .replace(/[^a-z0-9\s-]/g, "")
        .trim()
        .replace(/\s+/g, "-")

export default function ResourceCenter({ accentColor, activeFilterColor }) {
    const [resources, setResources] = useState([])
    const [activeFilter, setActiveFilter] = useState("Toutes")
    const [status, setStatus] = useState("loading")

    const JAUNE = activeFilterColor || "#E6AD1A"
    const ORANGE = accentColor || "#D8A03E"
    const BG_COLOR = "#F8F8F8"
    const TEXT = "#1A1A1A"
    const MUTED = "#7A7A7A"

    useEffect(() => {
        fetchResources()
    }, [])

    const fetchResources = async () => {
        try {
            const res = await fetch(
                `https://api.airtable.com/v0/${AIRTABLE_BASE_ID}/${encodeURIComponent(AIRTABLE_TABLE)}`,
                { headers: { Authorization: `Bearer ${AIRTABLE_PAT}` } }
            )
            if (!res.ok) throw new Error("Erreur de chargement")
            const data = await res.json()

            const formattedData = data.records.map((record) => ({
                id: record.id,
                titre: record.fields.titre || "Ressource",
                typeContenu: record.fields["Type de contenu"] || "Articles",
                statut: record.fields.statut || "",
                description: record.fields.description || "",
                sourceURL: record.fields.sourceURL || "",
                lieu: record.fields.Lieu || "En ligne",
                horaires: record.fields.Horaires || "Toujours disponible",
                tags: record.fields.tags
                    ? record.fields.tags.split(",").map((t) => t.trim())
                    : ["Santé Mentale", "Gratuit"],
            }))

            setResources(formattedData)
            setStatus("success")
        } catch (error) {
            console.error(error)
            setStatus("error")
        }
    }

    const filteredResources =
        activeFilter === "Toutes"
            ? resources
            : resources.filter((r) => r.typeContenu === activeFilter)

    const iconForType = (type) => {
        const map = {
            Podcasts: "🎙",
            Exercices: "🧘",
            Articles: "📖",
            Témoignages: "✨",
            "Contenu Positif": "⭐",
        }
        return map[type] || "✦"
    }

    const s = {
        container: {
            fontFamily: "'DM Sans', sans-serif",
            width: "100%",
            maxWidth: 430,
            margin: "0 auto",
            background: BG_COLOR,
            minHeight: "100vh",
        },
        header: {
            display: "flex",
            alignItems: "center",
            gap: "12px",
            padding: "20px 20px 12px 20px",
            background: BG_COLOR,
        },
        backButton: {
            width: 36,
            height: 36,
            borderRadius: "50%",
            background: "#FFF",
            border: "1px solid #EAEAEA",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            cursor: "pointer",
            fontSize: "16px",
            color: TEXT,
            flexShrink: 0,
        },
        headerText: {
            display: "flex",
            flexDirection: "column",
            gap: "2px",
        },
        headerTitle: {
            fontSize: "22px",
            fontWeight: 700,
            color: TEXT,
            margin: 0,
        },
        headerSubtitle: {
            fontSize: "13px",
            color: MUTED,
            margin: 0,
        },
        banner: {
            margin: "0 16px 16px 16px",
            background: "#F5EDD8",
            borderRadius: "20px",
            padding: "20px",
        },
        bannerTitle: {
            fontSize: "17px",
            fontWeight: 700,
            color: TEXT,
            margin: "0 0 8px 0",
        },
        bannerText: {
            fontSize: "14px",
            color: "#5A5040",
            lineHeight: 1.5,
            margin: 0,
        },
        filtersContainer: {
            background: "#FFF",
            borderRadius: "20px",
            padding: "16px",
            margin: "0 16px 16px 16px",
            boxShadow: "0px 2px 8px rgba(0,0,0,0.03)",
        },
        filtersWrap: {
            display: "flex",
            flexWrap: "wrap",
            gap: "10px",
        },
        filterChip: (isActive) => ({
            padding: "8px 18px",
            borderRadius: "50px",
            fontSize: "14px",
            fontWeight: 600,
            cursor: "pointer",
            background: isActive ? JAUNE : "#F5F5F5",
            color: isActive ? "#1A1A1A" : "#555",
            border: isActive ? `1.5px solid ${JAUNE}` : "1.5px solid #EAEAEA",
            transition: "all 0.15s ease",
            whiteSpace: "nowrap",
        }),
        content: {
            padding: "0 16px 24px 16px",
        },
        card: {
            background: "#FFF",
            borderRadius: "20px",
            padding: "20px",
            marginBottom: "14px",
            boxShadow: "0px 2px 8px rgba(0,0,0,0.03)",
            cursor: "pointer",
            transition: "transform 0.15s ease, box-shadow 0.15s ease",
        },
        cardHeader: {
            display: "flex",
            alignItems: "center",
            gap: "10px",
            marginBottom: "14px",
        },
        cardIcon: {
            width: 40,
            height: 40,
            borderRadius: "50%",
            background: "#F5EDD8",
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            fontSize: "18px",
            flexShrink: 0,
        },
        typeBadge: {
            background: "#F5EDD8",
            color: "#C49A2A",
            padding: "6px 14px",
            borderRadius: "50px",
            fontSize: "12px",
            fontWeight: 700,
        },
        title: {
            fontSize: "18px",
            fontWeight: 700,
            color: TEXT,
            margin: "0 0 10px 0",
            lineHeight: 1.3,
        },
        description: {
            fontSize: "14px",
            color: MUTED,
            lineHeight: 1.55,
            margin: "0 0 16px 0",
        },
        tagsWrap: {
            display: "flex",
            flexWrap: "wrap",
            gap: "8px",
            marginBottom: "20px",
        },
        tagGray: {
            background: "#F5F5F5",
            color: "#444",
            padding: "7px 14px",
            borderRadius: "50px",
            fontSize: "13px",
            fontWeight: 500,
        },
        divider: {
            borderTop: "1px solid #F0EEEA",
            paddingTop: "16px",
            display: "flex",
            flexDirection: "column",
            gap: "12px",
        },
        infoItem: {
            display: "flex",
            alignItems: "center",
            gap: "8px",
            fontSize: "14px",
            color: MUTED,
        },
    }

    if (status === "loading")
        return (
            <div
                style={{
                    ...s.container,
                    textAlign: "center",
                    color: MUTED,
                    paddingTop: 60,
                }}
            >
                Chargement...
            </div>
        )
    if (status === "error")
        return (
            <div
                style={{
                    ...s.container,
                    textAlign: "center",
                    color: "red",
                    paddingTop: 60,
                }}
            >
                Erreur Airtable.
            </div>
        )

    return (
        <div style={s.container}>
            <div style={s.filtersContainer}>
                <div style={s.filtersWrap}>
                    {FILTERS.map((filter) => (
                        <div
                            key={filter.label}
                            style={s.filterChip(activeFilter === filter.label)}
                            onClick={() => setActiveFilter(filter.label)}
                        >
                            {filter.label}
                        </div>
                    ))}
                </div>
            </div>

            <div style={s.content}>
                {filteredResources.map((res) => (
                    <div
                        key={res.id}
                        style={s.card}
                        onClick={() => {
                            const slug = toSlug(res.titre)
                            window.location.href = `/ressources/${slug}`
                        }}
                        onMouseEnter={(e) => {
                            e.currentTarget.style.transform = "translateY(-2px)"
                            e.currentTarget.style.boxShadow =
                                "0px 6px 20px rgba(0,0,0,0.08)"
                        }}
                        onMouseLeave={(e) => {
                            e.currentTarget.style.transform = "translateY(0)"
                            e.currentTarget.style.boxShadow =
                                "0px 2px 8px rgba(0,0,0,0.03)"
                        }}
                    >
                        <div style={s.cardHeader}>
                            <div style={s.cardIcon}>
                                {iconForType(res.typeContenu)}
                            </div>
                            <span style={s.typeBadge}>{res.typeContenu}</span>
                        </div>
                        <h3 style={s.title}>{res.titre}</h3>
                        <p style={s.description}>{res.description}</p>
                        <div style={s.tagsWrap}>
                            {res.tags.map((tag, i) => (
                                <span key={i} style={s.tagGray}>
                                    {tag}
                                </span>
                            ))}
                        </div>
                        <div style={s.divider}>
                            <div style={s.infoItem}>📍 {res.lieu}</div>
                            <div style={s.infoItem}>🕒 {res.horaires}</div>
                        </div>
                    </div>
                ))}
            </div>
        </div>
    )
}

addPropertyControls(ResourceCenter, {
    accentColor: {
        type: ControlType.Color,
        title: "Couleur Badge Orange",
        defaultValue: "#D8A03E",
    },
    activeFilterColor: {
        type: ControlType.Color,
        title: "Couleur Active (Jaune)",
        defaultValue: "#E6AD1A",
    },
})
