//  MindCare — Cards Contenu Aide
import { useState, useEffect } from "react"
import { addPropertyControls, ControlType } from "framer"

const AIRTABLE_PAT =
    "..."
const AIRTABLE_BASE_ID = "..."
const AIRTABLE_TABLE = "contenu aide"

const FILTERS = [
    { label: "Toutes" },
    { label: "Lignes d'écoute" },
    { label: "BAPU" },
    { label: "Associations" },
    { label: "Consultations" },
]

const BADGE_COLORS = {
    "Ligne d'écoute": { bg: "#FF6B6B", text: "#FFF" },
    "Lignes d'écoute": { bg: "#FF6B6B", text: "#FFF" },
    Associations: { bg: "#4CAF7D", text: "#FFF" },
    BAPU: { bg: "#7B61FF", text: "#FFF" },
    Consultations: { bg: "#FF9500", text: "#FFF" },
    default: { bg: "#E0E0E0", text: "#444" },
}

const ICON_COLORS = {
    "Ligne d'écoute": { bg: "#FFE8E8", icon: "#FF6B6B" },
    "Lignes d'écoute": { bg: "#FFE8E8", icon: "#FF6B6B" },
    Associations: { bg: "#E8F5EE", icon: "#4CAF7D" },
    BAPU: { bg: "#EDE8FF", icon: "#7B61FF" },
    Consultations: { bg: "#FFF3E0", icon: "#FF9500" },
    default: { bg: "#F0F0F0", icon: "#888" },
}

const ICONS = {
    "Ligne d'écoute": "📞",
    "Lignes d'écoute": "📞",
    Associations: "🤝",
    BAPU: "🏥",
    Consultations: "💬",
    default: "📋",
}

const toSlug = (titre) =>
    titre
        .toLowerCase()
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "")
        .replace(/[^a-z0-9\s-]/g, "")
        .trim()
        .replace(/\s+/g, "-")

export default function ResourceCenterAide({ accentColor, activeFilterColor }) {
    const [resources, setResources] = useState([])
    const [activeFilter, setActiveFilter] = useState("Toutes")
    const [status, setStatus] = useState("loading")

    const JAUNE = activeFilterColor || "#8A0088"
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
                typeContenu: record.fields["Type de contenu"] || "Associations",
                statut: record.fields.statut || "",
                description: record.fields.description || "",
                sourceURL: record.fields.sourceURL || "",
                lieu: record.fields.Lieu || "En ligne",
                horaires: record.fields.Horaires || "Toujours disponible",
                tags: record.fields.tags
                    ? record.fields.tags.split(",").map((t) => t.trim())
                    : ["Aide", "Gratuit"],
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

    const getBadgeColor = (type) => BADGE_COLORS[type] || BADGE_COLORS.default
    const getIconStyle = (type) => ICON_COLORS[type] || ICON_COLORS.default
    const getIcon = (type) => ICONS[type] || ICONS.default

    const s = {
        container: {
            fontFamily: "'DM Sans', sans-serif",
            width: "100%",
            maxWidth: 430,
            margin: "0 auto",
            background: BG_COLOR,
            minHeight: "100vh",
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
        cardIcon: (type) => ({
            width: 40,
            height: 40,
            borderRadius: "50%",
            background: getIconStyle(type).bg,
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
            fontSize: "18px",
            flexShrink: 0,
        }),
        typeBadge: (type) => ({
            background: getBadgeColor(type).bg,
            color: getBadgeColor(type).text,
            padding: "6px 14px",
            borderRadius: "50px",
            fontSize: "12px",
            fontWeight: 700,
        }),
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
        link: {
            color: "#4A90E2",
            textDecoration: "none",
            fontWeight: 500,
            display: "flex",
            alignItems: "center",
            gap: "6px",
            fontSize: "14px",
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
                            window.location.href = `/contenu-aide/${slug}`
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
                            <div style={s.cardIcon(res.typeContenu)}>
                                {getIcon(res.typeContenu)}
                            </div>
                            <span style={s.typeBadge(res.typeContenu)}>
                                {res.typeContenu}
                            </span>
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
                            {res.sourceURL && (
                                <div style={s.infoItem}>
                                    <a
                                        href={res.sourceURL}
                                        target="_blank"
                                        rel="noopener noreferrer"
                                        style={s.link}
                                    >
                                        ↗ Visiter le site web
                                    </a>
                                </div>
                            )}
                        </div>
                    </div>
                ))}
            </div>
        </div>
    )
}

addPropertyControls(ResourceCenterAide, {
    activeFilterColor: {
        type: ControlType.Color,
        title: "Couleur Filtre Actif",
        defaultValue: "#E6AD1A",
    },
    accentColor: {
        type: ControlType.Color,
        title: "Couleur Accent",
        defaultValue: "#D8A03E",
    },
})
